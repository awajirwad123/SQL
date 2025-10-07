# Deployment & Versioning

## Overview
Managing stored procedure lifecycle requires consistent version control, safe rollout practices, backward compatibility considerations, and traceability of changes. Poor process leads to drift across environments and hard-to-debug production issues.

## Core Concepts
- **Version Control**: Store CREATE scripts in repository; avoid editing ad-hoc in production.
- **Idempotent Scripts**: Use CREATE OR REPLACE / CREATE OR ALTER to streamline deployments.
- **Migration Tooling**: Liquibase, Flyway, sqitch, custom migration frameworks track applied revisions.
- **Backward Compatibility**: Avoid breaking parameter signatures in-place; use additive changes or new version name then deprecate old.
- **Change Audit**: Comment block with semantic version, author, date, purpose.
- **Rollback Plan**: Keep prior script; practice quick revert (feature flags at app layer for routing).
- **Signature Stability**: Parameter rename changes API; consider aliases / wrapper.
- **Testing**: Unit (expected outputs), performance baseline, error path simulation.

### Semantic Header Example
```sql
/* Procedure: dbo.CalcInvoice
   Version: 1.3.0
   Change: Added discount_rate parameter (optional)
   Author: J.Doe 2025-10-07 */
```

### Additive Change Pattern
1. Add new optional parameter with default.
2. Deploy application using it.
3. Remove old behavior after adoption window.

## Interview-Focused Notes
1. Why version procedures? Traceability + reproducibility.
2. How handle breaking change? Deploy new name; deprecate gradually.
3. Why idempotent DDL? Safe re-runs in CI/CD pipelines.
4. Rollback strategy? Keep prior script + revert deployment commit.
5. Migration order? Schema dependencies before procedures; drop obsolete last.

## Quick Recall ✅
- All code in repo.
- Use semantic headers.
- Prefer additive over destructive changes.
- Test performance pre/post deploy.
- Document rollback steps.

## Interview Traps & Confusions ⚠️
- Editing live without script capture.
- Dropping prior version before confirming compatibility.
- Forgetting to update dependency mapping docs.
- Uncontrolled drift between staging and prod.
- Big-bang parameter changes breaking older clients.

## Bonus
### Checksum Verification
Store hash of body in metadata table for drift detection.

### Deployment Pipeline Step
1. Apply schema migrations
2. Update reference data
3. Deploy procedures/functions
4. Run smoke tests

### Deprecation Log
Maintain table of soon-to-be-removed procedure names + target removal date.

### Automated Doc Generation
Parse headers to produce HTML/API spec for internal consumers.
