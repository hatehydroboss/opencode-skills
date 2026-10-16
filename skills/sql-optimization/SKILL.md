---
name: sql-optimization
description: Optimize slow SQL queries, design indexes, analyze execution plans, and tune database performance. Use when queries are slow, when writing complex JOINs, when designing indexes, when seeing table scans, or when working with EXPLAIN/EXPLAIN ANALYZE output. Trigger on "slow query", "optimize SQL", "index", "execution plan", "EXPLAIN", "N+1", or "query performance".
---

# SQL Optimization

Find and fix slow queries systematically. Always measure before and after.

## Diagnostic Workflow

1. Get the actual query (check logs or application code)
2. Run `EXPLAIN ANALYZE <query>` (PostgreSQL) or `EXPLAIN <query>` (MySQL)
3. Look for: sequential scans on large tables, high row estimates, nested loops over large sets, sorts on disk
4. Identify the bottleneck: missing index, bad join order, unnecessary columns, N+1 pattern

## Indexing Rules

- Index columns used in `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`
- Composite index column order: equality first, then range, then sort
  - `WHERE status = 'active' AND created_at > '2024-01-01' ORDER BY created_at`
  - → `(status, created_at)`
- One index per query path; don't over-index (slows writes)
- Partial indexes for filtered queries: `CREATE INDEX ... WHERE status = 'active'`
- Covering indexes to avoid table lookups: `INCLUDE (column)`

## Common Anti-Patterns

| Anti-Pattern | Fix |
|-------------|-----|
| `SELECT *` | Only select needed columns |
| `WHERE UPPER(col) = 'X'` | Functional index: `CREATE INDEX ON t (UPPER(col))` |
| `WHERE col LIKE '%foo%'` | Full-text search or trigram index |
| `WHERE col % 100 = 0` | Expression index |
| N+1 queries | Single query with JOIN or `WHERE id IN (...)` |
| `ORDER BY` without index on large table | Add matching index or pre-aggregate |
| Correlated subquery in SELECT | LEFT JOIN or lateral join |
| `IN (huge list)` | Use a temp table or CTE |

## JOIN Optimization

- Join on indexed columns
- Prefer inner joins when you don't need nulls
- Reduce rows before joining (filter in subqueries/CTEs first)
- Watch for fan-out: joining one-to-many then aggregating can multiply rows

## Reading EXPLAIN Output

Key columns to check:
- **Seq Scan** on a large table → needs an index
- **rows** estimate wildly off → run `ANALYZE <table>`
- **Sort Method: external merge** → increase work_mem or add matching index
- **Nested Loop** over thousands of rows → consider hash join or merge join

## N+1 Detection

Look for loops that execute queries:
```python
# BAD
for user in users:
    db.query("SELECT * FROM posts WHERE user_id = ?", user.id)

# GOOD
user_ids = [u.id for u in users]
posts = db.query("SELECT * FROM posts WHERE user_id IN (?)", user_ids)
```

## Checklist

- [ ] Ran EXPLAIN ANALYZE before making changes
- [ ] Indexes match actual query patterns (check pg_stat_user_indexes)
- [ ] Removed unused indexes (check index usage stats)
- [ ] No SELECT * in hot paths
- [ ] Connection pooling enabled
- [ ] Statistics refreshed after bulk data changes (ANALYZE)
