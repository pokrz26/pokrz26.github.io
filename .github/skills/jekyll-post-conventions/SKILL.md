---
applyTo:
  - "**/_posts/**/*.md"
  - "**/about.md"
  - "**/archives.md"
  - "**/categories.md"
  - "**/tags.md"
---

# Jekyll Markdown Post Conventions

This skill provides formatting guidelines for creating and editing Jekyll markdown posts in this repository.

## YAML Frontmatter

Every markdown file must start with a YAML frontmatter block containing:

```yaml
---
title: File title
date: 2026-05-14 20:10:00 +0200
categories: [Parent, Child, Sub-child]
tags: [tag 1, tag 2]
description: Description of the file content.
---
```

### Frontmatter Fields

- **title** — Post title (descriptive, concise)
- **date** — Last update date in format `YYYY-MM-DD HH:MM:SS +ZZZZ`
- **categories** — Hierarchical category array (Parent, Child, Sub-child as needed)
- **tags** — Comma-separated tags for content indexing
- **description** — Brief summary of post content

## Placeholder Conventions

When writing command examples, use angle-bracket placeholders for environment-specific values instead of literal names.

### Azure Placeholders

- `<resource-group-name>` — Azure resource group
- `<storage-account-name>` — Azure Storage account
- `<container-app-name>` — Container App name
- `<container-apps-environment-name>` — Container Apps Environment
- `<container-registry-name>` — Azure Container Registry
- `<log-analytics-workspace-id>` — Log Analytics Workspace ID
- `<subscription-id>` — Azure subscription ID
- `<location>` — Azure region (e.g., eastus)

### Kubernetes Placeholders

- `<namespace-name>` — Kubernetes namespace
- `<cluster-name>` or `<aks-cluster-name>` — AKS cluster name
- `<pod-name>` — Kubernetes pod
- `<job-name>` — Kubernetes job
- `<context-name>` — kubectl context
- `<service-name>` — Kubernetes service

### Generic Placeholders

- `<environment-name>` — Environment identifier
- `<resource-name>` — Generic resource name
- `<resource-type>` — Type of resource
- `<region>` — Geographic region (when not Azure-specific)

### Placeholder Naming Rules

- Keep placeholder names **descriptive and consistent** with the context they're used in
- Match the exact service or resource type in the name
- Use lowercase with hyphens (e.g., `<container-app-name>`, not `<ContainerAppName>`)

## Code Block Formatting

### PowerShell Scripts

```powershell
# Use backticks for line continuation
az containerapp logs show `
  --name $appName `
  --resource-group $resourceGroup `
  --revision $revisionName `
  --type console
```

### Indentation

- Use **2 spaces** for indentation in all scripts
- Never use tabs

### JSON in PowerShell

Use double-quoted JSON for PowerShell variable assignments:

```powershell
$body = "{'name':'<resource-name>','type':'<resource-type>'}"
```

## Callout Blocks

Use Jekyll prompt block syntax for informational callouts. Always wrap with markdownlint comments.

### Callout Types

#### Information Callout

```markdown
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> Configuration must be set on the Container Apps Environment, not individual apps.
> {: .prompt-info }

<!-- markdownlint-restore -->
```

Use `{: .prompt-info }` for:

- Important contextual notes
- Clarifications about feature behavior
- Setup prerequisites

#### Tip Callout

```markdown
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> Use the `--output json` flag for programmatic parsing of Azure CLI results.
> {: .prompt-tip }

<!-- markdownlint-restore -->
```

Use `{: .prompt-tip }` for:

- Best practices
- Performance optimization tips
- Helpful shortcuts or alternatives

#### Warning Callout

```markdown
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> Non-root containers cannot bind to privileged ports (below 1024). Use port 8080 or higher.
> {: .prompt-warning }

<!-- markdownlint-restore -->
```

Use `{: .prompt-warning }` for:

- Destructive operations (e.g., `--force` deletes)
- Configuration pitfalls
- Common errors to avoid
- Security considerations

#### Danger Callout

```markdown
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> This command permanently deletes all data. There is no recovery.
> {: .prompt-danger }

<!-- markdownlint-restore -->
```

Use `{: .prompt-danger }` for:

- Data loss risks
- Irreversible operations
- Critical warnings

## Post Structure Best Practices

1. **Start with Document Information** — Include tool/API versions relevant to the post
2. **Prerequisites Section** — Variables or setup needed before running commands
3. **Sequential Steps** — Number major workflow steps clearly
4. **Code Examples** — Follow placeholder and formatting conventions
5. **Callouts** — Add contextual notes at decision points
6. **Subheadings** — Use `##` for major sections, `###` for subsections

## File Heading Example

```markdown
---
title: Troubleshoot Azure Container App Activation Failures
date: 2026-08-11 18:20:00 +0200
categories: [Microsoft Azure, Container Apps]
tags: [azure, container-apps, aca, azure-cli, troubleshooting]
description: How to troubleshoot why an Azure Container App revision does not start by checking revision status, details, logs, and platform events.
---

## Document Information

- **Azure CLI Version:** 2.84.0

## Prerequisites

Set your variables before running commands...
```
