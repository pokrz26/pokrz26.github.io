# READ ME

## Registering service principal as user in different app registration

### 1. Get the service principal object ID of the service principal you want to register as a user in another app registration. App must be registered in the same tenant

``` PowerShell
az ad sp list `
    --filter "displayName eq '{Application name}' and servicePrincipalType eq 'Application'" `
    --query '[0].id'

// Example

az ad sp list `
    --filter "displayName eq 'Azure DevOps InsERT WSW Test V2' and servicePrincipalType eq 'Application'" `
    --query '[0].id'
```

### 2. Get the service principal object ID of the target app where you want to add the service principal as a user. App must be registered in the same tenant

``` PowerShell
az ad sp list `
    --filter "displayName eq '{Application name}' and servicePrincipalType eq 'Application'" `
    --query '[0].id'

// Example

az ad sp list `
    --filter "displayName eq 'wsw-insmonitor-dev' and servicePrincipalType eq 'Application'" `
    --query '[0].id'
```

### 3. Retrieving role id of the role you want to assign to the service principal in the target app registration

``` PowerShell
az ad sp show `
    --id {Object id from step 2} `
    --query 'appRoles' `
    | ConvertFrom-Json `
    | Select-Object displayName, value, id

// Example
az ad sp show `
    --id 9dfddf8c-3e16-435b-b93b-5d421aa134eb `
    --query 'appRoles' `
    | ConvertFrom-Json `
    | Select-Object displayName, value, id
```

### 4. Assign the role to the service principal in the target app registration

``` PowerShell
az rest `
    --method POST `
    --url "https://graph.microsoft.com/v1.0/servicePrincipals/{Object id of service principal from step 1}/appRoleAssignments" `
    --headers "Content-Type=application/json" `
    --body '{\"principalId\": \"{Object id of service principal from step 1}\", \"resourceId\": \"{Object id of service principal from step 2}\", \"appRoleId\": \"{Role id from step 3}\"}'

// Example
az rest `
    --method POST `
    --url "https://graph.microsoft.com/v1.0/servicePrincipals/3f4e5d6c-7a8b-4c9d-0e1f-2a3b4c5d6e7f/appRoleAssignments" `
    --headers "Content-Type=application/json" `
    --body '{\"principalId\": \"3f4e5d6c-7a8b-4c9d-0e1f-2a3b4c5d6e7f\", \"resourceId\": \"9dfddf8c-3e16-435b-b93b-5d421aa134eb\", \"appRoleId\": \"1a2b3c4d-5e6f-7a8b-9c0d-1e2f3a4b5c6d\"}'
```
