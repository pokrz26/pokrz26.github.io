---
title: Create Kali Linux VM
date: 2026-05-14 20:10:00 +0200
categories: [Microsoft Azure, Virtual Machine]
tags: [azure, virtual-machine, kali-linux, azure-cli, ssh]
description: This guide explains how to create a Kali Linux virtual machine in Azure and connect to it over SSH.
---

## Document Information

- **Platform:** Microsoft Azure
- **Image:** `kali-linux:kali:kali-2026-1:latest`
- **VM Size:** `Standard_D8as_v6`

## Kali Linux Marketplace Image

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Azure uses the `Publisher:Offer:Sku:Version` format for marketplace images.
{: .prompt-info }
<!-- markdownlint-restore -->

```json
"imageReference": {
  "publisher": "kali-linux",
  "offer": "kali",
  "sku": "kali-2026-1",
  "version": "latest"
}
```

Equivalent Azure CLI image value:

```text
kali-linux:kali:kali-2026-1:latest
```

## Create the Virtual Machine

Use the following command to create the Kali Linux VM:

```powershell
az vm create `
  --resource-group <resource-group-name> `
  --name <vm-name> `
  --size Standard_D8as_v6 `
  --image kali-linux:kali:kali-2026-1:latest `
  --plan-name kali-2026-1 `
  --plan-product kali `
  --plan-publisher kali-linux `
  --admin-username kaliadmin `
  --generate-ssh-keys `
  --os-disk-size-gb 128 `
  --os-disk-delete-option Delete `
  --storage-sku StandardSSD_LRS `
  --public-ip-sku Standard `
  --nsg team-red-burp-vm-test-nsg
```

The `--plan-*` parameters are required for marketplace images that need purchase plan information.

## Configure Auto-Shutdown

To automatically shut down the VM every day at 18:00, run:

```powershell
az vm auto-shutdown `
  --resource-group <resource-group-name> `
  --name <vm-name> `
  --time 1800
```

## Connect Over SSH

After the VM is created, connect to it with SSH:

```powershell
ssh -i .\kaliadmin.pem insadmin@<public-ip-address>
```

Replace `<public-ip-address>` with the public IP assigned to the virtual machine.