---
name: mssql-env-db-helper
description: Help design and query SQL Server schemas across prod/beta/dev environments, including inspecting metadata, generating safe SQL, and planning migrations. Use when working with SQL Server databases for this project, especially when environment-specific differences matter.
disable-model-invocation: true
---

# SQL Server Env Database Helper

## Goal

Provide consistent guidance for working with three SQL Server environments (`prod`, `beta`, `dev`) for this project:
- **Designing and evolving the schema**
- **Generating and reviewing SQL (queries, views, stored procedures)**
- **Planning safe changes across prod/beta/dev**

You will fill in the real connection strings; this skill tells the agent how to use them safely.

## Connection configuration (to be updated by you)

Update these placeholders with the actual connection strings once you have them. Prefer using secrets (user secrets, environment variables, or a local config file) instead of hardcoding them in code.

```markdown
Prod:
  ConnectionString: [[PROD_CONNECTION_STRING_GOES_HERE]]

Beta:
  ConnectionString: [[BETA_CONNECTION_STRING_GOES_HERE]]

Dev:
  ConnectionString: [[DEV_CONNECTION_STRING_GOES_HERE]]
```

When discussing connections or writing helper code:
- **Never print full connection strings** in logs, chat, or UI.
- It is fine to refer to them symbolically as `Prod`, `Beta`, and `Dev` connection strings.

## Environment usage rules

When the agent is helping with anything database-related:

1. **Default to Dev**
   - Assume all exploratory queries, schema inspection, and experiments run against **Dev**, unless the user explicitly says `beta` or `prod`.

2. **Beta usage**
   - Use **Beta** when:
     - Validating schema changes before a production rollout.
     - Testing performance and query plans with realistic data.
   - Avoid destructive operations on Beta unless the user explicitly allows them.

3. **Prod safety**
   - Treat **Prod as read-mostly** unless the user explicitly requests data changes.
   - When generating SQL for Prod:
     - Prefer **SELECT** and read-only diagnostics.
     - For DDL/DML, always:
       - Provide a **script in a transaction**.
       - Include a **`ROLLBACK TRAN` option** and clearly mark where `COMMIT` would occur.
       - Add a **backup/verification step** (e.g., row counts before/after, schema snapshot).

## Schema design and inspection workflow

When the user asks for help with schema design, relationships, or refactoring:

1. **Clarify the target environment**
   - Assume **Dev** unless the user specifies otherwise.

2. **Inspect current schema**
   - Use SQL Server metadata views (examples the agent can generate on request):
     - `INFORMATION_SCHEMA.TABLES`
     - `INFORMATION_SCHEMA.COLUMNS`
     - `sys.tables`, `sys.columns`, `sys.indexes`, `sys.foreign_keys`, etc.

3. **Propose or update design**
   - Suggest:
     - Table structures (columns, data types, nullability, defaults).
     - Primary and foreign keys.
     - Indexes (including filtered or composite indexes when needed).
   - Keep naming consistent with existing project conventions (e.g., `PascalCase` table names or whatever is already in use).

4. **Generate migration scripts**
   - For schema changes, always generate:
     - A **forward migration** (Dev → Beta → Prod).
     - If possible, a **rollback plan** (e.g., how to revert or backfill).
   - Scripts should be environment-agnostic; the **connection string** chooses the environment, not hardcoded database names unless the repo uses them consistently.

## Query and stored procedure guidance

When the user asks to create or optimize queries, views, or stored procedures:

- **Parameterization**
  - Always use parameterized SQL (avoid string concatenation for values).

- **Performance**
  - Encourage use of:
    - Appropriate indexes.
    - `EXISTS` vs `IN` where suitable.
    - Avoiding `SELECT *` in production-facing code.

- **Environment selection**
  - Assume Dev for:
    - New query development.
    - Trying out query rewrites and index suggestions.
  - Recommend Beta (not Prod) for:
    - Verifying performance with near-real data.

## Working across prod/beta/dev

When comparing environments or planning rollouts:

1. **Detect differences**
   - The agent can propose scripts to:
     - Compare schemas (e.g., diff tables/columns/indexes between Dev and Prod).
     - Check for missing indexes or tables.

2. **Rollout order**
   - Always assume rollout in this order:
     1. Dev
     2. Beta
     3. Prod

3. **Checks before Prod**
   - Before suggesting a Prod change, the agent should:
     - Confirm the script has been applied or at least tested on Dev.
     - Recommend testing on Beta.

## How the agent should use this skill

When this skill is active and the user mentions:
- `SQL Server`, `MSSQL`, `prod/beta/dev databases`
- schema design, table changes, migrations, or complex queries

The agent should:
1. **Assume SQL Server** as the database engine.
2. **Follow the environment rules** above (Dev by default, Prod read-mostly).
3. **Ask for or respect environment choice** if the user specifies one.
4. **Generate SQL and plans** that can be applied cleanly across Dev → Beta → Prod without hardcoding secrets.

