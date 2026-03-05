# Assign Service Principal as User in Another App Registration

This guide explains how to register a service principal as a user with an assigned role in another app registration within the same Azure AD tenant.

## Document Information

- **Last Updated:** January 23, 2026
- **Azure CLI Version:** 2.80.0
- **PowerShell Version:** 7.5.4
- **.NET SDK Version:** 8.0 (for C# examples)

## Prerequisites

- Both service principals must be registered in the same Azure AD tenant
- You must have sufficient permissions to manage app registrations and service principals
- Azure CLI installed and authenticated
- For cross-tenant scenarios, configure the app as multitenant first

## Configuration steps

### Step 1: Retrieve the Source Service Principal Object ID

Get the object ID of the service principal that will be granted access to the target application.

```powershell
$sourceAppObjectId = az ad sp list `
    --filter "displayName eq '<source-app-name>' and servicePrincipalType eq 'Application'" `
    --query '[0].id' `
    --output tsv
```

### Step 2: Retrieve the Target Service Principal Object ID

Get the object ID of the target application where the role assignment will be created.

```powershell
$targetAppObjectId = az ad sp list `
    --filter "displayName eq '<target-app-name>' and servicePrincipalType eq 'Application'" `
    --query '[0].id' `
    --output tsv
```

### Step 3: List Available App Roles

Retrieve the available app roles from the target application to identify the appropriate role ID.

```powershell
az ad sp show `
    --id $targetAppObjectId `
    --query 'appRoles' `
    | ConvertFrom-Json `
    | Select-Object displayName, value, id `
    | Format-Table
```

### Step 4: Create the App Role Assignment

Assign the selected role to the source service principal in the context of the target application.

```powershell
az rest `
    --method POST `
    --url "https://graph.microsoft.com/v1.0/servicePrincipals/${sourceAppObjectId}/appRoleAssignments" `
    --headers "Content-Type=application/json" `
    --body '{\"principalId\": \"'${sourceAppObjectId}'\", \"resourceId\": \"'${targetAppObjectId}'\", \"appRoleId\": \"<role-id>\"}'
```

## Verification

To verify the role assignment was successful:

```powershell
az rest `
    --method GET `
    --url "https://graph.microsoft.com/v1.0/servicePrincipals/${sourceAppObjectId}/appRoleAssignments"
```

## Example Usage in C\#

This example demonstrates how to use federated identity authentication in a C# application to access resources across multiple Azure AD tenants.

### Configure Settings and HttpClient

First, set up configuration management to read settings from environment variables, user secrets, or configuration files:

```csharp
using Microsoft.Extensions.Configuration;

namespace YourApp.Test.System.Helpers;

internal class Settings
{
    private static readonly HttpClient _httpClient = new();
    private static readonly Lazy<IConfiguration> Configuration = new(() =>
    {
        return new ConfigurationBuilder()
            .AddEnvironmentVariables()
#if DEBUG
            .AddUserSecrets<Settings>()
            .AddJsonFile("local.settings.json")
            //.AddJsonStream(new MemoryStream(Encoding.UTF8.GetBytes(@"{
            //    ""ENV_AppUrl"": ""https://appurl.com""
            //}")))
#endif
            .Build();
    });

    static Settings()
    {
        _httpClient.BaseAddress = new Uri(AppUrl.TrimEnd('/'));
        _httpClient.Timeout = TimeSpan.FromSeconds(30);

        _httpClient.DefaultRequestHeaders.Clear();
        _httpClient.DefaultRequestHeaders.Add("Accept", "application/json");
        _httpClient.DefaultRequestHeaders.Add("User-Agent", "MyApp-TestClient");
    }

    public static string AppUrl = Configuration.Value["ENV_AppUrl"] ?? throw new ArgumentNullException($"No value for ${nameof(AppUrl)}");
    public static string Scope = Configuration.Value["ENV_Scope"] ?? throw new ArgumentNullException($"No value for ${nameof(Scope)}");
    public static string FirstTenantId = Configuration.Value["ENV_FirstTenantId"] ?? throw new ArgumentNullException($"No value for ${nameof(FirstTenantId)}");
    public static string SecondTenantId = Configuration.Value["ENV_SecondTenantId"] ?? throw new ArgumentNullException($"No value for ${nameof(SecondTenantId)}");
    public static string ClientId = Configuration.Value["ENV_ClientId"] ?? throw new ArgumentNullException($"No value for ${nameof(ClientId)}");
    public static HttpClient HttpClient => _httpClient;
}
```

### Retrieve Federated Identity Token

Create an extension method to authenticate HTTP requests using `DefaultAzureCredential` with cross-tenant support:

```csharp
using Azure.Core;
using Azure.Identity;
using System.Net.Http.Headers;

namespace YourApp.Test.System.Helpers;

internal static class HttpRequestMessageExtensions
{
    private static readonly DefaultAzureCredential _defaultAzureCredential = new(new DefaultAzureCredentialOptions
    {
        TenantId = Settings.SecondTenantId,
        AdditionallyAllowedTenants = { Settings.FirstTenantId }
    });
    private static readonly TokenRequestContext _tokenRequestContext = new([Settings.Scope]);

    public static async Task<HttpRequestMessage> WithFederatedIdentityToken(this HttpRequestMessage request, CancellationToken cancellationToken = default)
    {
        var accessToken = await _defaultAzureCredential.GetTokenAsync(_tokenRequestContext, cancellationToken);

        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken.Token);

        return request;
    }
}
```
