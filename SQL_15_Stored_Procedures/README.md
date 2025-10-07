# SQL Stored Procedures Module (SQL_15_Stored_Procedures)

## Contents
1. `01_Stored_Procedures.md` – Purpose, encapsulation, advantages, risks.
2. `02_Create_Procedure.md` – Syntax across engines, idempotent creation.
3. `03_Parameters_and_Returns.md` – IN/OUT/INOUT, table-valued, return sets vs scalars.
4. `04_Control_Flow_and_Error_Handling.md` – TRY/CATCH, EXCEPTION, handlers.
5. `05_Transactions_in_Procedures.md` – Transaction scope, savepoints, nesting patterns.
6. `06_Result_Sets_vs_Output_Parameters.md` – Choosing return mechanisms.
7. `07_Security_and_Permissions.md` – Least privilege, execution context, injection mitigations.
8. `08_Performance_and_Caching.md` – Plan reuse, RBAR avoidance, instrumentation.
9. `09_Deployment_and_Versioning.md` – Version control, change management, rollback.
10. `10_AntiPatterns_and_Best_Practices.md` – Pitfall matrix & guidance.

## Design Goals
Provide an interview-focused, production-aware foundation for authoring, securing, deploying, and tuning stored procedures across major RDBMS platforms.

## Decision Heuristics
| Question | Heuristic |
|----------|-----------|
| Should logic live in DB? | Data integrity & cross-app consistency justify; else service layer |
| Function or procedure? | Need composability/returns scalar/table → function; multi-step side-effecting → procedure |
| OUT param or result set? | Scalar meta → OUT; tabular → result set |
| Dynamic SQL needed? | Only with vetted whitelist + parameters |
| Loop or set-based? | Prefer set-based unless proven impractical |

## Red Flags
- Monolithic 1000+ line procedures.
- Cursor loops for simple set modifications.
- Mixed unrelated result sets.
- Embedded credentials / unparameterized dynamic SQL.
- No version header or audit trail.

## Performance Checklist
1. Plan reuse vs parameter sniffing balance.
2. Avoid SELECT *; project minimal columns.
3. Table-valued parameters / staging tables for bulk sets.
4. Instrument execution time + logical reads.
5. Refresh stats after large batch loads.

## Security Checklist
- EXECUTE rights only to application role.
- Revoke direct table DML from consumers.
- Parameterize dynamic SQL.
- Log privileged operations.
- Review definer/invoker contexts quarterly.

## Practice Prompts
1. Convert cursor-based row update into set-based merge/update.
2. Add transactional safety + error handling to a multi-step transfer procedure.
3. Introduce table-valued parameter for bulk delete of ids.
4. Harden dynamic sort procedure against injection.
5. Implement versioned deployment with rollback script.

## Next Suggested Modules
- Triggers & Auditing
- Advanced Window Functions Optimization
- Partitioning & Lifecycle Management
- Security (RLS, Encryption, Auditing) Deep Dive

Use with Performance Tuning + Indexes modules to reason about execution & scaling impacts.
