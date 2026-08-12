---
title: Troubleshoot Azure Container App Activation Failures
date: 2026-08-11 18:20:00 +0200
categories: [Microsoft Azure, Container Apps]
tags: [azure, container-apps, aca, azure-cli, troubleshooting]
description: How to troubleshoot why an Azure Container App revision does not start by checking revision status, details, logs, and platform events.
---

## Document Information

- **Azure CLI Version:** 2.84.0

## Prerequisites

Set your app and resource group variables before running the commands.

```powershell
$appName = "<container-app-name>"
$resourceGroup = "<resource-group-name>"
```

## Important note about non-root containers

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> If your container runs as a non-root user, it cannot bind to privileged ports (typically ports below 1024). If your app listens on one of these ports, the container may fail to start.
{: .prompt-warning }
<!-- markdownlint-restore -->

Use a non-privileged port (for example `8080`) and ensure the Container App ingress/target port configuration matches your application.

## Step 1: Get exact revision status

List revisions and identify the one that is failing.

```powershell
az containerapp revision list `
  --name $appName `
  --resource-group $resourceGroup `
  --output table
```

## Step 2: Inspect the problematic revision

Set the failing revision name

```powershell
$revisionName = "<revision-name>"
```

and inspect its full details.

```powershell
az containerapp revision show `
  --name $appName `
  --resource-group $resourceGroup `
  --revision $revisionName `
  --output json
```

Check the output for error details and status-related fields that explain why activation failed.

## Step 3: Check container console logs

### First available replica of the revision

Retrieve logs for the specific failing revision (will show logs from the first available instance).

```powershell
az containerapp logs show `
  --name $appName `
  --resource-group $resourceGroup `
  --revision $revisionName `
  --type console
```

### Specific revision instance

If you need logs from an exact revision instance, first list all instances:

```powershell
az containerapp replica list `
  --name $appName `
  --resource-group $resourceGroup `
  --revision $revisionName `
  --output table
```

Store the replica name of the instance you want to inspect:

```powershell
$replicaName = "<replica-name>"
```

Then retrieve logs for a specific instance:

```powershell
az containerapp logs show `
  --name $appName `
  --resource-group $resourceGroup `
  --revision $revisionName `
  --replica $replicaName `
  --type console
```

## Step 4: If logs return "Could not find a replica for this app"

If no replica is available, use Azure Portal events for additional diagnostics:

`Container App -> Diagnose and solve problems -> Container Exit Events`

These event views often contain startup and runtime failure reasons that are not visible from revision console logs.

## Step 5: Store console logs in log analytics workspace

If there are intermittent container startup problems, you may not have time to connect to stream logs. Enable console log persistence to Log Analytics for proactive diagnostics.

### Configure Log Analytics on the Container Apps Environment

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The logging configuration must be set on the **Container Apps Environment**, not on individual Container Apps. Once configured, all Container Apps in that environment send their console output to the workspace.
{: .prompt-info }
<!-- markdownlint-restore -->

#### Using Azure Portal

1. Navigate to **Container Apps Environment → Monitoring → Logging options**
2. Set:
   - **Logs destination:** `Azure Log Analytics`
   - Select your **Log Analytics Workspace**
3. Save the configuration

#### Using Azure CLI

Set the environment variables:

```powershell
$environmentName = "<container-apps-environment-name>"
$resourceGroup = "<resource-group-name>"
$workspaceId = "<log-analytics-workspace-id>"
```

Update the Container Apps Environment:

```powershell
az containerapp env update `
  --name $environmentName `
  --resource-group $resourceGroup `
  --logs-destination log-analytics `
  --logs-workspace-id $workspaceId
```

Once configured, all Container Apps revisions in that environment will automatically send console logs to Log Analytics for investigation.


