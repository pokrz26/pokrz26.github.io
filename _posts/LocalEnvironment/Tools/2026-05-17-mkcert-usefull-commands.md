---
title: mkcert usefull commands
date: 2026-05-17 14:30:00 +0200
categories: [Local Environment, Tools]
tags: [mkcert]
description: Here you can find mkcert usefull commands.
---

## Document Information

- **PowerShell Version:** 7.6.1

## Install mkcert

```powershell
winget install mkcert
```

## Install mkcert root CA

```powershell
mkcert -install
```

## Generate local certificate

This cert is generated for the root CA installed before.

```powershell
mkcert <certn-name>
```