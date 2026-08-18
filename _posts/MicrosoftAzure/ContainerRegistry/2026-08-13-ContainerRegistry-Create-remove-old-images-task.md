---
title: Keep the Latest Container Images in Azure Container Registry
date: 2026-08-13 09:15:00 +0200
categories: [Microsoft Azure, Container Registry]
tags: [azure, azure-container-registry, acr, bicep, azure-cli, containers]
description: Deploy an Azure Container Registry Task with Bicep that keeps a configurable number of recent images in each repository.
---

## Document Information

- **Azure CLI:** 2.84.0 or later
- **Bicep CLI:** Installed through the Azure CLI
- **ACR CLI image:** `mcr.microsoft.com/acr/acr-cli:0.19`

## Overview

Container image repositories can accumulate a large number of old tags after every build. Azure Container Registry (ACR) provides the `acr purge` command for removing old images, and an ACR Task can run that command on a schedule.

This Bicep template creates a system-assigned managed identity and a timer-triggered ACR Task. The task:

1. Logs in to Azure using its managed identity.
2. Lists every repository in the registry.
3. Selects the retention count for the repository.
4. Keeps the newest configured number of images.
5. Deletes the remaining images.

Repositories use `defaultImagesToKeep` unless their name starts with a prefix in `imageRetentionPolicies`. This makes it possible to apply stricter retention to large repositories while keeping more images for others.

## Prerequisites

- An existing Azure Container Registry.
- Permission to deploy resources and create role assignments in the registry's resource group.
- Azure CLI authenticated to the correct subscription.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The task has permission to delete images from the registry. Test the purge command with `--dry-run` before enabling deletion in a production registry.
{: .prompt-warning }
<!-- markdownlint-restore -->

## Bicep Template

Save the following as `acr-image-retention.bicep`:

```bicep
@export()
type ImageRetentionPolicy = {
  namespacePrefix: string
  imagesToKeep: int
}

param acrName string
param location string = resourceGroup().location
param imageRetentionPolicies ImageRetentionPolicy[] = []
param defaultImagesToKeep int = 10

var clearImagesPolicyCases = [for policy in imageRetentionPolicies: format(
  '        {0}*) imagesToKeep={1} ;;',
  policy.namespacePrefix,
  policy.imagesToKeep
)]

resource containerRegistry 'Microsoft.ContainerRegistry/registries@2025-11-01' existing = {
  name: acrName
}

var acrLoginServer = containerRegistry.properties.loginServer
var acrLoginSuffix = replace(replace(acrLoginServer, format('{0}-', acrName), ''), '.azurecr.io', '')

var clearImagesCommands = format('''
- cmd: az login --identity

- cmd: az acr login --name "{2}" --suffix "{3}" --expose-token --output tsv --query accessToken > /workspace/acr-token

- cmd: az acr repository list --name "{2}" --suffix "{3}" --output tsv > /workspace/repositories

- cmd: >-
    mcr.microsoft.com/acr/acr-cli:0.19 -c 'set -eu;
    repositoryList=/workspace/repositories;
    test -s "$repositoryList";
    token=$(cat /workspace/acr-token);
    while IFS= read -r repository; do
    imagesToKeep={0};
    case "$repository" in
    {1}
    esac;
    echo "Purging images from repository: $repository, keeping $imagesToKeep images";
    acr purge --registry "{2}" --username 00000000-0000-0000-0000-000000000000 --password "$token" --filter "$repository:.*" --ago 0d --keep "$imagesToKeep";
    done < "$repositoryList"'
  entryPoint: /bin/sh
  disableWorkingDirectoryOverride: true
  timeout: 3600
''', defaultImagesToKeep, join(clearImagesPolicyCases, ' '), acrLoginServer, acrLoginSuffix)

resource clearImagesTask 'Microsoft.ContainerRegistry/registries/tasks@2025-03-01-preview' = {
  parent: containerRegistry
  name: 'clear-images-weekly'
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    status: 'Enabled'
    platform: {
      os: 'Linux'
      architecture: 'amd64'
    }
    timeout: 3600
    trigger: {
      timerTriggers: [
        {
          name: 'clear-images-on-monday'
          schedule: '0 0 * * Mon'
          status: 'Enabled'
        }
      ]
    }
    step: {
      type: 'EncodedTask'
      encodedTaskContent: base64(format('version: v1.1.0\nsteps:\n{0}', clearImagesCommands))
      contextPath: '/dev/null'
    }
  }
}

var acrRepositoryCatalogListerRoleDefinitionId = subscriptionResourceId(
  'Microsoft.Authorization/roleDefinitions',
  'bfdb9389-c9a5-478a-bb2f-ba9ca092c3c7'
)

var acrRepositoryContributorRoleDefinitionId = subscriptionResourceId(
  'Microsoft.Authorization/roleDefinitions',
  '2efddaa5-3f1f-4df3-97df-af3f13818f4c'
)

resource clearImagesTaskAcrRepositoryCatalogListerRole 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(containerRegistry.id, clearImagesTask.identity.principalId, 'Container Registry Repository Catalog Lister')
  scope: containerRegistry
  properties: {
    roleDefinitionId: acrRepositoryCatalogListerRoleDefinitionId
    principalId: clearImagesTask.identity.principalId
    principalType: 'ServicePrincipal'
  }
}

resource clearImagesTaskAcrRepositoryContributorRole 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(containerRegistry.id, clearImagesTask.identity.principalId, 'Container Registry Repository Contributor')
  scope: containerRegistry
  properties: {
    roleDefinitionId: acrRepositoryContributorRoleDefinitionId
    principalId: clearImagesTask.identity.principalId
    principalType: 'ServicePrincipal'
  }
}
```

