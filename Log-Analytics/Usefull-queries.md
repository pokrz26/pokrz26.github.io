# Log Analytics usefull queries

## Document Information

- **Last Updated:** March 06, 2026

### AzureDiagnostics

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
