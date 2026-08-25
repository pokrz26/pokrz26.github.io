---
title: SQL useful queries
date: 2026-08-25 10:10:00 +0200
categories: [Workspace, Tools, SQL]
tags: [sql, t-sql, mssql, database, permissions]
description: A curated set of useful SQL Server queries for permissions, role membership, and database auditing.
---

## Document Information

- **Platform:** Microsoft SQL Server / Azure SQL Database-compatible T-SQL
- **Use case:** Database permissions, security review, role membership, and object-level auditing

## Check whether permission exists

```sql
SELECT
    permission_name,
    class_desc
FROM sys.fn_builtin_permissions(DEFAULT)
WHERE permission_name IN ('VIEW DEFINITION', 'VIEW SECURITY DEFINITION')
ORDER BY permission_name;
```

## Grant or revoke security permissions

Grant user a permission.

```sql
GRANT VIEW SECURITY DEFINITION TO [user-name];
```

Remove user a permission.

```sql
REVOKE VIEW SECURITY DEFINITION FROM [user-name];
```

## Assign or drop a role for a user

Assign role to user.

```sql
ALTER ROLE db_datawriter ADD MEMBER [user-name];
```

Remove role from user.

```sql
ALTER ROLE db_datawriter DROP MEMBER [user-name]
```

## Create or drop user

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>Every database user is a member of the database-level `public` role by default. Similarly, every server login belongs to the server-level `public` role. This means a principal always has the permissions granted to `public` unless those permissions are explicitly removed or overridden by more specific grants or role membership.
{: .prompt-info }
<!-- markdownlint-restore -->

Create user without login.

```sql
CREATE USER [user-name] WITHOUT LOGIN;
```

Create user with login ability.

```sql
CREATE USER [user-name] WITH PASSWORD = '<strong_password>';
```

Create user from external provider.

```sql
CREATE USER [user-name] FROM EXTERNAL PROVIDER
```

Drop user.

```sql
DROP USER [user-name]
```

## Run command as user

```sql
EXECUTE AS USER = 'MP-READ-SQL-ROLES-TEST';

<command>

REVERT;
```

## Check database permission support dynamically

```sql
IF EXISTS (
    SELECT 1
    FROM sys.fn_builtin_permissions(DEFAULT)
    WHERE permission_name = 'VIEW SECURITY DEFINITION'
      AND class_desc = 'DATABASE'
)
BEGIN
    PRINT 'VIEW SECURITY DEFINITION is supported in this database engine.';
    GRANT VIEW SECURITY DEFINITION TO [user-name];
END
ELSE
BEGIN
    PRINT 'VIEW SECURITY DEFINITION is not supported. Use VIEW DEFINITION instead.';
    GRANT VIEW DEFINITION TO [user-name];
END
```

## Show direct and nested role memberships

```sql
WITH RoleHierarchy AS
(
    SELECT
        drm.member_principal_id AS PrincipalId,
        drm.role_principal_id AS RoleId,
        1 AS RoleDepth
    FROM sys.database_role_members drm

    UNION ALL

    SELECT
        rh.PrincipalId,
        drm.role_principal_id AS RoleId,
        rh.RoleDepth + 1
    FROM RoleHierarchy rh
    INNER JOIN sys.database_role_members drm
        ON drm.member_principal_id = rh.RoleId
    WHERE rh.RoleDepth < 100
)
SELECT
    member.name AS MemberName,
    role.name AS RoleName,
    rh.RoleDepth
FROM RoleHierarchy rh
INNER JOIN sys.database_principals member
    ON member.principal_id = rh.PrincipalId
INNER JOIN sys.database_principals role
    ON role.principal_id = rh.RoleId
ORDER BY
    member.name,
    rh.RoleDepth,
    role.name;
```

## Audit effective database permissions for principals and roles

