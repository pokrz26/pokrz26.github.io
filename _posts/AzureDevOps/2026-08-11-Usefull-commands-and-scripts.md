---
title: Azure DevOps usefull commands and scripts
date: 2026-08-11 19:00:00 +0200
categories: [Azure DevOps]
tags: [azure-devops, scripts, api, agents]
description: Useful Azure DevOps API scripts for agent pool maintenance and organization user discovery.
---

## Document Information

- **PowerShell Version:** 7.6.4
- **Azure CLI Version:** 2.87.0
- **curl Version:** 8.18.0

## Authorize to Azure DevOps

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Before calling Azure DevOps REST APIs, make sure your Azure CLI context is set to the correct tenant connected to your DevOps organization.
{: .prompt-tip }
<!-- markdownlint-restore -->

### 1. Switch to the correct tenant and subscription

Login to Azure with the tenant ID associated with your Azure DevOps organization.

```powershell
az login `
  --tenant <tenant-id>
```

If you don't have access to any subscription in that tenant, use the `--allow-no-subscriptions` flag.

```powershell
az login `
  --tenant <tenant-id> `
  --allow-no-subscriptions
```

If you are already logged in, list all subscriptions and find the one associated with your DevOps organization.

```powershell
az account list `
  --all `
  | ConvertFrom-Json `
  | Select-Object id, name `
  | Select-String '<subscription-name-filter>'
```

Set the subscription ID from the output to switch to the correct subscription.

```powershell
az account set `
  --subscription <subscription-id>
```

### 2. Get token for Azure DevOps REST API

```powershell
$token = az account get-access-token `
  --resource 499b84ac-1321-427f-aa17-267ca6975798 `
  --query 'accessToken' `
  --output tsv
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Use `$token` as a Bearer token in your REST API requests.
{: .prompt-tip }
<!-- markdownlint-restore -->

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Keep your access token secure and do not commit it to source control.
{: .prompt-warning }
<!-- markdownlint-restore -->

## Add dummy agent for pool

Use this script to register an offline dummy agent entry in an existing agent pool.

```powershell
$organization = "<organization-name>"
$poolId = <agent-pool-id>
$token = "<access-token>"

curl -X POST `
  --header "Authorization: Bearer $token" `
  --header "Content-Type: application/json" `
  --data "{'name':'dummy','enabled':false,'status':'offline','version':'4.251.0'}" `
  "https://dev.azure.com/$organization/_apis/distributedtask/pools/$poolId/agents?api-version=7.1"
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> To get `<agent-pool-id>`, navigate to your Azure DevOps organization in a web browser, go to **Organization Settings** > **Agent Pools**, and select the desired agent pool. The URL will look like `https://dev.azure.com/<organization>/_settings/agentpools?poolId=<agent-pool-id>&view=jobs`. Copy `<agent-pool-id>` from that URL.
{: .prompt-info }
<!-- markdownlint-restore -->

## Get users in organization

Use the Graph Users API endpoint to list users in your Azure DevOps organization.

```powershell
$organization = "<organization-name>"
$token = "<access-token>"

curl -X GET `
  --header "Authorization: Bearer $token" `
  "https://vssps.dev.azure.com/$organization/_apis/graph/users?api-version=7.1-preview.1"
```
