# Project Guidelines

## Conventions

When editing markdown posts, use angle-bracket placeholders for sample command arguments instead of literal environment-specific names.

Examples:

- Use `<namespace-name>` instead of a real namespace such as `some-namespace`
- Use `<resource-group-name>` and `<aks-cluster-name>` for Azure CLI samples
- Use `<pod-name>`, `<job-name>`, `<context-name>`, and similar placeholders for Kubernetes examples

Keep placeholder names descriptive and consistent with the command they are used in.

When adding informational or warning callouts to markdown posts, use the existing prompt block style instead of inline labels such as `Warning:`.

Examples:

- Use `{: .prompt-tip }` for tips and best practices
- Use `{: .prompt-info }` for informational notes
- Use `{: .prompt-warning }` for warnings, especially for destructive commands such as `--force` deletes
- Use `{: .prompt-danger }` for critical warnings, especially for commands that can cause data loss
- Wrap these callouts with `<!-- markdownlint-capture -->`, `<!-- markdownlint-disable -->`, and `<!-- markdownlint-restore -->` comments

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
