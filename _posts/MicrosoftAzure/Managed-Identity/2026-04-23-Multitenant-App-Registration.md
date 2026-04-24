---
title: Multitenant App Registration in Azure AD
date: 2026-04-23 14:04:00 +0200
categories: [Microsoft Azure, Managed Identity]
tags: [azure, managed identity, service principal, app registration, multitenant]
description: This guide explains how to configure a multitenant app registration in Azure AD.
---

## Document Information

- **Azure CLI Version:** 2.80.0
- **PowerShell Version:** 7.5.4

## Overview

To enable an app registration across multiple tenants, you need to:

1. Configure the app registration as multitenant in the source tenant
2. Create a service principal in each destination tenant where you want to use the app

## Prerequisites

- Azure CLI installed
- Appropriate permissions in both source and destination tenants
- App Owner or Application Administrator role in the source tenant
- Cloud Application Administrator or Application Administrator role in the destination tenant

## Configuration Steps

### Step 1: Connect to Source Tenant

#### If the tenant has a subscription

List and select the appropriate subscription:

```powershell
# List all subscriptions
az account list `
    --all

# Find specific subscription by name
az account list `
    --all `
    --query "[?contains(name, '<subscription-name>')].{Id:id, Name:name}"

# Set active subscription
az account set `
    -s <subscription-id>
```

#### If the tenant has no subscription

Connect directly to the tenant:

```powershell
# List available tenants
az account tenant list

# Login to specific tenant without subscription
az login `
    --tenant <tenant-id> `
    --allow-no-subscriptions
```

### Step 2: Configure Multitenant Support

#### Find your app registration

```powershell
# Find app by display name and get its ID
$clientId = az ad app list `
    --filter "displayName eq 'YourAppName'" `
    --query '[0].appId' `
    -o tsv
```

#### Update the sign-in audience

```powershell
# Configure app registration for multitenant access
az ad app update `
    --id $clientId `
    --sign-in-audience AzureADMultipleOrgs
```

**Note:** Available sign-in audience values:

- `AzureADMyOrg` - Single tenant
- `AzureADMultipleOrgs` - Multitenant (any Azure AD tenant)
- `AzureADandPersonalMicrosoftAccount` - Multitenant + personal Microsoft accounts
- `PersonalMicrosoftAccount` - Personal Microsoft accounts only

### Step 3: Connect to Destination Tenant

Use the same authentication steps from [Step 1](#step-1-connect-to-source-tenant) to connect to the destination tenant where you want to provision the app.

### Step 4: Create Service Principal in Destination Tenant

Once connected to the destination tenant, create the service principal:

```powershell
# Create service principal using the app registration's client ID
az ad sp create `
    --id $clientId
```

This provisions the application in the destination tenant and allows you to assign it permissions and roles.

## Verification

To verify the service principal was created successfully:

```powershell
# List service principals by app ID
az ad sp list `
    --filter "appId eq '${clientId}'" `
    --query '[].{DisplayName:displayName, ObjectId:id}'
```
