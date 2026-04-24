---
title: DotNet tools
date: 2026-04-23 14:04:00 +0200
categories: [DotNet]
tags: [dotnet]
description: A guide on how to install and update .NET tools globally and locally.
---

## Used tools and versions

- **.NET SDK Version:** 10.0.202
- **PowerShell Version:** 7.6.1

## Install tool

### Install a tool globally

```powershell
dotnet tool install --global dotnet-ef --version <tool-version>
```

### Install a tool locally

```powershell
dotnet tool update dotnet-ef --version <tool-version>
```

And this creates a manifest file in the current directory if it doesn't exist.

```json
{
  "version": 1,
  "isRoot": true,
  "tools": {
    "dotnet-ef": {
      "version": "<tool-version>",
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
dotnet tool update --global dotnet-ef --version <tool-version>
```

### Update a tool locally

```powershell
dotnet tool update dotnet-ef --version <tool-version>
```
