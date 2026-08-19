---
title: Troubleshoot SSL/TLS certificate validation failures in Azure App Service
date: 2026-08-19 14:45:00 +0200
categories: [Microsoft Azure, App Service]
tags: [azure, app-service, dotnet, ssl, tls, certificates, troubleshooting]
description: How to validate and troubleshoot SSL/TLS failures in Azure App Service caused by a missing root certificate, with Linux and Windows validation steps and practical fixes.
---

## Document Information

- **Runtime:** .NET 6 and later, plus OpenSSL-based services running in Azure App Service
- **Operating System:** Linux-based Azure App Service containers
- **Environment:** Azure App Service / Linux web app
- **Focus:** Root certificate validation and trust store troubleshooting in Azure-hosted workloads

## Problem description

An application can fail to connect to an HTTPS endpoint even when the remote server is reachable and the certificate looks valid. Usually the problem is not the application code itself, but a missing or outdated root certificate in the operating system trust store or in the runtime environment.

This issue can happen on any platform, and it often appears as one of these errors:

- `The remote certificate is invalid according to the validation procedure`
- `SSL/TLS handshake failed`
- `The certificate chain was issued by an authority that is not trusted`
- `unable to get local issuer certificate`

## 1. Linux solution

Use the commands below when the service is running on Linux, a container, or an Azure App Service container.

### 1.1 Validate the certificate chain and trust relationship

```bash
# Connect to the remote host and print the full certificate chain as presented by the server
openssl s_client \
  -connect <hostname>:443 \
  -servername <hostname> \
  -showcerts

# Validate the connection using the system CA bundle
openssl s_client \
  -connect <hostname>:443 \
  -servername <hostname> \
  -CAfile /etc/ssl/certs/ca-certificates.crt

# Validate the connection using the default CA directory
openssl s_client \
  -connect <hostname>:443 \
  -servername <hostname> \
  -CApath /etc/ssl/certs

# Show the TLS handshake details from curl to compare the result with OpenSSL
curl -v https://<hostname>
```

### 1.2 Check whether the required root certificate is installed

```bash
# Search the trusted cert store for the expected issuer name
grep -R "Certum Trusted Root CA" /etc/ssl/certs

# List files that look related to the root certificate
ls -l /etc/ssl/certs | grep -i certum

# Print subjects of installed certificates and filter for the expected root issuer
find /etc/ssl/certs -type f -exec openssl x509 -noout -subject {} \; | grep "Certum Trusted Root CA"

# Check whether the root certificate is present in the combined bundle file
grep -i "Certum Trusted Root CA" /etc/ssl/certs/ca-certificates.crt
```

If the expected issuer is missing, the machine is missing a trusted root certificate and TLS validation fails even though the server is online.

### 1.3 Check the OS and certificate package state

```bash
# Verify whether the CA certificates package is installed and up to date
dpkg -l ca-certificates

# Check the Linux distribution version that is running
cat /etc/os-release
```

## 2. Windows solution

Use the commands below when the service is running locally on Windows, in a Windows host, or in a .NET diagnostics session.

### 2.1 Inspect the certificate and full chain

```powershell
$hostName = "<hostname>"

$tcp = New-Object System.Net.Sockets.TcpClient($hostName, 443)

$ssl = New-Object System.Net.Security.SslStream($tcp.GetStream(), $false, {
    param($sender, $certificate, $chain, $errors)

    Write-Host "===== CERTIFICATE ====="
    Write-Host "Subject:    $($certificate.Subject)"
    Write-Host "Issuer:     $($certificate.Issuer)"
    Write-Host "Thumbprint: $($certificate.Thumbprint)"
    Write-Host "Errors:     $errors"
    Write-Host ""

    if ($chain) {
        Write-Host "===== CHAIN ====="

        foreach ($element in $chain.ChainElements) {
            Write-Host "Subject:    $($element.Certificate.Subject)"
            Write-Host "Issuer:     $($element.Certificate.Issuer)"
            Write-Host "Thumbprint: $($element.Certificate.Thumbprint)"
            Write-Host "NotBefore:  $($element.Certificate.NotBefore)"
            Write-Host "NotAfter:   $($element.Certificate.NotAfter)"
            Write-Host "Status:     $($element.ChainElementStatus.Status)"
            Write-Host ""
        }
    }

    return $true
})

$ssl.AuthenticateAsClient($hostName)
```

