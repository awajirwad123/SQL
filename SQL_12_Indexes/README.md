# SQL Indexes Module (SQL_12_Indexes)

## Contents
1. `01_Indexes.md` – Concepts, structures, selectivity, covering, partial & functional.
2. `02_Create_Index.md` – Design strategy, composite ordering, functional/partial, online builds.
3. `03_Drop_Index.md` – Identifying unused/redundant indexes, safe removal.
4. `04_Show_Indexes.md` – Catalog queries & usage stats.
5. `05_Unique_Index.md` – Uniqueness semantics, functional & partial uniqueness.
6. `06_Clustered_vs_NonClustered_Index.md` – Physical storage vs secondary access paths.

## Goals
Provide an interview-ready and practical framework to reason about when, why, and how to apply indexing for performance without over-indexing.

## Key Decision Flow
1. Identify high-frequency WHERE / JOIN predicates.
2. Determine column selectivity & cardinality.
3. Design composite indexes: equality columns → range columns → ordering columns.
4. Consider covering (INCLUDE) vs table lookup trade-off.
5. Add functional/partial indexes only when predicate stable & selective.
6. Periodically audit: usage stats + storage footprint.

## Quick Heuristics
| Pattern | Index Approach |
|---------|----------------|
| `WHERE user_id=? AND created_at BETWEEN` | `(user_id, created_at)` |
| Case-insensitive email lookup | Functional index on `LOWER(email)` |
| Frequently filtered active flag + user | Partial index `WHERE active` |
| Composite duplicates | Drop subset if unused |
| Slow ORDER BY after filter | Add ordering column at index tail |

## Anti-Patterns
- Index every column touched in SELECT list.
- Leading column of composite rarely filtered.
- Multiple overlapping indexes not pruned.
- Using wide text as leading key.
- Ignoring maintenance (stats, bloat, fragmentation).

## Practice Prompts
1. Optimize query: `SELECT * FROM orders WHERE customer_id=? AND created_at>=? ORDER BY created_at DESC LIMIT 20;`
2. Enforce case-insensitive uniqueness for multi-tenant usernames.
3. Drop redundant indexes safely in a staging environment.
4. Diagnose why optimizer chose a full scan despite an index.
5. Design indexes for a reporting query grouping by status and date bucket.

## Next Suggested Modules
- Query Execution Plans & Cost Estimation
- Advanced Window Function Optimization
- Locking & Concurrency (Transaction Isolation)

Use these files alongside Joins/Functions/Views modules for end-to-end performance literacy.
