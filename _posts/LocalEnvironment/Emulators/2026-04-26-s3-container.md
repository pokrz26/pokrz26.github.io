---
title: s3 emulator
date: 2026-04-26 12:10:00 +0200
categories: [Local Environment, Emulators]
tags: [local environemnt, emulator, s3]
description: A guide on how to start s3 emulator.
---

## Used tools and versions

- **PowerShell Version:** 7.6.1
- **Docker:** 29.4.1
- **Podman:** 5.8.2

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> To create buckets modify `initialBuckets` with your values.
{: .prompt-tip }
<!-- markdownlint-restore -->

## Docker

```powershell
docker pull adobe/s3mock:latest
docker volume create dev_s3
docker run `
  --name dev_s3 `
  -p 9090:9090 `
  -p 9191:9191 `
  -e initialBuckets=container1,container2 `
  -e debug=true `
  -v dev_s3:/containers3root `
  -t adobe/s3mock:latest
```

## Podman

```powershell
podman pull adobe/s3mock:latest
podman volume create dev_s3
podman run `
  --name dev_s3 `
  -p 9090:9090 `
  -p 9191:9191 `
  -e initialBuckets=container1,container2 `
  -e debug=true `
  -v dev_s3:/containers3root `
  -t adobe/s3mock:latest
```
