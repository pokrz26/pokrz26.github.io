---
title: MS SQL emulator
date: 2026-04-26 12:12:00 +0200
categories: [Local Environment, Emulators]
tags: [local environemnt, emulator, MS SQL]
description: A guide on how to start MS SQL emulator.
---

## Used tools and versions

- **PowerShell Version:** 7.6.1
- **Docker:** 29.4.1
- **Podman:** 5.8.2

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Connection string: `Data Source=127.0.0.1,5433;database=<database-name>;User Id=sa;Password=Pass@word;TrustServerCertificate=true `.
{: .prompt-tip }
<!-- markdownlint-restore -->

## Docker

```powershell
docker pull mcr.microsoft.com/mssql/server:2025-latest
docker volume create dev_mssql (opcjonalnie - utworzy się sam razem z komendą docker run)
docker run `
  --name dev_mssql `
  -e "ACCEPT_EULA=Y" `
  -e "SA_PASSWORD=P@ssw0rd" `
  -p 1433:1433 `
  -v dev_mssql:/var/opt/mssql `
  -d mcr.microsoft.com/mssql/server:2025-latest
```

## Podman

```powershell
podman pull mcr.microsoft.com/mssql/server:2025-latest
podman volume create dev_mssql (opcjonalnie - utworzy się sam razem z komendą docker run)
podman run `
  --name dev_mssql `
  -e "ACCEPT_EULA=Y" `
  -e "SA_PASSWORD=P@ssw0rd" `
  -p 1433:1433 `
  -v dev_mssql:/var/opt/mssql `
  -d mcr.microsoft.com/mssql/server:2025-latest
```