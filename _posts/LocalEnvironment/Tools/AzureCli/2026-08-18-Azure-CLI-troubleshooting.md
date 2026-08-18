---
title: Azure CLI troubleshooting
date: 2026-08-18 16:05:00 +0200
categories: [Local Environment, Tools, Azure CLI]
tags: [azure, azure-cli, troubleshooting, authentication, wam]
description: Common Azure CLI troubleshooting patterns, including WAM login failures, stale cached sessions, and tenant-switch issues.
---

## Document Information

- **Azure CLI Version:** 2.84.0+
- **Operating System:** Windows
- **Focus:** Authentication and sign-in troubleshooting

## 1. WAM-related login failures

When Azure CLI uses the Windows broker integration, some machines can fail with errors such as:

```text
AADSTS1400011: Session key is not provided
```

This usually means the cached broker session is stale or invalid for the current tenant context. To fix this run below commands.

```powershell
az config set core.enable_broker_on_windows=false
az logout
az account clear
az login --tenant <tenant-id>
```

`core.enable_broker_on_windows=false` disables the Windows broker-based auth flow. After that, Azure CLI falls back to the standard interactive sign-in process instead of reusing a broken WAM session. `az logout` and `az account clear` remove cached Azure sessions so the next login starts fresh.

## 2. Stale Azure CLI session after tenant change

This is another common issue when the user changes tenants, subscriptions, or account contexts and Azure CLI still tries to reuse an outdated session. To fix this run below commands.

```powershell
az logout
az account clear
az login --tenant <tenant-id>
az account set --subscription <subscription-id>
```

If the tenant or subscription is correct but the CLI still points to the wrong account, clear the local state and sign in again.

## 3. Re-login after browser or token expiry

Sometimes the browser session or Azure AD token is expired, even though the local CLI still appears to be authenticated. To fix this run below commands.

```powershell
az logout
az account clear
az login
```

If the login is still blocked, open a new browser session and make sure the tenant is the correct one before re-running the command.
