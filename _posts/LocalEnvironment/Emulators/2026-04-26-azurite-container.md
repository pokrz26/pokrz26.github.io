---
title: azurite emulator
date: 2026-04-26 12:11:00 +0200
categories: [Local Environment, Emulators]
tags: [local environemnt, emulator, azurite]
description: A guide on how to start azurite emulator.
---

## Used tools and versions

- **PowerShell Version:** 7.6.1
- **Docker:** 29.4.1
- **Podman:** 5.8.2
- **mkcert:** 3.2.0

## http

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Connection string:
> - `UseDevelopmentStorage=true`
> - `DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;TableEndpoint=http://127.0.0.1:10002/devstoreaccount1;`
{: .prompt-info }
<!-- markdownlint-restore -->

### Docker

```powershell
docker pull mcr.microsoft.com/azure-storage/azurite:latest
docker volume create dev_azurite (opcjonalnie - utworzy się sam razem z komendą docker run)
docker run `
  --name dev_azurite `
  -p 10000:10000 `
  -p 10001:10001 `
  -p 10002:10002 `
  -v dev_azurite:/data `
  -d mcr.microsoft.com/azure-storage/azurite:latest
```

### Podman

```powershell
podman pull mcr.microsoft.com/azure-storage/azurite:latest
podman volume create dev_azurite (opcjonalnie - utworzy się sam razem z komendą docker run)
podman run `
  --name dev_azurite `
  -p 10000:10000 `
  -p 10001:10001 `
  -p 10002:10002 `
  -v dev_azurite:/data `
  -d mcr.microsoft.com/azure-storage/azurite:latest
```

## https

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Connection string:
> - Blob endpoint: `https://127.0.0.1:10000/devstoreaccount1`
> - Table endpoint: `https://127.0.0.1:10002/devstoreaccount1`
> - Queue endpoint: `https://127.0.0.1:10001/devstoreaccount1`
> - Connection string: `DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;TableEndpoint=https://127.0.0.1:10002/devstoreaccount1;`
{: .prompt-info }
<!-- markdownlint-restore -->

### Docker

```powershell
docker pull mcr.microsoft.com/azure-storage/azurite
docker volume create dev_azurite

cd '\\wsl$\docker-desktop-data\data\docker\volumes'

mkcert 127.0.0.1 do folderu dev_azurite/data/cert

docker run `
  --name dev_azurite `
  -p 10000:10000 `
  -p 10001:10001 `
  -p 10002:10002 `
  -v dev_azurite:/data `
  -d mcr.microsoft.com/azure-storage/azurite:latest `
  azurite --location /data --cert /data/cert/127.0.0.1.pem --key /data/cert/127.0.0.1-key.pem --oauth basic --blobHost 0.0.0.0 --queueHost 0.0.0.0 --tableHost 0.0.0.0 --skipApiVersionCheck
```

### Podman

```powershell
podman pull mcr.microsoft.com/azure-storage/azurite
podman volume create dev_azurite

cd '\\wsl.localhost\podman-machine-default/\home\user\.local\share\containers\storage\volumes'

mkcert 127.0.0.1 do folderu dev_azurite/_data/cert

docker run `
  --name dev_azurite `
  -p 10000:10000 `
  -p 10001:10001 `
  -p 10002:10002 `
  -v dev_azurite:/data `
  -d mcr.microsoft.com/azure-storage/azurite:latest `
  azurite --location /data --cert /data/cert/127.0.0.1.pem --key /data/cert/127.0.0.1-key.pem --oauth basic --blobHost 0.0.0.0 --queueHost 0.0.0.0 --tableHost 0.0.0.0 --skipApiVersionCheck
```