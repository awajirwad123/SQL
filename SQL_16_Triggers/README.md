# SQL Triggers Module (SQL_16_Triggers)

## Contents
1. `01_Triggers.md` – Concept & fundamentals.
2. `02_Create_Trigger.md` – Syntax & creation patterns.
3. `03_Timing_and_Event_Types.md` – BEFORE / AFTER / INSTEAD OF; event selection.
4. `04_Row_vs_Statement_Level.md` – Granularity trade-offs & transition tables.
5. `05_Auditing_and_Change_Capture.md` – Audit patterns & CDC alternatives.
6. `06_Business_Rules_vs_Procedural_Logic.md` – Proper scope of triggers.
7. `07_Performance_and_Scalability_Impact.md` – Latency, contention, mitigation.
8. `08_Recursive_and_Nesting_Control.md` – Guards, recursion prevention.
9. `09_Security_and_Safeguards.md` – Privileges, injection prevention, integrity monitoring.
10. `10_AntiPatterns_and_Best_Practices.md` – Pitfall matrix & operational hygiene.

## Design Goals
Enable interview-ready understanding of triggers while emphasizing disciplined, minimal, and auditable use. Provide clear heuristics to decide when NOT to use triggers.

## Decision Heuristics
| Question | Consider |
|----------|----------|
| Can a constraint do this? | Prefer constraint/generator |
| Does logic need external service? | Move out of trigger |
| High-frequency table? | Ensure O(1) lightweight or avoid |
| Need per-row mutation? | Row-level BEFORE |
| Only need summary/log? | AFTER statement-level / transition table |
| Is recursion possible? | Add guard & idempotent predicates |

## Red Flags
- Multiple triggers overlapping same event
- Long-running queries inside triggers
- External network calls
- Circular table modification chain
- Unbounded audit table growth

## Performance Checklist
1. Time per trigger invocation (p95/p99)
2. Rows modified vs trigger executions
3. Contention hotspots (summary counters)
4. Recursion / nesting depth occurrences
5. Audit table size & partition health

## Security Checklist
- Least privilege on trigger functions
- No unchecked dynamic SQL
- Catalog diff monitoring
- Audit log tamper protection
- Sensitive field masking

## Practice Prompts
1. Convert redundant row-level audit to statement-level summary + selective detail.
2. Add recursion guard to existing self-updating trigger.
3. Replace trigger-based default population with generated column.
4. Propose migration away from trigger-based aggregation to async job.
5. Evaluate trigger necessity vs CHECK constraint for a positivity rule.

## Next Suggested Modules
- Partitioning & Data Lifecycle
- Advanced Window Functions Optimization
- Security Deep Dive (RLS, Encryption, Auditing)
- Change Data Capture & Streaming Patterns

Use with Stored Procedures & Performance Tuning modules to design safe, observable database behavior.