What each part does:

- `TcpClient` opens a socket to the target host on port 443.
- `SslStream` performs the TLS handshake and gives access to the certificate and chain.
- The callback prints the leaf certificate and every issuer in the chain.
- `AuthenticateAsClient` starts the validation process and triggers the callback.

This is useful because it shows which certificate in the chain fails to validate and which issuer is missing from the local trust store.

### 2.2 Inspect `SslPolicyErrors` and chain status

```powershell
$hostName = "<hostname>"

$tcp = New-Object System.Net.Sockets.TcpClient($hostName, 443)

$ssl = New-Object System.Net.Security.SslStream(
    $tcp.GetStream(),
    $false,
    {
        param($sender, $certificate, $chain, $errors)

        Write-Host "SslPolicyErrors = $errors"

        if ($chain) {
            Write-Host "ChainStatus:"
            $chain.ChainStatus | ForEach-Object {
                Write-Host "  $($_.Status) - $($_.StatusInformation)"
            }
        }

        return $true
    }
)

$ssl.AuthenticateAsClient($hostName)
```

What each part does:

- `SslPolicyErrors` tells you whether .NET rejected the certificate because of validation, chain, or name issues.
- `ChainStatus` shows the exact reason the chain is not trusted.
- `ForEach-Object` prints each chain status in a readable format.

This is often the fastest way to confirm whether the problem is a missing root certificate or a different certificate issue.

## 3. How to interpret the results

The key question is always the same: is the certificate chain valid, but the local machine does not trust one of the issuers?

If you see results like these, the problem is usually a trust-store issue:

- `unable to get local issuer certificate`
- `certificate verify failed`
- `The certificate chain was issued by an authority that is not trusted`
- `ChainStatus` includes `UntrustedRoot`

This means the server certificate may be correct, but one of the root or intermediate certificates is missing from the environment.

## 4. Fix options

### Option A: update the trust store

This depends on the operating system.

#### Linux

If the issue is caused by an outdated base image or incomplete CA bundle, update the system certificate package.

```bash
apt-get update
apt-get install -y ca-certificates
update-ca-certificates
```

What each command does:

- `apt-get update` refreshes package metadata.
- `apt-get install -y ca-certificates` installs or updates the CA bundle.
- `update-ca-certificates` rebuilds the system trust store so the new root certificates are available.

This is the correct fix when the problem is caused by a stale root certificate bundle in Linux.

#### Windows

On Windows, the equivalent action is to install the root certificate into the Trusted Root Certification Authorities store for the local machine or current user.

```powershell
certutil -addstore -f Root "C:\\path\\to\\root-ca.cer"
```

What this command does:

- `certutil` is the Windows certificate management utility.
- `-addstore Root` imports the certificate into the Trusted Root Certification Authorities store.
- `-f` forces the import if the certificate already exists.

You can also do the same manually in MMC by importing the certificate into the `Trusted Root Certification Authorities` store.

### Option B: upload a custom certificate for private or enterprise CAs

If the application must trust a private CA or a custom internal root certificate, upload it in Azure and load it from the app configuration.

1. Go to Azure Portal.
2. Open the App Service instance.
3. Go to **Certificates**.
4. Upload the `.cer` file for the root CA and, if needed, the intermediate CA.
5. Add an application setting:

```text
WEBSITE_LOAD_CERTIFICATES=*
```

Or limit it to specific certificates:

```text
WEBSITE_LOAD_CERTIFICATES=<thumbprint1>,<thumbprint2>
```

6. Restart the app.

This is the right approach when the environment needs to trust a corporate or private CA that is not included in the standard trust store.

### Option C: update the .NET runtime to a newer supported version

If the app is running on an older .NET runtime, such as .NET 6 on an older Linux image, it may be missing a newer root certificate that is already available in more recent runtime stacks or newer base images. In that case, upgrading the app to a newer supported .NET version can resolve the issue without custom certificate workarounds.

For Azure App Service, the practical approach is:

1. Check which .NET version the app is running on.
2. Upgrade the app to the latest supported LTS runtime available in the App Service stack.
3. Redeploy and validate the outbound HTTPS connection again.

This is especially relevant when the root certificate issue is tied to an old runtime image or a certificate store that has not been refreshed for that platform version.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> A missing root certificate is usually a trust-store issue, not an application logic issue.
{: .prompt-info }
<!-- markdownlint-restore -->
