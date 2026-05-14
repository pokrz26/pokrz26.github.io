---
title: Cross-Project repo checkout
date: 2026-05-14 20:20:00 +0200
categories: [Azure DevOps]
tags: [azure-devops, azure-pipelines, repository, checkout, permissions]
description: This guide explains how to fix Azure DevOps cross-project repository checkout issues by configuring repository resources, build service permissions, and pipeline authorization.
---

## Problem

When an Azure DevOps pipeline checks out a repository from another project, the checkout may fail even when the repository path is correct. In most cases, the failure is caused by missing permissions on the target project or repository.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The required configuration depends on whether the `Limit job authorization scope to current project for non-release pipelines` setting is enabled or disabled.
{: .prompt-info }
<!-- markdownlint-restore -->

## 1. Define the external repository in the pipeline

Add the target repository to `resources.repositories` and then check it out explicitly.

```yaml
resources:
  repositories:
    - repository: shared
      type: git
      name: OtherProject/RepoName

steps:
  - checkout: self
  - checkout: shared
```

## 2. Use the correct build identity

If the `Limit job authorization scope to current project for non-release pipelines` setting is enabled in **Project Settings** -> **Settings**, use the project-scoped build identity.

```text
<project-name> Build Service (<org-name>)
```

Otherwise, use the collection-scoped build identity.

```text
Project Collection Build Service (<org-name>)
```

## 3. Grant project-level permission

If you use the project-scoped build identity, grant it the required permission in the target project under **Project Settings** -> **Permissions**.

Find the build service identity and allow:

- `View project-level information`

Adding the build identity to the `Readers` group may also be enough.

This permission is required even if repository permissions already look correct.

## 4. Grant repository permission

In the target project, go to **Repos** -> **Repo** -> **Security**.

Allow:

- `Read`

Grant this directly to the correct build service identity.

## 5. Authorize the pipeline on first run

On the first run, Azure DevOps may ask for resource authorization.

If prompted, select:

- `Authorize resources`

## Troubleshooting order

If the checkout still fails, verify these items in order:

1. `resources.repositories` is defined in YAML.
2. The target repository grants `Read` permission.
3. The target project grants `View project-level information`, or the build identity is in a group such as `Readers`.
4. The correct build service identity is used.
5. The pipeline was authorized after the first run.

Following that sequence usually isolates the problem quickly.