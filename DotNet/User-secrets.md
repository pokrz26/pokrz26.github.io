# User Secrets management

## Document Information

- **Last Updated:** February 25, 2026
- **.NET SDK Version:** 10.0.103
- **PowerShell Version:** 7.5.4

## Path to secrets file

### Windows

```powershell
%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json
```

### Linux/MacOS

```bash
~/.microsoft/usersecrets/<user_secrets_id>/secrets.json
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
