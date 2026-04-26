---
title: User Secrets
date: 2026-04-23 14:18:00 +0200
categories: [Local Environment, Tools, DotNet]
tags: [local environemnt,  tools, dotnet]
description: A guide on how to manage user secrets in .NET applications.
---

## Used tools and versions

- **.NET SDK Version:** 10.0.202
- **PowerShell Version:** 7.6.1

## Path to secrets file

### Windows

```powershell
%APPDATA%\Microsoft\UserSecrets\<user-secrets-id>\secrets.json
```

### Linux/MacOS

```bash
~/.microsoft/usersecrets/<user-secrets-id>/secrets.json
```

## DotNet CLI commands

### Create a new user secrets file

```powershell
dotnet user-secrets init
```

### Set a secret value

```powershell
dotnet user-secrets set "Movies:ServiceApiKey" "12345"
dotnet user-secrets set "Movies:ServiceApiKey" "12345" --project "C:\apps\WebApp1\src\WebApp1"
```

### List all secrets

```powershell
dotnet user-secrets list
dotnet user-secrets list --project "C:\apps\WebApp1\src\WebApp1"
```

### Remove a secret

```powershell
dotnet user-secrets remove "Movies:ConnectionString"
dotnet user-secrets remove "Movies:ConnectionString" --project "C:\apps\WebApp1\src\WebApp1"
```

### Clear all secrets

```powershell
dotnet user-secrets clear
dotnet user-secrets clear --project "C:\apps\WebApp1\src\WebApp1"
```
