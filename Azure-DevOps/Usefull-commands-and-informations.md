# Useful Azure DevOps Commands and Information

## Document Information

- **Last Updated:** March 06, 2026
- **Azure CLI Version:** 2.80.0
- **PowerShell Version:** 7.5.4
- **Azure DevOps CLI Extension Version:** 0.18.0

## Sign in with az devops login

You need to this only if you are not already logged in with `az login` or if you want to use a different account for Azure DevOps operations.

```powershell
az devops login --organization https://dev.azure.com/<organization>
```

After that prompt will ask you to enter your `PAT` (Personal Access Token) for authentication. You can also provide `PAT` token directly in the command line, but be cautious as it may expose your token in command history or logs.

```powershell
"######" | az devops login --organization https://dev.azure.com/contoso/
```

Or you can set the `AZURE_DEVOPS_EXT_PAT` environment variable with your `PAT` token before running the login command.

```powershell
$env:AZURE_DEVOPS_EXT_PAT = 'xxxxxxxxxx'
az devops login --organization https://dev.azure.com/contoso/
```

## Azure DevOps REST API

Rest API documentation is available at [Microsoft Learn](https://learn.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-7.1).

## Get feed id

```powershell
# Project level feed
az rest --method get --uri "https://pkgs.dev.azure.com/<organization>/<project>/_apis/packaging/feeds?api-version=7.1-preview.1"
# Organization level feed
az rest --method get --uri "https://pkgs.dev.azure.com/<organization>/_apis/packaging/feeds?api-version=7.1-preview.1"
```

## Download package from Azure Artifacts

More info in [Microsoft Learn](https://learn.microsoft.com/en-us/rest/api/azure/devops/artifactspackagetypes/nuget/download-package).

```powershell
# Project level feed
az rest --method get --uri "https://pkgs.dev.azure.com/<organization>/<project>/_apis/packaging/feeds/<feed>/packages?api-version=7.1-preview.1"
# Organization level feed
az rest --method get --uri "https://pkgs.dev.azure.com/<organization>/_apis/packaging/feeds/<feed>/packages?api-version=7.1-preview.1"
```

## Get package content

```powershell
## Project level feed
az rest --method get --uri "https://pkgs.dev.azure.com/<organization>/<project>/_apis/packaging/feeds/<feed>/packages/<packageId>/versions/<versionId>/content?api-version=7.1-preview.1"
## Organization level feed
az rest --method get --uri "https://pkgs.dev.azure.com/<organization>/_apis/packaging/feeds/<feed>/packages/<packageId>/versions/<versionId>/content?api-version=7.1-preview.1"
```
