# SQL Joins Module Index

## 1. Purpose
This index gives you a **fast revision map** of all logical join types, modification joins, hierarchical techniques, and physical/distributed join algorithms covered in this module (`SQL_9_Joins`). Use it to:
- Pick the right join for a problem (semantic ➜ physical ➜ optimization)
- Recall interview phrasing & traps quickly
- Navigate to the detailed markdown files (each topic has its own file)

---
## 2. Logical Join Types (Result-Set Semantics)
| File | Join Type | What It Returns | Typical Use | Key Trap |
|------|-----------|-----------------|------------|----------|
| `01_INNER_JOIN.md` | INNER | Only matching rows | Standard relational linking | Unintentional row multiplication |
| `02_Outer_Join.md` | OUTER (concept) | Rows + NULL padding | Conceptual grouping of LEFT/RIGHT/FULL | Filters in WHERE killing outer effect |
| `03_Left_Join.md` | LEFT | All left + matched right | Driving report list | Filters on right in WHERE = accidental inner |
| `04_Right_Join.md` | RIGHT | All right + matched left | Legacy / perspective choice | Prefer rewrite as LEFT |
| `05_Full_Join.md` | FULL | All rows both sides | Reconciliation / diff | Not in MySQL (must emulate) |
| `06_Cross_Join.md` | CROSS | Cartesian product | Combination grids / scaffolds | Explosive cardinality |
| `07_Self_Join.md` | SELF | Table to itself | Hierarchies, comparisons | Forgetting LEFT for roots |
| `08_Natural_Join.md` | NATURAL | Auto join on same column names | Quick ad‑hoc | Schema drift changing semantics |

---
## 3. DML Joins (Modification with Joins)
| File | Pattern | Purpose | Safety Tip |
|------|---------|---------|-----------|
| `09_Update_with_Join.md` | UPDATE + JOIN / FROM | Sync / cache / derived columns | Preview with SELECT first |
| `10_Delete_Join.md` | DELETE + JOIN / USING | Cleanup / cascade substitute | Anti-join orphans via NOT EXISTS |

---
## 4. Hierarchical / Recursive
| File | Technique | Use Case | Advantage |
|------|-----------|----------|----------|
| `11_Recursive_Join.md` | Recursive CTE (repeated self-join) | Trees, org charts, category expansion | Depth tracking + path creation |
| `07_Self_Join.md` | Single-level self join | Manager lookup, peer comparisons | Simpler & faster for 1-level |

