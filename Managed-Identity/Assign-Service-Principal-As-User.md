# Assign Service Principal as User in Another App Registration

This guide explains how to register a service principal as a user with an assigned role in another app registration within the same Azure AD tenant.

## Prerequisites

- Both service principals must be registered in the same Azure AD tenant
- You must have sufficient permissions to manage app registrations and service principals
- Azure CLI installed and authenticated
- For cross-tenant scenarios, configure the app as multitenant first

## Step 1: Retrieve the Source Service Principal Object ID

Get the object ID of the service principal that will be granted access to the target application.

```powershell
az ad sp list `
    --filter "displayName eq '<SOURCE_APP_NAME>' and servicePrincipalType eq 'Application'" `
    --query '[0].id' `
    --output tsv
```

## Step 2: Retrieve the Target Service Principal Object ID

Get the object ID of the target application where the role assignment will be created.

```powershell
az ad sp list `
    --filter "displayName eq '<TARGET_APP_NAME>' and servicePrincipalType eq 'Application'" `
    --query '[0].id' `
    --output tsv
```

## Step 3: List Available App Roles

Retrieve the available app roles from the target application to identify the appropriate role ID.

```powershell
az ad sp show `
    --id <TARGET_SP_OBJECT_ID> `
    --query 'appRoles' `
    | ConvertFrom-Json `
    | Select-Object displayName, value, id `
    | Format-Table
```

## Step 4: Create the App Role Assignment

Assign the selected role to the source service principal in the context of the target application.

```powershell
az rest `
    --method POST `
    --url "https://graph.microsoft.com/v1.0/servicePrincipals/<SOURCE_SP_OBJECT_ID>/appRoleAssignments" `
    --headers "Content-Type=application/json" `
    --body '{\"principalId\": \"<SOURCE_SP_OBJECT_ID>\", \"resourceId\": \"<TARGET_SP_OBJECT_ID>\", \"appRoleId\": \"<ROLE_ID>\"}'
```

## Verification

To verify the role assignment was successful:

```powershell
az rest `
    --method GET `
    --url "https://graph.microsoft.com/v1.0/servicePrincipals/<SOURCE_SP_OBJECT_ID>/appRoleAssignments"
```
