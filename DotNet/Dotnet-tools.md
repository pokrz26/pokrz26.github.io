# DotNet tools

## Document Information

- **Last Updated:** March 11, 2026
- **.NET SDK Version:** 10.0.103
- **PowerShell Version:** 7.5.4

## Install tool

### Install a tool globally

```powershell
dotnet tool install --global dotnet-ef --version 10.0.4
```

### Install a tool locally

```powershell
dotnet tool update dotnet-ef --version 10.0.4
```

And this creates a manifest file in the current directory if it doesn't exist.

```json
{
  "version": 1,
  "isRoot": true,
  "tools": {
    "dotnet-ef": {
      "version": "10.0.4",
      "commands": [
        "dotnet-ef"
      ],
      "rollForward": false
    }
  }
}
```

### Update a tool

### Update a tool globally

```powershell
dotnet tool update --global dotnet-ef --version 10.0.4
```

### Update a tool locally

```powershell
dotnet tool update dotnet-ef --version 10.0.4
```
