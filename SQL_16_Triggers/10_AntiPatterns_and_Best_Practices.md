# Anti-Patterns & Best Practices (Triggers)

## Overview
Triggers can easily become a hidden source of complexity. Recognizing anti-patterns ensures maintainable, predictable systems.

## Anti-Patterns
| Anti-Pattern | Issue | Better Approach |
|--------------|-------|-----------------|
| Business workflow in trigger | Tight coupling & hidden side-effects | Move to service/procedure layer |
| Heavy computations per row | Latency amplification | Precompute batch / asynchronous processing |
| Global counter increment | Contention hotspot | Sharded/partitioned counters or async aggregation |
| Overlapping triggers same event | Undefined ordering & duplication | Consolidate into single guarded trigger |
| Unbounded recursion | Infinite or deep loops | Nest level guard + idempotent DML |
| Auditing every column change | Bloat & write overhead | Selective / hashed delta | 
| Dynamic SQL string concat | Injection, plan instability | Parameterized / whitelist |
| Silent exception swallow | Corrupted state hidden | Raise/log and rollback |
| Redundant updates (no value change) | Wasted trigger firing | Conditional UPDATE with change detection |
| Using trigger for default values | Complexity vs built-in DEFAULT | Use DEFAULT / generated column |

## Best Practices
1. Single Responsibility: One logical concern per trigger.
2. Keep It Small: <25 lines body or offload to function.
3. Guard Recursion: Check nesting depth or context variable.
4. Selective Firing: Column-specific UPDATE triggers; WHEN clause filters.
5. Idempotency: Only act when state changes.
6. Measure Impact: Periodically benchmark high-frequency triggers.
7. Prefer Declarative: Constraints / generated columns before procedural logic.
8. Secure Ownership: Least privilege for trigger functions.
9. Document Rationale: Purpose, contact, review date.
10. Version Control: Scripts tracked; checksum monitoring.

## Interview-Focused Notes
1. Biggest maintainability risk? Hidden side-effects & duplicated business rules.
2. Why consolidate triggers? Predictability & reduced duplication.
3. Guard condition benefits? Avoid unnecessary work & recursion.
4. Prefer constraints why? Engine optimized & self-documenting.
5. Benchmark necessity? Validate overhead vs expected load.

## Quick Recall ✅
- Constraints first.
- Single concern per trigger.
- Guard recursion & redundant fires.
- Keep logic lean & set-based.
- Document & version.

## Interview Traps & Confusions ⚠️
- Believing triggers free performance cost.
- Using them as universal solution for integration.
- Ignoring concurrency effects on summary tables.
- Over-logging sensitive data.
- Relying on implicit ordering.

## Bonus
### Trigger Review Cadence
Quarterly audit of triggers for necessity & performance metrics.

### Complexity Metrics
Track lines, branching, dependent table count.

### Pre-Deployment Diff
Automated diff of trigger definitions vs repo baseline before migration.

### Observability Hook
Emit structured event with trigger name & duration to monitoring pipeline.
