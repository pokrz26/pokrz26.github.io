---
title: PowerShell usefull tips
date: 2026-08-11 18:45:00 +0200
categories: [Workspace, Terminal, PowerShell]
tags: [powershell, json, troubleshooting]
description: Why ConvertFrom-Json can flatten single-item arrays and how to preserve array behavior with -NoEnumerate.
---

## Document Information

- **PowerShell Version:** 7.6.4

## When JSON arrays stop behaving like arrays

While parsing JSON in PowerShell, you may expect an array every time the JSON contains `[]`. A common edge case appears when the array has only one item.

Without extra handling, `ConvertFrom-Json` can return a scalar value instead of an array-like object.

### Reproduce the issue

```powershell
$json = '["only-one"]'
$result = $json | ConvertFrom-Json

$result.GetType().Name
```

Expected behavior for many scripts is `Object[]`, but in this case PowerShell can return `String`.

That difference can break logic like loops, index access, or validations that assume an array.

### Fix for PowerShell 7.3+ with `-NoEnumerate`

```powershell
$json = '["only-one"]'
$result = $json | ConvertFrom-Json -NoEnumerate

$result.GetType().Name
```

With `-NoEnumerate`, the result keeps array semantics even when there is only one element.