---
## 5. Physical Join Algorithms (Query Engine Execution)
These implement logical joins internally.
| File | Algorithm | Best When | Weakness |
|------|-----------|-----------|----------|
| `12_Nested_Loop_Join.md` | Nested Loop | Small outer, indexed inner | Quadratic w/ large scans |
| `13_Index_Nested_Loop_Join.md` | Index Nested Loop | Highly selective probes | Outer growth degrades quickly |
| `14_Block_Nested_Loop_Join.md` | Block Nested Loop | Non-equality predicates, moderate size | Still O(N×M) worst case |
| `15_Hash_Join.md` | Hash Join | Large unsorted equi-joins | Memory / skew / spill |
| `16_Grace_Hash_Join.md` | Grace Hash Join | Very large (doesn't fit RAM) | Extra disk I/O partitions |
| `17_Merge_Join.md` | Merge (Sort-Merge) | Both sides sorted / order needed | Costly sorts if not pre-ordered |

---
## 6. Distributed / Parallel Join Strategies
| File | Strategy | Platform Context | When Chosen | Trade-off |
|------|----------|------------------|-------------|-----------|
| `18_Broadcast_Join.md` | Broadcast | Spark / Presto / Snowflake | Small dimension + large fact | Memory replication |
| `19_Shuffle_Join.md` | Shuffle (Repartition) | Distributed Engines | Large-large joins | Network cost + skew |

---
## 7. Join Selection Decision Flow
1. **Need all combinations?** ➜ `CROSS JOIN` (verify size) or generate dimension programmatically.
2. **Have relationship predicate?** ➜ Choose logical type (INNER vs OUTER vs FULL).
3. **Need hierarchy traversal?** ➜ Self join (1 level) or recursive CTE (multi-level).
4. **Doing UPDATE/DELETE?** ➜ Use join variant or correlated subquery; preview.
5. **Execution scale concerns?** ➜ Map logical join to physical algorithm:
   - Small outer + index: Index Nested Loop
   - Large unsorted equi: Hash Join
   - Pre-sorted sources / need order: Merge Join
   - Very large (memory overflow): Grace Hash Join
   - Non-equality (e.g., BETWEEN): Block Nested Loop
6. **Distributed environment?**
   - Small side tiny: Broadcast Join
   - Both large: Shuffle Join (watch skew)

---
## 8. Anti‑Join & Semi‑Join Patterns (Common Interview Edge)
| Goal | Example Pattern | Notes |
|------|-----------------|-------|
| Rows in A with no match in B | `LEFT JOIN ... WHERE b.key IS NULL` | Or `NOT EXISTS` (preferred clarity) |
| Rows in A that have at least one match in B | `EXISTS (SELECT 1 ...)` | Avoid duplicate expansion |
| First match only | Window function + filter `ROW_NUMBER() = 1` | Better than DISTINCT after join |

---
## 9. Null & Filter Placement Cheat Sheet
| Scenario | Correct Placement | Wrong Placement Effect |
|----------|-------------------|------------------------|
| Preserve unmatched right rows while filtering matches | Put predicate in `ON` | Putting in `WHERE` drops unmatched rows (becomes inner) |
| Count all left rows | `COUNT(*)` | `COUNT(right.col)` skips NULL padded rows |
| Anti-join intention | `LEFT JOIN` + `WHERE right.pk IS NULL` | Filtering in ON loses unmatched detection |

---
## 10. Performance Quick Wins
- Index foreign keys and join key columns.
- Push selective predicates BEFORE join (reduce outer/probe side size).
- Avoid `SELECT *` on wide tables—column pruning reduces memory & network.
- Watch data skew (Hot keys → Hash / Shuffle imbalance.)
- Prefer `EXISTS` over `IN` with correlated subqueries in some optimizers.
- Use EXPLAIN to confirm chosen physical algorithm; mismatch may imply missing index or bad stats.

---
## 11. Interview Flash Answers
| Question | 3-Second Answer |
|----------|-----------------|
| Difference INNER vs LEFT | INNER: only matches; LEFT: all left + NULLs |
| FULL OUTER JOIN use case | Reconciliation / diff reporting |
| Prevent accidental Cartesian | Always specify ON; use explicit CROSS for intent |
| Why NATURAL JOIN risky | Schema changes silently alter predicate |
| Hash vs Merge | Hash: memory + unsorted; Merge: needs order, preserves order |
| Broadcast vs Shuffle | Broadcast replicates small; Shuffle repartitions both |
| Recursive CTE purpose | Hierarchical multi-level expansion |
| Anti-join pattern | LEFT JOIN ... WHERE right IS NULL OR NOT EXISTS |

---
## 12. Common Traps (Red Flags)
- Filtering right table columns in WHERE after a LEFT JOIN.
- Assuming `UNION` simulation of FULL JOIN preserves duplicates (it removes them; use `UNION ALL` + filters if needed).
- Using CROSS JOIN accidentally due to missing predicate (row explosion).
- Large join chosen with Nested Loop due to missing stats—ANALYZE / UPDATE STATISTICS.
- Forcing broadcast join on mid-sized table → executor OOM.

---
## 13. When to Refactor Join Logic
| Symptom | Refactor To |
|---------|-------------|
| Many self joins for hierarchy depth | Recursive CTE |
| Repeated point lookups inside loop (app layer) | Single SQL join |
| Large CROSS JOIN + later filter | Turn filter into ON predicate / semi-join |
| Slow UPDATE ... JOIN | Add index to target filter or rewrite using EXISTS |
| Hash join spilling | Increase memory / rewrite to pre-aggregate smaller side |

---
## 14. File Map & Navigation
```
SQL_9_Joins/
  00_Joins_Index.md (this file)
  01_INNER_JOIN.md
  02_Outer_Join.md
  03_Left_Join.md
  04_Right_Join.md
  05_Full_Join.md
  06_Cross_Join.md
  07_Self_Join.md
  08_Natural_Join.md
  09_Update_with_Join.md
  10_Delete_Join.md
  11_Recursive_Join.md
  12_Nested_Loop_Join.md
  13_Index_Nested_Loop_Join.md
  14_Block_Nested_Loop_Join.md
  15_Hash_Join.md
  16_Grace_Hash_Join.md
  17_Merge_Join.md
  18_Broadcast_Join.md
  19_Shuffle_Join.md
```

---
## 15. Practice Prompts
1. Write a query to list all customers and number of orders including those with zero orders (LEFT JOIN + aggregation).  
2. Identify products referenced in orders that no longer exist in the products table (RIGHT/LEFT anti-join).  
3. Compare two dimension snapshots (FULL OUTER JOIN classify match types).  
4. Generate scaffold of 7 days × all products (CROSS JOIN + date filter).  
5. Retrieve full employee chain from CEO to leaf (Recursive CTE).  
6. Explain why optimizer chose Hash Join over Merge for a large equi-join (unsorted inputs).  
7. Convert a DELETE JOIN to a NOT EXISTS pattern and explain benefits.  

---
## 16. Rapid Selection Matrix
| Need | Pick |
|------|------|
| Mandatory relationship only | INNER JOIN |
| Keep all left even if missing right | LEFT JOIN |
| Reconciliation between datasets | FULL OUTER JOIN |
| All combinations required | CROSS JOIN |
| Single-level parent lookups | SELF JOIN |
| Multi-level tree traversal | Recursive CTE |
| Very selective small probe | Index Nested Loop |
| Huge unsorted equi-join | Hash Join |
| Pre-sorted large tables | Merge Join |
| Large join not fitting memory | Grace Hash Join |
| Distributed (small + large) | Broadcast Join |
| Distributed (large + large) | Shuffle Join |

---
## 17. Suggested Next Modules
After Joins:  
- Window Functions  
- Indexing & Query Plans  
- Transactions & Isolation  
- CTEs & Subquery Strategies  
- Performance / Cost-Based Optimization

---
## 18. Final Revision Strategy
1. Skim this index (2–3 min).  
2. Drill only weak zones (open corresponding file).  
3. Practice 3 mixed queries (cover left, full, recursive).  
4. Rehearse 5 flash answers aloud.  
5. Do a final EXPLAIN plan interpretation exercise.

---
**Use this file as your mental compass before deep dives.**
