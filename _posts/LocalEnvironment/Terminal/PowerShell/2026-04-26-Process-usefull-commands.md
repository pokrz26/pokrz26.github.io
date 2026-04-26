---
title: PowerShell process usefull commands
date: 2026-04-26 12:12:00 +0200
categories: [Local Environment, Terminal, PowerShell]
tags: [local environemnt, terminal, powershell]
description: Usefull commands for process management.
---

## Used tools and versions

- **PowerShell Version:** 7.6.1

## Find process that is using port

```powershell
Get-Process -Id (Get-NetTCPConnection -LocalPort <port>).OwningProcess
```

## Kill process by name

```powershell
Get-Process -Name <process-name> | Stop-Process
```

## Start process

```powershell
Start-Process "C:\example\path\migrationBundle.exe" `
  -WorkingDirectory "C:\example\path" `
  -NoNewWindow `
  -ArgumentList @(`
    "--verbose", `
    "--connection", `
    "`"Data Source=(localdb)\mssqllocaldb;database=MIG_1;trusted_connection=yes;`"", `
    "--", `
    "`"ENV_VARIABLE=Value`""`
  )
```