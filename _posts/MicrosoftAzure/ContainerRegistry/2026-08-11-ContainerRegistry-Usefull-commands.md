---
title: Useful Azure Container Registry Commands
date: 2026-08-11 18:10:00 +0200
categories: [Microsoft Azure, Container Registry]
tags: [azure, acr, azure-cli, container-registry]
description: A collection of useful Azure Container Registry commands for running and managing ACR tasks.
---

## Document Information

- **Azure CLI Version:** 2.84.0

## Run ACR tasks

### Run a specific ACR task manually

Use this command to manually trigger an existing scheduled task.

```powershell
az acr task run `
  --registry <acr-name> `
  --name <acr-task-name> `
  --no-logs `
  --debug
```