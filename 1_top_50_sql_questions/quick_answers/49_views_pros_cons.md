# 49. Views: Pros & Cons

Q: What is a view?
A: A named, stored SELECT (logical layer) that presents derived or restricted data.

Pros:
- Abstraction & simplification.
- Security (column/row exposure control).
- Reuse of complex joins/calculations.
- Insulation from base schema changes (stable interface).

Cons:
- Performance: complex nested views can bloat query plans.
- Not automatically indexed (unless materialized view variant or indexed view).
- Can hide costly operations, misleading developers.

Updatability:
- Simple views (single table, direct columns) often updatable.
- Complex (aggregates, joins, DISTINCT) usually read-only.

Pitfall: Layering views-on-views causing opaque performance issues—flatten frequently-used logic.
