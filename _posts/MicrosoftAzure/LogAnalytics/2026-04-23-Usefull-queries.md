---
title: Useful Log Analytics Queries
date: 2026-04-23 14:04:00 +0200
categories: [Microsoft Azure, Log Analytics]
tags: [log analytics, azure, kusto, queries]
description: A collection of useful Log Analytics queries for managing and analyzing Azure diagnostics.
---

## AzureDiagnostics

### Get diagnostic grouped by rules and hosts

```kusto
AzureDiagnostics
| summarize by host_s, ruleName_s
```

### Get diagnostic grouped by rules for specific host

```kusto
AzureDiagnostics
| where host_s == '<host-name>'
| summarize by ruleName_s
```

### Get diagnostics fer specific host and rule

```kusto
AzureDiagnostics
| where host_s == '<host-name>'
| where ruleName_s == '<rule-name>'
```