## How the Template Works

### Retention policy selection

The `case` block generated from `imageRetentionPolicies` matches repository names by prefix:

```sh
case "$repository" in
  team-a/*) imagesToKeep=5 ;;
  team-b/*) imagesToKeep=20 ;;
esac
```

If no prefix matches, the task uses `defaultImagesToKeep`.

### Managed identity and permissions

The task uses a system-assigned managed identity instead of storing registry credentials. The template assigns two built-in registry roles to that identity:

- **Container Registry Repository Catalog Lister** to list repositories.
- **Container Registry Repository Contributor** to delete images.

The role assignments are scoped to the existing registry rather than the whole subscription.

### Schedule and purge behavior

The task runs at midnight every Monday using the `0 0 * * Mon` cron expression. `acr purge --keep` retains the newest matching images, while `--ago 0d` allows all older images to be considered immediately. To validate the generated purge commands first, add `--dry-run` to the `acr purge` command and remove it only after reviewing the task logs. The template above is configured to perform deletion.

## Deploy with Azure CLI

Log in and select the subscription that contains the registry:

```powershell
az login
az account set --subscription "<subscription-id>"
```

Deploy the Bicep template to the registry's resource group. The following example keeps 10 images by default, 5 images in repositories beginning with `team-a/`, and 20 images in repositories beginning with `team-b/`:

```powershell
az deployment group create `
  --resource-group "<resource-group-name>" `
  --template-file .\acr-image-retention.bicep `
  --parameters `
    acrName="<container-registry-name>" `
    defaultImagesToKeep=10 `
    imageRetentionPolicies='[{"namespacePrefix":"team-a/","imagesToKeep":5},{"namespacePrefix":"team-b/","imagesToKeep":20}]'
```

The deployment creates or updates the task and its role assignments. The task then runs on its schedule; it does not need a separate container or virtual machine.

## Verify the Task

List the task and its timer trigger:

```powershell
az acr task show `
  --registry "<container-registry-name>" `
  --name "clear-images-weekly" `
  --output table
```

To run it immediately instead of waiting for Monday:

```powershell
az acr task run `
  --registry "<container-registry-name>" `
  --name "clear-images-weekly"
```

Inspect recent runs and logs:

```powershell
az acr task list-runs `
  --registry "<container-registry-name>" `
  --output table

az acr task logs `
  --registry "<container-registry-name>" `
  --run-id "<run-id>"
```

The logs show each repository and the number of images that the task keeps.
