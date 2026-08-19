---
title: Azure CLI usefull commands
date: 2026-07-14 00:00:00 +0200
categories: [Local Environment, Tools, Azure CLI]
tags: [azure, azure-cli]
description: Here you can find Azure Cli usefull commands.
---

## Document Information

- **Azure CLI Version:** 2.80.0
- **PowerShell Version:** 7.6.1

## Login

```powershell
az login
```

## Login with tenant

```powershell
az login `
  --tenant <tenant-id>
```

## List subscriptions

```powershell
az account list `
  --output table
```

## Show current subscription

```powershell
az account show `
  --output table
```

## Set active subscription

```powershell
az account set `
  --subscription <subscription-id>
```

## Generate sas for blob storage with read and list rights

```powershell
az storage container generate-sas `
  --account-name <storage-account-name> `
  --name <container-name> `
  --permissions lr `
  --expiry '2024-11-06T16:42Z' `
  --auth-mode login `
  --as-user
```

## Check resource name availability with Azure Resource Manager API

Use this pattern to verify whether a resource name is available in the selected subscription.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The exact endpoint, API version, and request body are resource-provider specific. Replace `<resource-provider>`, `<check-name-path>`, `<api-version>`, and `<resource-type>` with values supported by the target Azure service.
{: .prompt-info }
<!-- markdownlint-restore -->

```powershell
az rest `
  --method post `
  --uri 'https://management.azure.com/subscriptions/<subscription-id>/providers/<resource-provider>/<check-name-path>?api-version=<api-version>' `
  --headers 'Content-Type=application/json' `
  --body "{'name':'<resource-name>','type':'<resource-type>'}"
```

Example for Key Vault:

```powershell
az rest `
  --method post `
  --uri 'https://management.azure.com/subscriptions/<subscription-id>/providers/Microsoft.KeyVault/checkNameAvailability?api-version=2019-09-01' `
  --headers 'Content-Type=application/json' `
  --body "{'name':'<key-vault-name>','type':'Microsoft.KeyVault/vaults'}"
```
