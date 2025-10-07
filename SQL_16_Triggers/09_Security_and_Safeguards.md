# Security & Safeguards

## Overview
Triggers run with implicit privileges that can elevate risk if logic is exploitable (dynamic SQL, external calls). Proper security patterns limit abuse and ensure auditability.

## Core Concepts
- **Execution Context**: Owner rights (e.g., definer) vs invoker affects accessible objects.
- **Privilege Minimization**: Trigger function owner should have only required rights.
- **Immutable vs Mutable Code**: Use signature or checksum verification for critical triggers.
- **Injection Risks**: Dynamic SQL inside trigger subject to user-supplied data—parameterize or whitelist.
- **Auditable Trail**: Log trigger-created admin actions separately for traceability.
- **Tamper Detection**: Monitor system catalogs for unexpected trigger creation/alteration.
- **Error Handling**: Fail fast & loud—silent swallow hides compromise.

### Owner Isolation (Postgres)
Create trigger function under dedicated role with least privileges; table grants limited.

### Whitelisting Dynamic Column Use
Validate input against approved list before building dynamic statement.

## Interview-Focused Notes
1. How can trigger elevate privilege? Runs under definer; may access restricted tables.
2. Risk of dynamic SQL inside trigger? Injection & privilege escalation.
3. Protect audit triggers? Restrict DROP/ALTER; monitoring job compares checksum.
4. Why fail loudly? Silent failures mask integrity/security breaches.
5. Catalog monitoring? Detect unauthorized trigger insertions.

## Quick Recall ✅
- Least privilege for trigger owner.
- No unvalidated dynamic SQL.
- Catalog change monitoring.
- Separate audit log for sensitive operations.
- Version & checksum critical triggers.

## Interview Traps & Confusions ⚠️
- Assuming triggers inherit caller identity (often definer context).
- Logging sensitive data (PII) without masking.
- Over-broad privileges on trigger function schema.
- Ignoring encryption at rest for audit log.
- Forgetting to revoke default PUBLIC execute on helper functions.

## Bonus
### Integrity Watchdog
Scheduled job compares pg_trigger / sys.triggers definitions vs golden repository hash.

### Masking Strategy
Store hashed sensitive field deltas instead of raw values.

### Compartmentalization
Separate schema for utility trigger functions distinct from application tables.

### Change Approval
Require code review & signature before deploying trigger changes.