```sql
WITH RoleHierarchy AS
(
    SELECT
        drm.member_principal_id AS PrincipalId,
        drm.role_principal_id AS RoleId,
        1 AS RoleDepth
    FROM sys.database_role_members drm

    UNION ALL

    SELECT
        rh.PrincipalId,
        drm.role_principal_id AS RoleId,
        rh.RoleDepth + 1
    FROM RoleHierarchy rh
    INNER JOIN sys.database_role_members drm
        ON drm.member_principal_id = rh.RoleId
    WHERE rh.RoleDepth < 100
),
Audit AS
(
    SELECT
        p.principal_id AS PrincipalId,
        p.name AS PrincipalName,
        p.type_desc AS PrincipalType,
        p.authentication_type_desc AS AuthenticationType,
        sp.name AS LoginName,
        sp.type_desc AS LoginType,
        sp.is_disabled AS IsLoginDisabled,
        NULL AS RoleId,
        NULL AS RoleName,
        NULL AS RoleDepth,
        'DIRECT' AS AccessSource,
        dp.permission_name AS PermissionName,
        dp.state_desc AS PermissionState,
        dp.class_desc AS PermissionClass,
        dp.major_id,
        dp.minor_id
    FROM sys.database_principals p
    INNER JOIN sys.database_permissions dp
        ON dp.grantee_principal_id = p.principal_id
    LEFT JOIN sys.server_principals sp
        ON sp.sid = p.sid

    UNION ALL

    SELECT
        member.principal_id AS PrincipalId,
        member.name AS PrincipalName,
        member.type_desc AS PrincipalType,
        member.authentication_type_desc AS AuthenticationType,
        sp.name AS LoginName,
        sp.type_desc AS LoginType,
        sp.is_disabled AS IsLoginDisabled,
        rh.RoleId,
        role.name AS RoleName,
        rh.RoleDepth,
        CASE
            WHEN role.is_fixed_role = 1 THEN 'FIXED_ROLE'
            WHEN rh.RoleDepth = 1 THEN 'DIRECT_ROLE'
            ELSE 'NESTED_ROLE'
        END AS AccessSource,
        dp.permission_name AS PermissionName,
        dp.state_desc AS PermissionState,
        dp.class_desc AS PermissionClass,
        dp.major_id,
        dp.minor_id
    FROM RoleHierarchy rh
    INNER JOIN sys.database_principals member
        ON member.principal_id = rh.PrincipalId
    INNER JOIN sys.database_principals role
        ON role.principal_id = rh.RoleId
    LEFT JOIN sys.server_principals sp
        ON sp.sid = member.sid
    LEFT JOIN sys.database_permissions dp
        ON dp.grantee_principal_id = role.principal_id
),
AllPrincipals AS
(
    SELECT
        p.principal_id AS PrincipalId,
        p.name AS PrincipalName,
        p.type_desc AS PrincipalType,
        p.authentication_type_desc AS AuthenticationType,
        sp.name AS LoginName,
        sp.type_desc AS LoginType,
        sp.is_disabled AS IsLoginDisabled
    FROM sys.database_principals p
    LEFT JOIN sys.server_principals sp
        ON sp.sid = p.sid
)
SELECT DISTINCT
    p.PrincipalId,
    p.PrincipalName,
    p.PrincipalType,
    p.AuthenticationType,
    p.LoginName,
    p.LoginType,
    p.IsLoginDisabled,
    a.RoleId,
    a.RoleName,
    a.RoleDepth,
    a.AccessSource,
    a.PermissionName,
    a.PermissionState,
    a.PermissionClass,
    CASE
        WHEN a.PermissionClass = 'DATABASE' THEN DB_NAME()
        WHEN a.PermissionClass = 'SCHEMA' THEN SCHEMA_NAME(a.major_id)
        WHEN a.PermissionClass IN ('OBJECT', 'OBJECT_OR_COLUMN') THEN OBJECT_SCHEMA_NAME(a.major_id)
        ELSE NULL
    END AS SchemaName,
    CASE
        WHEN a.PermissionClass IN ('OBJECT', 'OBJECT_OR_COLUMN') THEN OBJECT_NAME(a.major_id)
        ELSE NULL
    END AS ObjectName,
    CASE
        WHEN a.PermissionClass = 'OBJECT_OR_COLUMN' AND a.minor_id > 0 THEN COL_NAME(a.major_id, a.minor_id)
        ELSE NULL
    END AS ColumnName
FROM AllPrincipals p
LEFT JOIN Audit a
    ON a.PrincipalId = p.PrincipalId
ORDER BY
    p.PrincipalName,
    a.RoleName,
    a.RoleDepth,
    a.AccessSource,
    a.PermissionClass,
    SchemaName,
    ObjectName,
    ColumnName;
```

## Show all database principals and their types

```sql
SELECT
    principal_id,
    name,
    type_desc,
    authentication_type_desc,
    is_fixed_role,
    is_disabled
FROM sys.database_principals
ORDER BY name;
```

## Show all database permissions for a specific principal

```sql
SELECT
    p.name AS PrincipalName,
    p.type_desc AS PrincipalType,
    perm.permission_name,
    perm.state_desc,
    perm.class_desc,
    OBJECT_SCHEMA_NAME(perm.major_id) AS SchemaName,
    OBJECT_NAME(perm.major_id) AS ObjectName
FROM sys.database_principals p
LEFT JOIN sys.database_permissions perm
    ON perm.grantee_principal_id = p.principal_id
WHERE p.name = 'local-user-test'
ORDER BY
    perm.class_desc,
    ObjectName;
```
