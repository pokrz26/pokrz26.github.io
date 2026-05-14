Access repo in different project when limit project is on

Cross-project repo checkout fix (Azure DevOps)


resources:
  repositories:
    - repository: shared
      type: git
      name: OtherProject/RepoName

steps:
- checkout: self
- checkout: shared

2. Confirm correct identity

In target project, you must use:

<YourPipelineProject> Build Service (Org)

Not:

Readers group
Project Collection Build Service (unless using collection scope)


3. Give project-level permission (critical)

Target project:

Project Settings → Permissions

Find build service identity and allow:

✔ View project-level information

4. Give repo permission

Target project:

Repos → Repo → Security

Allow:

✔ Read

5. Authorize pipeline on first run

If prompted:

Click “Authorize resources”
6. Don’t rely on groups

Avoid depending on:

Readers group
inherited permissions

Always test with direct identity permissions.

Quick mental model

If it fails again, check in this order:

YAML resources.repositories present?
Repo permission granted?
Project-level permission granted?
Correct Build Service identity used?
Pipeline authorized after first run?

If you follow that order, you’ll fix it in under 2 minutes next time.