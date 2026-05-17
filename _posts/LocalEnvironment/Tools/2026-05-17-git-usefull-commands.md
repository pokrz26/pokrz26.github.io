---
title: git usefull commands
date: 2026-05-17 14:35:00 +0200
categories: [Local Environment, Tools]
tags: [git]
description: Here you can git rclone usefull commands.
---

## Document Information

- **PowerShell Version:** 7.6.1

## Reset head

```powershell
git reset HEAD~1
```

## Show status

```powershell
git status
```

## Show recent commits

```powershell
git log --oneline --decorate -n 10
```

## Create and switch to new branch

```powershell
git switch -c <branch-name>
```

## Switch branch

```powershell
git switch <branch-name>
```

## Unstage file

```powershell
git restore --staged <file-path>
```

## Discard unstaged changes in file

```powershell
git restore <file-path>
```

## Show changed files

```powershell
git diff --name-only
```

## Stash changes

```powershell
git stash push -m "<stash-message>"
```

## Apply and remove last stash

```powershell
git stash pop
```

## Pull with rebase

```powershell
git pull --rebase
```