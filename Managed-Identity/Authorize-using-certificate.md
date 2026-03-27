# Assign Service Principal as User in Another App Registration

This guide explains how to register a service principal as a user with an assigned role in another app registration within the same Azure AD tenant.

## Document Information

- **Last Updated:** March 27, 2026
- **Azure CLI Version:** 2.80.0
- **PowerShell Version:** 7.6.0
- **.NET SDK Version:** 10.0 (for C# examples)

## 1. Generate self signed certificate

### Run as user

If you generate the certificate as a user, you can use the following command:

```powershell
runas /user:<domain-name>\<username> powershell
```

### Generate the certificate

```powershell
$certname = "<your-cert-name>" 
$cert = New-SelfSignedCertificate `
  -Subject "CN=$certname" `
  -CertStoreLocation "Cert:\CurrentUser\My" `
  -KeyExportPolicy Exportable `
  -KeySpec Signature `
  -KeyLength 2048 `
  -KeyAlgorithm RSA `
  -HashAlgorithm SHA256
Export-Certificate `
  -Cert $cert `
  -FilePath "<path-to-save-certificate>\$certname.cer"
```

### Get generated certificate thumbprint

```powershell
Get-ChildItem Cert:\CurrentUser\My |
  Where-Object { $_.Subject -like "*CN=$certname*" } |
  Select-Object Subject, Thumbprint
```

## 2. Upload the certificate to Azure AD app registration

To do this got to Azure Portal -> Azure Active Directory -> App registrations -> Your app registration -> Certificates & secrets -> Certificates -> Upload certificate.

## 3. Example C# code to authenticate using the certificate

### appsettings.json

Remember to set this file to `Copy if newer` in the file properties to ensure it is copied to the output directory.

```json
{
  "ClientOptions": {
    "ClientId": "<your-client-id>",
    "TenantId": "<your-tenant-id>",
    "Scope": "<your-scope>",
    "CertificateThumbPrint": "<your-certificate-thumbprint>",
    "ApiUrl": "<your-api-url>"
  }
}
```

### ClientOptions.cs

```csharp
namespace ConsoleApp1;

public class ClientOptions
{
    public required string ClientId { get; init; }
    public required string TenantId { get; init; }
    public required string Scope { get; init; }
    public required string CertificateThumbPrint { get; init; }
    public required string ApiUrl { get; init; }
}
```

### ClientTokenProvider.cs

```csharp
using Azure.Core;
using Azure.Identity;
using Microsoft.Extensions.Options;

namespace ConsoleApp1;

public interface IClientTokenProvider
{
    Task<string> GetToken(CancellationToken cancellationToken = default);
}

public class ClientTokenProvider : IClientTokenProvider
{
    private readonly ClientOptions _options;
    private readonly TokenCredential _tokenCreential;

    public ClientTokenProvider(IOptions<ClientOptions> options)
    {
        _options = options.Value;
        _tokenCreential = new ClientCertificateCredential(
            tenantId: _options.TenantId,
            clientId: _options.ClientId,
            clientCertificatePath: $"Cert:/CurrentUser/My/{_options.CertificateThumbPrint}");
    }

    public async Task<string> GetToken(CancellationToken cancellationToken = default)
    {
        var token = await _tokenCreential.GetTokenAsync(
            new TokenRequestContext([_options.Scope]),
            cancellationToken);

        return token.Token;
    }
}
```

### Program.cs

```csharp
using ConsoleApp1;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Options;

var configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json")
    .Build();
var serviceCollection = new ServiceCollection()
    .AddHttpClient()
    .AddSingleton(configuration);
serviceCollection
    .AddOptions<FerfsClientOptions>()
    .Bind(configuration.GetSection(nameof(ClientOptions)));
serviceCollection
    .AddSingleton<IClientTokenProvider, ClientTokenProvider>()
    .AddSingleton(sp =>
    {
        var config = sp.GetRequiredService<IOptions<ClientOptions>>();

        return new ClientConfiguration(config.Value.ApiUrl);
    })
    .AddSingleton<IClient, Client>()    // This is example client that uses the token provider to call API.
    .BuildServiceProvider();

var serviceProvider = serviceCollection.BuildServiceProvider();
var client = serviceProvider.GetRequiredService<IClient>();
var guids = await client.GetProgramGuids("example");

Console.WriteLine(guids.Count());
```
