---
name: postgresql
description: Design, query, index, and operate PostgreSQL in production covering schema modeling, transactions and locking, concurrency, query optimization, zero-downtime migrations, backup/recovery, and operational monitoring.
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# PostgreSQL

## Purpose

Work with PostgreSQL at a production standard: schema design and normalization, indexing, transactions/isolation/locking, concurrency patterns, query analysis with EXPLAIN, zero-downtime migrations, connection management, backup and recovery, and monitoring. Use when the task involves database modeling, slow queries, migrations, or Postgres operations.

## Instructions

### 1. Connection and setup

- `DATABASE_URL` env var (`postgres://user:password@host:5432/dbname`); never commit connection strings. In Django use `django-environ`/`dj-database-url`.
- Local dev: Postgres in Docker (see `docker` skill), **pinned to the same major version as production**.
- Django: `CONN_MAX_AGE = 60` (persistent connections). Add **PgBouncer in transaction mode** when connections approach ~half of `max_connections` (note: transaction mode forbids session state — no `SET`, no advisory session locks, disable server-side cursors in Django with `DISABLE_SERVER_SIDE_CURSORS = True`).
- Set safety timeouts for app connections (per role or in the pooler):

```sql
ALTER ROLE app_user SET statement_timeout = '30s';
ALTER ROLE app_user SET lock_timeout = '5s';
ALTER ROLE app_user SET idle_in_transaction_session_timeout = '60s';
```

- Least privilege: the app role owns DML only; a separate migration role owns DDL; nobody applicational is superuser.

### 2. Schema design and modeling

- **Normalize to 3NF by default**; denormalize only for a measured read-path problem, and document the invariant that keeps the copy consistent (trigger, service code, or periodic reconciliation). Duplicated data without an owner of consistency is a bug factory.
- **Decision rule — when to create a new table**: create one when the concept has its own lifecycle/identity or a 1:N/M:N relation ("a user has many addresses"). Do NOT create a table for attributes that are 1:1 with an existing row (add columns), nor an EAV (`entity_attribute_value`) table to avoid migrations — use `jsonb` for genuinely dynamic, non-queried-relationally attributes instead.
- **Decision rule — jsonb vs columns**: `jsonb` for heterogeneous, schema-less payloads (webhook bodies, third-party responses, user-defined metadata). Real columns for anything you filter, join, constrain, or aggregate on. If you keep writing `data->>'status'` in WHERE clauses, promote it to a column.
- PKs: `bigint GENERATED ALWAYS AS IDENTITY` (Django `BigAutoField`). UUIDs only when IDs must be non-guessable externally or generated across systems (see `django` skill decision rule).
- Types: `timestamptz` always (never `timestamp`), `numeric` for money, `text` over `varchar(n)` unless a real business limit exists, `boolean` not `char(1)`, arrays sparingly.
- **Integrity lives in the database**: `NOT NULL`, `UNIQUE`, `CHECK`, FKs with explicit `ON DELETE` (`RESTRICT`/`PROTECT` for business-critical parents, `CASCADE` only for true composition like order → order_items). Application-only validation WILL be bypassed eventually.
- Every business table: `created_at timestamptz NOT NULL DEFAULT now()` and `updated_at`.
- Naming: snake_case, plural tables (pick one convention and keep it), index names `ix_<table>_<cols>`, constraint names explicit.
- **Decision rule — partitioning**: only for very large tables (100M+ rows / retention-based deletes) where queries carry the partition key. It complicates indexes and FKs — do not partition preemptively.

### 3. Transactions, concurrency, locking

- Keep transactions **short**: never hold one open across user interaction, external API calls, or queue waits (`idle in transaction` blocks vacuum and locks).
- Default isolation `READ COMMITTED` is right for most OLTP. Use `SERIALIZABLE`/`REPEATABLE READ` only for genuinely conflicting business invariants, and handle serialization failures with retry.
- **Preventing lost updates** (two writers, same row):
  - Atomic in-place math: `UPDATE accounts SET balance = balance - 100 WHERE id = $1` (Django `F()` expressions) — preferred.
  - Pessimistic: `SELECT ... FOR UPDATE` inside the transaction (Django `select_for_update()`); add `SKIP LOCKED` for job-queue patterns (`FOR UPDATE SKIP LOCKED` is THE pattern for pulling work from a table).
  - Optimistic: a `version` column checked in the UPDATE's WHERE clause when contention is rare and holding locks is undesirable.
- Enforce cross-row invariants ("max 3 active bookings per user") with a constraint when possible (partial unique index, exclusion constraint for ranges — `EXCLUDE USING gist` for booking overlaps); otherwise lock the parent row first.
- Deadlocks: always acquire locks in a consistent order (e.g. by ascending id); keep `lock_timeout` set so DDL and hot paths fail fast instead of queueing.

### 4. Indexing

- Index: FK columns (Postgres does NOT auto-index them), columns in frequent `WHERE`/`JOIN`/`ORDER BY`, unique business keys.
- Composite order: equality columns first, then the range/sort column. An index on `(a, b)` serves `a` alone but not `b` alone.
- Special cases:
  - `GIN` for `jsonb` containment and full-text search (`tsvector`).
  - Partial indexes for hot subsets: `CREATE INDEX ... WHERE status = 'active';` — also the way to make "unique among active rows" constraints.
  - Covering indexes (`INCLUDE`) for index-only scans on hot read paths.
  - `text_pattern_ops` / trigram (`pg_trgm` + GIN) for `LIKE 'abc%'` / fuzzy search.
