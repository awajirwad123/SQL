# 3. SELECT DISTINCT

Q: What does DISTINCT do?
A: Eliminates duplicate rows based on the full select-list projection.

Q: Syntax example?
A: SELECT DISTINCT country FROM customers;

Q: Multi-column effect?
A: DISTINCT applies to the combination: DISTINCT (country, status) dedupes pairs.

Q: Performance tip?
A: If aggregating anyway, GROUP BY can serve the same dedupe purpose.

Q: Pitfall?
A: Using DISTINCT to mask unintended join multiplication instead of fixing the join.
