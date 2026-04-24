---
title: Configure Service Principal for Easy Auth
date: 2026-04-23 14:04:00 +0200
categories: [EasyAuth]
tags: [easyauth, azure, service-principal, authentication]
description: This guide provides instructions on how to configure a service principal for Easy Auth. A service principal allows your application to authenticate and access resources securely.
---

## Used tools and versions

- **Azure CLI Version:** 2.84.0
- **PowerShell Version:** 7.6.1

## 1. Get Service Principal application ID

```powershell
(az rest `
  --method GET `
  --url "https://graph.microsoft.com/v1.0/servicePrincipals/<system-assigned-managed-identity-object-id>" `
  | ConvertFrom-Json).appId
```

## 2. Set the application ID as allowed client application in Easy Auth

### View current allowed client applications

```powershell
az webapp auth show \
  --resource-group <resource-group> \
  --name <app-name>
```

### Update allowed client applications list

```powershell
az webapp auth update \
  --resource-group <resource-group> \
  --name <app-name> \
  --set identityProviders.azureActiveDirectory.validation.allowedAudiences="['<app-id>']"
```
