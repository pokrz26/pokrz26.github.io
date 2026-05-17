---
title: curl usefull Commands
date: 2026-05-17 14:30:00 +0200
categories: [Local Environment, Tools]
tags: [curl]
description: Here you can find curl usefull commands.
---

## Document Information

- **PowerShell Version:** 7.6.1

## Upload file

```powershell
curl -X PUT `
  --data-binary "@<file-name>" `
  '<upload-url>'
```

## Download file

```powershell
curl -L '<download-url>' -o '<output-file-name>'
```

## Send GET request

```powershell
curl '<request-url>'
```

## Show response headers

```powershell
curl -I '<request-url>'
```

## Send POST request with JSON body

```powershell
curl -X POST `
  -H 'Content-Type: application/json' `
  -d '{"name":"<value>"}' `
  '<request-url>'
```

## Send request with bearer token

```powershell
curl `
  -H 'Authorization: Bearer <access-token>' `
  '<request-url>'
```

## Send form data

```powershell
curl -X POST `
  -F 'file=@<file-name>' `
  -F 'description=<value>' `
  '<request-url>'
```