---
title: Cosmos DB emulator
date: 2026-04-26 12:20:00 +0200
categories: [Local Environment, Emulators]
tags: [local environemnt, emulator, Cosmos DB]
description: A guide on how to start Cosmos DB emulator.
---

## Used tools and versions

- **PowerShell Version:** 7.6.1
- **Docker:** 29.4.1
- **Podman:** 5.8.2

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Connection string: `AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`
{: .prompt-info }

> To enter portal use one of thet link: `https://localhost:8081/_explorer/index.html`
{: .prompt-tip }
<!-- markdownlint-restore -->

## Docker

```powershell
docker pull mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator
docker volume create dev_cosmosdb-no-sql (opcjonalnie - utworzy się sam razem z komendą docker run)
```

To find our IP address

```powershell
#windows
Ipconfig  #And we search for entry called WSL np. Ethernet adapter vEthernet (WSL (Hyper-V firewall)) …

#linux
ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}' | head -n 1
```

```powershell
docker run `
  --name=cosmosdb-no-sql `
  --publish 8081:8081 `
  --publish 10250-10255:10250-10255 `
  --memory 3g `
  --cpus=4.0 `
  --env AZURE_COSMOS_EMULATOR_PARTITION_COUNT=10 `
  --env AZURE_COSMOS_EMULATOR_ENABLE_DATA_PERSISTENCE=true `
  --env AZURE_COSMOS_EMULATOR_IP_ADDRESS_OVERRIDE=<ip-from-step-above> `
  --interactive `
  --tty `
  -v dev_cosmosdb-no-sql:/tmp/cosmos/appdata `
  -d mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator
```

Download certificate from Cosmos DB emulator

```powershell
curl -k https://localhost:8081/_explorer/emulator.pem > azure.cosmosdb.crt
```

Now install this certificate in Trusted Root Certification Authorities

## Podman

```powershell
podman pull mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator
podman volume create dev_cosmosdb-no-sql (opcjonalnie - utworzy się sam razem z komendą docker run)
```

To find our IP address

```powershell
#windows
Ipconfig  #And we search for entry called WSL np. Ethernet adapter vEthernet (WSL (Hyper-V firewall)) …

#linux
ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}' | head -n 1
```

```powershell
podman run `
  --name=cosmosdb-no-sql `
  --publish 8081:8081 `
  --publish 10250-10255:10250-10255 `
  --memory 3g `
  --cpus=4.0 `
  --env AZURE_COSMOS_EMULATOR_PARTITION_COUNT=10 `
  --env AZURE_COSMOS_EMULATOR_ENABLE_DATA_PERSISTENCE=true `
  --env AZURE_COSMOS_EMULATOR_IP_ADDRESS_OVERRIDE=<ip-from-step-above> `
  --interactive `
  --tty `
  -v dev_cosmosdb-no-sql:/tmp/cosmos/appdata `
  -d mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator
```

Download certificate from Cosmos DB emulator

```powershell
curl -k https://localhost:8081/_explorer/emulator.pem > azure.cosmosdb.crt
```

Now install this certificate in Trusted Root Certification Authorities
