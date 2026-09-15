---
title: CTEs vs Temporary Tables
date: 2019-10-02 00:00:00 +0100
tags: [SQL]
---

## CTEs vs. Temporary Tables – Which One Should You Use?
Both Common Table Expressions (CTEs) and temporary tables allow you to work with intermediate data inside a query, but they behave differently in scope, performance, and use cases. Understanding when to use each can make your SQL simpler — and sometimes faster.

## Common Table Expressions (CTEs)
A **CTE (Common Table Expression)** is a named, temporary result set that exists only for the duration of a single SQL statement. You define it using the `WITH` clause, which can contain one or more comma-separated subqueries. Each subquery defines a logical result set that can be referenced later in the query.

Example syntax:

```sql
WITH cte_name (column1, column2, ...) AS (
  SELECT ...
)
SELECT ...
FROM cte_name;
```

CTEs make complex queries easier to read and maintain by replacing repeated subqueries with a single definition. They can also reference themselves (recursive CTEs) and can be reused multiple times within the same statement.

In Amazon Redshift and most modern databases, `WITH` clause subqueries can be optimized internally so that repeated references to the same CTE don’t always trigger multiple evaluations. This depends on the optimizer and query complexity.

A few important notes:
* The result set exists only for the duration of one query.
* A CTE is not a stored object — it disappears immediately after execution.
* It’s ideal when you need to simplify a query or organize logic without creating a physical table or view.

*[Read more: Amazon Redshift documentation on the WITH clause](https://docs.aws.amazon.com/redshift/latest/dg/r_WITH_clause.html)*

## Temporary Tables
A **temporary table** physically exists for the duration of a session and is automatically dropped when the session ends. Temporary tables are stored in a session-specific schema (which users cannot specify) and can safely share a name with a permanent table without causing conflicts.

Example syntax:

```sql
CREATE TEMPORARY TABLE my_temp AS
SELECT ...
FROM source_table;
```

Key characteristics:

- Temporary tables persist for the entire session and can be used across multiple queries.
- They can have indexes and constraints, which CTEs cannot.
- Because they are materialized, they can be joined, updated, or queried repeatedly.
- They are often used for staging, data transformations, or intermediate aggregations.

When a temporary table shares a name with a permanent one, the temporary table takes precedence in the current session’s search path (unless you explicitly qualify the permanent table’s schema).

*[Read more: Amazon Redshift documentation on CREATE TABLE](https://docs.aws.amazon.com/redshift/latest/dg/r_CREATE_TABLE_NEW.html)*

## Choosing Between a CTE and a Temporary Table

| Use Case                                                | Recommended Option  |
| ------------------------------------------------------- | ------------------- |
| One-off query simplification                            | **CTE**             |
| Reusing intermediate data across multiple statements    | **Temporary table** |
| Need for indexes or constraints                         | **Temporary table** |
| Quick, inline transformation or readability improvement | **CTE**             |
| Large or complex transformations requiring reuse        | **Temporary table** |

In short:

- Use a CTE for readability and short-lived transformations within a single query.
- Use a temporary table when you need to persist intermediate results, reuse data, or improve performance through indexing and repeated access.

For deeper discussion and community insights, check out this [Stack Overflow thread](https://dba.stackexchange.com/questions/13112/whats-the-difference-between-a-cte-and-a-temp-table/13117#13117).
