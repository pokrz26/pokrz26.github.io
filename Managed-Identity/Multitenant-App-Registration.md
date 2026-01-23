# Registering multitenant app registration

## 1. Select source tenant

If tenant has at least one subscription.

```PowerShell
// Get subscription id
az account list --all
az account list --all --query "[?contains(name, '${subscription_name_or_part_of_the_name}')].{Id:id, Name:name}"
az account list --all | ConvertFrom-Json | Select id, name | Select-String '${subscription_name_or_part_of_the_name}'

// Set subscription id
az account set -s ${subscription_id}
```

If there's no subscription.

```PowerShell
az account tenant list
az login --tenant ${tenant_id} --allow-no-subscriptions
```

## 2. Change app registration audience to multi tenant

Find app registration.

```PowerShell
az ad app list --filter "displayName eq '${app_registration_name}'" --query '[0].id'
```

Update app registration.

```PowerShell
az ad app update --id ${app_registration_client_id} --sign-in-audience AzureADMultipleOrgs
```

## 3. Switch to destination tenant

Same steps as in 1. Select source tenant.

## 4. Create service principal

Create service principal for the app registration in the destination tenant.

```PowerShell
az ad sp create --id ${app_registration_client_id}
```
