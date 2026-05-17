---
title: rclone usefull commands
date: 2026-05-17 14:30:00 +0200
categories: [Local Environment, Tools]
tags: [rclone]
description: Here you can find rclone usefull commands.
---

## Document Information

- **PowerShell Version:** 7.6.1

## Configuration

To find rclone config path type

```powershell
rclone config file
```

Example configuration

```
[<azure-blob-remote-name>]
type = azureblob
account = 
sas_url = 

[<s3-compatible-remote-name>]
type = s3
provider = Cloudflare
access_key_id = 
secret_access_key = 
endpoint = 
acl = private
```

## List commands

### List azure blobs

```powershell
rclone lsf <azure-blob-remote-name>:<container-name>
```

### List cloudflare blobs

```powershell
rclone lsf <s3-compatible-remote-name>:<bucket-name>
```

## Sync commands

### Sync blobs dry run

```powershell
rclone sync --progress <azure-blob-remote-name>:<container-name> <s3-compatible-remote-name>:<bucket-name> --dry-run
```

### Sync blobs

```powershell
rclone sync --progress <azure-blob-remote-name>:<container-name> <s3-compatible-remote-name>:<bucket-name>
```

### Sync blobs and fix md5 validation issue

```powershell
rclone sync --progress <azure-blob-remote-name>:<container-name> <s3-compatible-remote-name>:<bucket-name> --ignore-checksum
```