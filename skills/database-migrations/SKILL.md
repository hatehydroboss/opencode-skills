---
name: database-migrations
description: Design and execute safe database schema migrations with zero downtime. Use when altering tables, adding columns, creating indexes on production data, backfilling data, or planning deployment order. Trigger on "migration", "schema change", "alter table", "add column", "database migration", "zero downtime", or "backfill".
---

# Database Migrations

Schema changes are the riskiest deployments. Plan for zero downtime.

## The Golden Rule

**Expand and contract** — never break existing code with a schema change.

1. **Expand**: Add new schema (column, table) — old code still works
2. **Migrate**: Deploy code that uses new schema
3. **Contract**: Remove old schema after all code is updated

## Safe Migration Patterns

### Adding a Column
```sql
-- Safe: nullable or with default
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- DANGEROUS on large tables (locks table)
ALTER TABLE users ADD COLUMN phone VARCHAR(20) NOT NULL DEFAULT '';
-- Instead: add nullable → backfill → add constraint separately
```

### Adding an Index
```sql
-- PostgreSQL: non-blocking
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- MySQL: check if ALGORITHM=INPLACE is supported for your version
CREATE INDEX idx_users_email ON users(email);
```

### Removing a Column
```sql
-- Step 1: Stop reading the column in application code (deploy)
-- Step 2: Stop writing the column (deploy)
-- Step 3: Drop the column (run this migration)
ALTER TABLE users DROP COLUMN legacy_field;
```

### Renaming
Never `ALTER TABLE ... RENAME`. Instead:
1. Add new column
2. Backfill data from old to new
3. Deploy code using new column
4. Drop old column

## Backfill Strategy

For large tables (millions of rows):

```sql
-- Process in batches to avoid long locks
UPDATE users SET normalized_email = LOWER(email)
WHERE id BETWEEN ? AND ? AND normalized_email IS NULL
LIMIT 10000;
```

- Batch size: 1,000-10,000 rows
- Run during low traffic
- Monitor replication lag
- Make idempotent (safe to re-run)

## Migration Ordering

Deploy migrations BEFORE the code that depends on them:

```
Deploy 1: Add column (migration)     ← safe, old code ignores it
Deploy 2: Use column (code change)   ← column exists
Deploy 3: Drop old column (cleanup)  ← no code uses it anymore
```

## Testing Migrations

- Run migrations against a production-sized dataset copy
- Measure lock duration on the largest table
- Test rollback path
- Verify replication doesn't lag beyond threshold

## Rollback Strategy

Every migration needs a rollback plan:
- **Forward-only** for data migrations (write a compensating migration)
- **Reversible** for schema-only changes when possible

```sql
-- Up
ALTER TABLE orders ADD COLUMN status_v2 VARCHAR(20);

-- Down
ALTER TABLE orders DROP COLUMN status_v2;
```

## Checklist Before Running

- [ ] Migration is idempotent (safe to re-run)
- [ ] Lock duration tested on production-sized data
- [ ] Uses CONCURRENTLY for index creation (PostgreSQL)
- [ ] Backfill runs in batches, not one statement
- [ ] Rollback plan documented
- [ ] Migration order accounts for multi-step deploys
- [ ] Monitoring alerts ready for replication lag
