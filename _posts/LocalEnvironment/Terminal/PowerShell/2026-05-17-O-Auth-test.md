---
title: PowerShell oAuth test
date: 2026-05-17 14:17:00 +0200
categories: [Local Environment, Terminal, PowerShell]
tags: [local environemnt, terminal, powershell]
description: Using PowerShell to make on oAuth authorization.
---

## Used tools and versions

- **PowerShell Version:** 7.6.1

## oAuth script

```powershell

$clientId = ""
$clientSecret = ""
$rediectUri = "http://localhost/<application-name>"

$redirectUriEncoded = [System.Web.HttpUtility]::UrlEncode($rediectUri)

$randomNumberGenerator = [Security.Cryptography.RandomNumberGenerator]::Create()
$randomBytes = [byte[]]::new(32)
$randomNumberGenerator.GetBytes($randomBytes)
$codeVerifier = [Convert]::ToBase64String($randomBytes).TrimEnd("=").Replace("+", "-").Replace("/", "_")
$codeChallenge = [Convert]::ToBase64String([Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes($codeVerifier))).TrimEnd("=").Replace("+", "-").Replace("/", "_")
$state = (New-Guid).ToString()

$url = "https://<identity-provider-host>/connect/authorize?response_type=code&client_id=$clientId&state=$state&scope=openid%20profile%20email%20<api-scope>%20offline_access&redirect_uri=$redirectUriEncoded&code_challenge=$codeChallenge&code_challenge_method=S256"

Write-Host "Provided values:"
Write-Host "Client ID: $clientId"
Write-Host "Client Secret: $clientSecret"
Write-Host "Redirect URI: $rediectUri"
Write-Host "Redirect URI Encoded: $redirectUriEncoded"
Write-Host "State: $state"
Write-Host "Code Verifier: $codeVerifier"
Write-Host "Code Challenge: $codeChallenge"
Write-Host ""
Write-Host "Now do the following steps:"
Write-Host "1. Open the following URL in your browser: $url"
Write-Host "2. Go through the login process"
Write-Host "3. Copy whole url back to console"

Write-Host ""

$returnedUrl = Read-Host -Prompt "Copy and past url that shows up in browser after login"
$code = $returnedUrl.Split("?")[1].Split("&") | Where-Object { $_ -match "^code=" } | ForEach-Object { $_.Split("=")[1] }
$tokenRequest = @{
    grant_type="authorization_code"
    client_id=$clientId
    client_secret=$clientSecret
    code=$code
    redirect_uri=$rediectUri
    code_verifier=$codeVerifier
}
$tokenReposne = Invoke-WebRequest -Uri "https://<identity-provider-host>/connect/token" -Method Post -Body $tokenRequest -ContentType "application/x-www-form-urlencoded"

Write-Host ""
Write-Host $tokenReposne

$refreshToken = $tokenReposne.Content | ConvertFrom-Json | Select-Object -ExpandProperty refresh_token
$refreshTokenRequest = @{
    grant_type="refresh_token"
    client_id=$clientId
    client_secret=$clientSecret
    refresh_token=$refreshToken
}
$newTokenReposne = Invoke-WebRequest -Uri "https://<identity-provider-host>/connect/token" -Method Post -Body $refreshTokenRequest -ContentType "application/x-www-form-urlencoded"

Write-Host ""
Write-Host $newTokenReposne
```