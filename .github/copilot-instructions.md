# Project Guidelines

## Conventions
When editing markdown posts, use angle-bracket placeholders for sample command arguments instead of literal environment-specific names.

Examples:
- Use `<namespace-name>` instead of a real namespace such as `some-namespace`
- Use `<resource-group-name>` and `<aks-cluster-name>` for Azure CLI samples
- Use `<pod-name>`, `<job-name>`, `<context-name>`, and similar placeholders for Kubernetes examples

Keep placeholder names descriptive and consistent with the command they are used in.

## File heading 
At the top of each markdown file, include text section as below.

---
title: File title
date: 2026-05-14 20:10:00 +0200
categories: [Parent, Child, Sub-child]
tags: [tag 1, tag 2]
description: Description of the file content.
---

Make sure to update the title, date, categories, tags, and description according to the content of the file. The date should reflect the last update time of the file.
