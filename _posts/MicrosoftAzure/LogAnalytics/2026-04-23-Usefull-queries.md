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
| summarize count() by host_s, ruleName_s
```

### Get diagnostic grouped by rules for specific host

```sql
AzureDiagnostics
| where host_s == '<host-name>'
| summarize count() by ruleName_s
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

### Get diagnostics for a specific date range

```sql
AzureDiagnostics
| where TimeGenerated between (datetime(2026-09-01) .. datetime(2026-09-30))
| where host_s == '<host-name>'
| where ruleName_s == '<rule-name>'
| summarize count() by requestUri_s, details_msg_s, details_data_s, details_matches_s, action_s
```

### Get diagnostics for hosts a specific date range

```sql
AzureDiagnostics
| where TimeGenerated between (datetime(2026-09-01) .. datetime(2026-09-30))
| where host_s in ('<host-name-1>', '<host-name-2>', '<host-name-3>', '<host-name-4>')
| where ruleName_s == '<rule-name>'
| summarize count() by requestUri_s, details_msg_s, details_data_s, details_matches_s, action_s
```
