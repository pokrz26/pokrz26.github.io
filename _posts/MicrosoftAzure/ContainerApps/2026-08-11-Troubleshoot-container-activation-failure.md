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

Retrieve logs for the specific failing revision.

```powershell
az containerapp logs show `
  --name $appName `
  --resource-group $resourceGroup `
  --revision $revisionName `
  --type console
```

## Step 4: If logs return "Could not find a replica for this app"

If no replica is available, use Azure Portal events for additional diagnostics:

1. `Container App -> Diagnose and solve problems -> Container Exit Events`
2. `Container App -> Revisions and replicas -> select revision -> Events`

These event views often contain startup and runtime failure reasons that are not visible from revision console logs.
