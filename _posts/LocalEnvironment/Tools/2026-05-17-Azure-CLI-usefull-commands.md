---
title: Azure CLI usefull commands
date: 2026-05-14 20:10:00 +0200
categories: [Local Environment, Tools]
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
az login --tenant <tenant-id>
```

## List subscriptions

```powershell
az account list --output table
```

## Show current subscription

```powershell
az account show --output table
```

## Set active subscription

```powershell
az account set --subscription <subscription-id>
```

## Generate sas for blob storage with read and list rights

```powershell
az storage container generate-sas --account-name <storage-account-name> --name <container-name> --permissions lr --expiry '2024-11-06T16:42Z' --auth-mode login --as-user
```