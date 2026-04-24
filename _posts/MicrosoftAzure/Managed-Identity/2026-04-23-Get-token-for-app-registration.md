---
title: Get Token for App Registration
date: 2026-04-23 14:04:00 +0200
categories: [Microsoft Azure, Managed Identity]
tags: [azure, managed identity, service principal, access token]
description: This guide explains how to get an access token for an app registration in Azure AD.
---

## Document Information

- **Azure CLI Version:** 2.80.0  
- **PowerShell Version:** 7.5.4

## 1. Get Access Token

```powershell
az account get-access-token --scope api://<app-id-uri>/.default
```

## 2. Login Interactively

```powershell
az login --tenant "<tenant-id>" --scope "api://<app-id-uri>/.default"
```

## 3. Use the Token

```powershell
curl -v https://<app-url>/api/test -H "Authorization: Bearer $TOKEN"
```

## 4. App Registration Configuration

### If You Get This Error

```text
AADSTS650057: Invalid resource. The client has requested access to a resource which is not listed in the requested permissions in the client's application registration. ...
```

**How to Fix:**

1. Go to Azure Portal → App Registrations → Your App.
2. Under **Expose an API**:

- Add your API and Application ID URI.
- Add a scope (e.g., `user_impersonation`) and set admin consent display name/description.
- Add a client application (the app calling the API) and grant it access to the scope.

## 5. Predefined App IDs

| App Name            | App ID                               |
| ------------------- | ------------------------------------ |
| Visual Studio       | 04f0c124-f2bc-4f59-8241-bf6df9866bbd |
| Microsoft Azure CLI | 04b07795-8ddb-461a-bbee-02f9e1bf7b46 |
