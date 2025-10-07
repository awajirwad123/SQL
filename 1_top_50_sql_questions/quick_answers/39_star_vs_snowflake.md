# 39. Star vs Snowflake Schemas

Q: Star schema?
A: Central fact table linked directly to denormalized dimension tables (wide, flattened dims).

Q: Snowflake schema?
A: Dimensions normalized into multiple related tables (hierarchies split out).

Q: When choose Star?
A: Faster, simpler BI querying; fewer joins; good for common analytics workloads.

Q: When choose Snowflake?
A: Reduce dimension redundancy (large shared hierarchies), improve maintainability where storage or repeated attributes matter.

Q: Trade-offs?
A: Star: Slight redundancy & possible update complexity. Snowflake: More joins, potentially slower queries.

Q: Hybrid?
A: Start star, selectively snowflake high-cardinality or shared hierarchical dims (e.g., geography).

Q: Pitfall?
A: Over-normalizing dims harming self-service BI performance.
