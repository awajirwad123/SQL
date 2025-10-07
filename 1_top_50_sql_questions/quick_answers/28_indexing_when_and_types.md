# 28. Indexing: When & Clustered vs Non-Clustered

Q: When to add an index?
A: Frequent selective predicates, join keys, foreign keys, ORDER/GROUP BY columns, covering critical queries.

Q: Clustered index?
A: Physical row order defined by index key (only one). In InnoDB it's the PK; in SQL Server you choose.

Q: Non-clustered index?
A: Separate structure storing key + pointer (row id or clustered key) enabling multiple alternative access paths.

Q: Write impact?
A: Each INSERT/UPDATE/DELETE must maintain all indexes—over-indexing slows writes.

Q: Covering index?
A: Index containing all referenced columns—avoids table lookup.

Q: Pitfall?
A: Adding indexes to low-selectivity columns (e.g., status flag) yields little benefit and adds write cost.
