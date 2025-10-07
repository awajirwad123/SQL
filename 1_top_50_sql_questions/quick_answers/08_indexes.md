# 8. Indexes

Q: What is an index?
A: Auxiliary structure (often B-Tree) mapping key values to row locations for faster reads.

Q: Benefits?
A: Speeds selective WHERE, JOIN, ORDER BY, GROUP BY—enables index-only scans.

Q: Cost?
A: Extra storage + slower writes (maintain each index per DML).

Q: Composite order rule?
A: Leftmost prefix—(a,b,c) supports (a), (a,b), (a,b,c) predicates.

Q: When not to index?
A: Tiny tables, low-selectivity flags, high-churn columns, unused in predicates.

Q: Sargability?
A: Write predicates so engine can seek (avoid wrapping indexed column in functions).

Q: Covering index?
A: Contains all needed columns so base table not touched.

Q: Common mistake?
A: Many overlapping single-column indexes instead of a well-designed composite.
