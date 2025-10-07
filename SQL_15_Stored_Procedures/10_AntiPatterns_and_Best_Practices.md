# Anti-Patterns & Best Practices

## Overview
Recognizing common stored procedure anti-patterns prevents performance, maintainability, and security issues. This guide contrasts pitfalls with recommended practices.

## Anti-Patterns
| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Business logic overload | Harder to test/version | Keep core data integrity in DB, move domain logic to app/service layer |
| RBAR loops | Slow row-by-row processing | Set-based operations, window functions |
| SELECT * | Extra I/O, brittle schema coupling | Explicit column list |
| Dynamic SQL concatenation | Injection risk, poor plan reuse | Parameterized dynamic SQL / whitelisting |
| Overusing temp tables | I/O overhead, contention | Inline views / CTE unless reuse justified |
| Ignoring error handling | Silent failures, corruption risk | TRY/CATCH or EXCEPTION blocks with rollback |
| Excessive OUT params | Complex interface | Return single result set or structured JSON |
| Embedding credentials | Security risk | External secret management |
| No version control | Drift, rollback pain | Store scripts in VCS, migrations |
| Large monolith procedure | Hard to optimize & reason | Refactor into cohesive, composable routines |

## Best Practices
1. Parameterization & exact typing to preserve SARGability.
2. Set-based design first; loops only when provably necessary.
3. Clear, minimal interface (avoid INOUT complexities).
4. Consistent error handling & rethrow with context.
5. Logging/auditing for critical state changes.
6. Secure execution context with least privilege.
7. Performance baselines before and after changes.
8. Idempotent, versioned deployment scripts.
9. Avoid premature micro-optimizations; measure.
10. Document side effects & transactional behavior.

## Interview-Focused Notes
1. Biggest risk with dynamic SQL? Injection + plan cache fragmentation.
2. Why avoid SELECT *? Schema drift + wasted bandwidth.
3. When accept loop? Small dataset, logic impossible set-based (rare) and proven hotspot absent.
4. Benefit of table-valued parameter vs CSV string? Typed, safe, reusable, better plans.
5. Why version stored procedures? Traceability + safe rollback.

## Quick Recall ✅
- Set-based > loops.
- Parameterize dynamic SQL.
- Minimal, explicit interface.
- Version + test + baseline.
- Log sensitive mutations.

## Interview Traps & Confusions ⚠️
- Assuming procedures inherently secure.
- Using them to bypass proper domain validation.
- Over-indexing due to procedure-specific patterns.
- Neglecting to update stats after large procedural batch loads.
- Monolithic refactor avoidance leading to technical debt.

## Bonus
### Refactor Flag
Mark candidates >500 lines or high cyclomatic complexity for decomposition.

### Baseline Harness
Procedure wrapper capturing start/end timestamp & row counts for performance diff automation.

### Complexity Metric
Track lines, temporary object count, loop count per procedure.

### Observability Integration
Emit structured log row per invocation with correlation id.
