---
title: PowerShell usefull commands
date: 2026-04-26 12:12:00 +0200
categories: [Local Environment, Terminal, PowerShell]
tags: [local environemnt, terminal, powershell]
description: Usefull commands for powershell.
---

## Used tools and versions

- **PowerShell Version:** 7.6.1

## Find process that is using port

```powershell
Get-Process -Id (Get-NetTCPConnection -LocalPort <port>).OwningProcess
```

## Kill process by name

```powershell
Get-Process -Name <process-name> | Stop-Process
```

## Start process

```powershell
Start-Process "C:\example\path\migrationBundle.exe" `
  -WorkingDirectory "C:\example\path" `
  -NoNewWindow `
  -ArgumentList @(`
    "--verbose", `
    "--connection", `
    "`"Data Source=<sql-instance-name>;database=<database-name>;trusted_connection=yes;`"", `
    "--", `
    "`"<environment-variable-name>=<value>`""`
  )
```

## Import classes in modules

First create file that wil hold your class eg. `personClass.psm1`.

```powershell
class Person {
    [String]$Name
    [int]$Age

    Person([String]$Name, [int]$Age) {
        $this.Name = $Name
        $this.Age = $Age
    }

    [String]GreetPerson([String]$PreferredGreeting) {
        if($PreferredGreeting -eq "" -or $null -eq $PreferredGreeting) {
            $PreferredGreeting = "Hello"
        }
        return "$PreferredGreeting, $($this.Name). I can't believe you're $($this.Age) years old."
    }
}
```

In the file you want to use this class do:

```powershell
using module ./personClass.psm1

$me = [Person]::new("Tommy", <age>)
Write-Output $me.GreetPerson("Salutations")
```

## Create alias

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Alias created that way exists only in current session.
{: .prompt-info }
<!-- markdownlint-restore -->

```powershell
New-Alias -Name trivy -Value "C:\<path-to-exe>\exe_file.exe"
```