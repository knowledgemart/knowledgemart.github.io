---
title: "Managing Users, Roles & Privileges in Amazon Redshift" 
date: 2021-01-12 00:00:00 +0100
tags: [AWS, Redshift]
---

Managing access in Redshift means balancing control, collaboration, and security. Whether you’re defining data ownership, setting schema permissions, or granting privileges across teams, understanding Redshift’s access model helps keep your environment secure and maintainable.

## Understanding Privileges and Ownership
In Redshift, every database object has an owner — the user who created it. Owners automatically receive full control over their objects and can modify, drop, or delegate permissions to others.

Privileges are managed through the `GRANT` and `REVOKE` commands:

```sql
GRANT <privilege> ON <object> TO <user_or_role>;
REVOKE <privilege> ON <object> FROM <user_or_role>;
```

Supported privileges include:

- `SELECT`, `INSERT`, `UPDATE`, `DELETE`
- `REFERENCES` (for foreign key constraints)
- `CREATE` (for object creation in schemas)
- `TEMPORARY` (to allow creation of temp tables)
- `USAGE` (to access a schema)

By default, all users have `CREATE` and `USAGE` on the `public` schema unless revoked. 

## Superusers
Superusers bypass all privilege checks. They implicitly own all objects and can grant or revoke any privilege. Because of their power, avoid using the superuser role for daily tasks unrelated to security or administration.

To create a superuser (you must already be a superuser):

```sql
CREATE USER username CREATEUSER PASSWORD 'password';
```

The `CREATEUSER` keyword designates the new user as a superuser (default is `NOCREATEUSER`).

## Users & Roles
Only a superuser can create or drop users. Users can own objects, but you can’t drop a user who owns objects without first transferring or dropping those objects.

```sql
CREATE USER username PASSWORD 'password';
ALTER USER username [options...];
DROP USER username;
```

You can view user information with:

```sql
SELECT * FROM pg_user ORDER BY usename;
```

or for system-level metadata:

```sql
SELECT * FROM svl_user_info;
```

Redshift supports groups (roles) — collections of users that share the same privileges. Any privilege granted to a group applies automatically to all its members. You can manage roles via:

```sql
CREATE GROUP groupname;
ALTER GROUP groupname ADD/DROP USER username;
DROP GROUP groupname;
```

Use groups to simplify privilege management, especially in environments with many users. 

## Schemas & Search Path

A database contains schemas, each of which holds objects like tables and views. By default, every database includes a schema named `public`. Schemas help organize your data logically and isolate workloads across teams or applications.

Common operations:

```sql
-- Create a schema
CREATE SCHEMA schemaname AUTHORIZATION username1;

-- Change owner
ALTER SCHEMA schemaname OWNER TO username2;

-- Drop schema and all its objects
DROP SCHEMA schemaname CASCADE;
```

Schemas are similar to directories — they help organize and isolate objects. Redshift limits databases to a maximum number of schemas (commonly ~9,900).

The **search path** defines which schema Redshift searches first when referencing unqualified object names. Objects created without a schema are placed in the first schema in the search path. The system catalog (`pg_catalog`) is always searched, even if not explicitly listed.

```sql
SET search_path TO schemaname, public;
```

## Inspecting Permissions
You can view, test, or audit privileges using built-in system views and functions:

| Purpose                              | View / Function                                        |
| ------------------------------------ | ------------------------------------------------------ |
| List grants for an object            | `SHOW GRANTS`                                          |
| View schema-level grants             | `SVV_SCHEMA_PRIVILEGES`                                |
| Check if a user has schema privilege | `HAS_SCHEMA_PRIVILEGE(user, schema, privilege)`        |
| Check if a user has table privilege  | `HAS_TABLE_PRIVILEGE(user, 'schema.table', privilege)` |
| Inspect users and groups             | `pg_user`, `pg_group`                                  |
| Inspect schema definitions           | `pg_namespace`                                         |

## Best Practices
- Use roles or groups for privilege assignment — avoid granting directly to users.
- Limit superuser accounts to administrative tasks only.
- Revoke default `CREATE `on `public` schema in shared databases.
- Regularly audit privileges using `SHOW GRANTS` and system views.
- Document ownership of critical tables and schemas to prevent orphaned objects.

## Summary

Managing users and privileges in Redshift revolves around three key principles:

1. Define clear ownership and least-privilege access.
2. Centralize permissions through groups or roles.
3. Use superuser rights responsibly and sparingly.

With these practices, you can maintain a secure, organized Redshift environment that scales cleanly across users and teams.
