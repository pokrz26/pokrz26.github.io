---
title: Useful Log Analytics Queries
date: 2026-07-31 09:03:00 +0200
categories: [Microsoft Azure, Log Analytics]
tags: [log analytics, azure, kusto, queries]
description: A collection of useful Log Analytics queries for managing and analyzing Azure diagnostics.
---

## AzureDiagnostics

### Get diagnostic grouped by rules and hosts

```sql
AzureDiagnostics
| summarize by host_s, ruleName_s
```

### Get diagnostic grouped by rules for specific host

```sql
AzureDiagnostics
| where host_s == '<host-name>'
| summarize by ruleName_s
```

### Get diagnostics fer specific host and rule

```sql
AzureDiagnostics
| where host_s == '<host-name>'
| where ruleName_s == '<rule-name>'
```

### Get rule hits grouped by request details

```sql
AzureDiagnostics
| where host_s == '<host-name>'
| where ruleName_s == '<rule-name>'
| summarize count() by requestUri_s, details_msg_s, details_data_s, details_matches_s
```