- Don't over-index: every index taxes writes and vacuum. Audit unused ones:

```sql
SELECT indexrelname, idx_scan, pg_size_pretty(pg_relation_size(indexrelid))
FROM pg_stat_user_indexes ORDER BY idx_scan ASC;
```

### 5. Query analysis and optimization

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```

- Red flags: `Seq Scan` on large tables with selective filters, `Rows Removed by Filter` huge, nested loops over high row counts, `Sort Method: external merge` (spilling to disk), estimated vs actual rows off by orders of magnitude (stale stats → `ANALYZE`).
- Fix order: right index → rewrite query (avoid `SELECT *`, avoid functions over indexed columns in WHERE, avoid `OFFSET` for deep pagination — use keyset/cursor pagination) → only then consider denormalization/caching.
- Find the worst offenders with `pg_stat_statements`:

```sql
SELECT query, calls, mean_exec_time, rows
FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 10;
```

- In Django: `django-debug-toolbar` in dev, `assertNumQueries` in tests, `select_related`/`prefetch_related` against N+1.

### 6. Zero-downtime migrations

Safe / unsafe operations on live tables:

| Operation | Safe approach |
|---|---|
| Add column | Nullable, or with constant default (PG 11+) — safe |
| Add index | `CREATE INDEX CONCURRENTLY` (Django: `AddIndexConcurrently`, `atomic = False`) |
| Add NOT NULL / FK / CHECK | Add `NOT VALID` → backfill in batches → `VALIDATE CONSTRAINT` |
| Rename/drop column | Never while old code runs: **expand → migrate → contract** across ≥2 deploys |
| Change column type | New column → dual-write → backfill → swap reads → drop old |
| Backfill data | Batched UPDATEs (1k–10k rows) with pauses; never one giant UPDATE |

- Always run DDL with `lock_timeout` set (e.g. `5s`) so a blocked `ALTER TABLE` fails instead of queueing behind a long transaction and freezing the app.
- Migrations must be backward-compatible with the previous app release (deploys overlap). DB rollback = forward-fix migration, not restoring a dump.

### 7. Backup, recovery, retention

```bash
pg_dump -Fc "$DATABASE_URL" > backup_$(date +%F).dump    # custom format, compressed
pg_restore -d "$DATABASE_URL" --clean --if-exists backup.dump
```

- Define **RPO/RTO** explicitly. Daily dumps give RPO = up to 24h; if that is unacceptable, use WAL archiving / managed PITR.
- **A backup that was never restored is not a backup**: test restores on a schedule (restore into a scratch DB, run sanity queries).
- Store backups off-instance, encrypted, with retention policy (e.g. 7 daily + 4 weekly + 12 monthly).
- Before every risky migration or bulk operation: take a snapshot/dump first.

### 8. Monitoring and maintenance

- Watch: connection count vs `max_connections`, replication lag (if replicas), table/index bloat, cache hit ratio, long-running transactions:

```sql
SELECT pid, state, now() - xact_start AS xact_age, query
FROM pg_stat_activity
WHERE state <> 'idle' ORDER BY xact_age DESC;
```

- Autovacuum must keep up; on write-heavy tables tune per-table (`autovacuum_vacuum_scale_factor`) rather than disabling it.
- Log slow queries: `log_min_duration_statement = 500ms` (or provider equivalent) and review regularly.

### 9. Useful psql commands

```
\dt   list tables      \d+ table   describe with sizes
\di   list indexes     \x          expanded output
\l    list databases   \timing     query timing
```

## Checklists

### Before implementing (schema change)
- [ ] New table justified by decision rule §2 (own lifecycle / 1:N), not attribute sprawl
- [ ] Types correct (`timestamptz`, `numeric` for money); constraints in the DB
- [ ] FK `ON DELETE` behavior chosen deliberately
- [ ] Migration is zero-downtime safe (table §6) and backward compatible

### During implementation
- [ ] Indexes for new FK columns and query patterns; `CONCURRENTLY` on live tables
- [ ] Concurrent writers analyzed: `F()`/atomic UPDATE, `FOR UPDATE`, or version column
- [ ] Backfills batched; `lock_timeout` set for DDL

### Before delivering
- [ ] `EXPLAIN (ANALYZE, BUFFERS)` run on new hot queries — no unexpected Seq Scans
- [ ] Migration applied + rolled forward on a staging copy
- [ ] Backup/snapshot taken before running against production data
- [ ] No credentials in code or logs

## Best Practices

- Short transactions; timeouts everywhere; consistent lock ordering.
- Prefer database constraints over application discipline.
- Measure before optimizing: `pg_stat_statements` + EXPLAIN, not guesses.
- Match dev/CI Postgres major version to production.

## Triggers

- "postgres", "postgresql", "banco de dados", "modelagem", "índice", "query lenta", "explain", "deadlock", "lock", "migração de banco", "pg_dump", "backup", "sql"

## Related Skills

- `django`: ORM layer and migration workflow
- `docker`: local Postgres via compose
- `ci-cd`: migration checks and service containers in pipelines
