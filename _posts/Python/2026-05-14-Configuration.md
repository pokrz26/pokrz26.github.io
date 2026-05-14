---
title: Python Configuration
date: 2026-05-14 20:00:00 +0200
categories: [Python]
tags: [python, configuration]
description: This guide explains how to configure Python and fix missing script commands.
---

## Document Information

- **PowerShell Version:** 7.6.1
- **Python:** 3.13

## Fix Missing Python Script Commands

If Python is installed from the Microsoft Store, commands installed by `pip` may not be available in your terminal because the user-level `Scripts` folder is missing from `PATH`.

To fix this, add the `Scripts` folder to the current PowerShell session.

### Find your Python install location

```powershell
py -c "import sys; print(sys.executable)"
```

The command returns a path similar to `C:\Users\<user-name>\AppData\Local\Programs\Python\Python313\python.exe`.

Replace `python.exe` with `Scripts`, then add that folder to `PATH` as shown below.

```powershell
$env:Path = "$HOME\AppData\Local\Programs\Python\Python313\Scripts;$env:Path"
```

This is useful when tools installed with `pip`, such as package-specific CLI commands, are not recognized.

To verify that the path was added correctly, run:

```powershell
$env:Path -split ';' | Select-String 'Python313\\Scripts'
```
