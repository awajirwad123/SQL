# Business Rules vs Procedural Logic

## Overview
Determining which logic belongs in triggers vs application or procedure layers is critical. Triggers should enforce invariant, data-centric rules (integrity, auditing) rather than orchestrate multi-step business workflows better handled elsewhere.

## Core Concepts
- **Data Integrity Enforcement**: Use constraints first (CHECK, FOREIGN KEY, UNIQUE) before triggers.
- **Invariant vs Process Flow**:
  - Invariant: Always true about data (e.g., quantity >= 0) → constraint/trigger (if constraint insufficient).
  - Process Flow: Sequencing of steps (email notifications, payments) → application/service.
- **Duplication Risk**: Business rules in triggers + app duplicates create divergence.
- **Isolation of Concerns**: Triggers small, single-purpose; orchestration outside.
- **Testability**: Harder to unit test distributed logic hidden in triggers.

### Good Trigger Rule Example
Ensure end_date >= start_date (if not expressible via simple CHECK due to dependency on dynamic default adjustments).

### Poor Trigger Use
Calling external API to send email inside high-volume INSERT trigger.

## Interview-Focused Notes
1. When put logic in trigger? Enforce critical invariants across all write paths.
2. When avoid triggers? Workflow orchestration, external side-effects, heavy computations.
3. Prefer constraint why? Declarative, optimized, clearer to tools & engine.
4. Risk of logic sprawl? Hard debugging, performance unpredictability.
5. Alternative pattern? Use stored procedure/API gateway to centralize multi-step operations.

## Quick Recall ✅
- Constraints first; triggers fallback.
- Data invariants inside; process orchestration outside.
- Keep triggers minimal & idempotent.
- Document purpose explicitly.
- Revisit triggers during schema change.

## Interview Traps & Confusions ⚠️
- Implementing validation easily handled by CHECK constraint.
- Embedding business workflow communications.
- Layering multiple triggers duplicating logic.
- Not re-validating triggers after refactor removing dependent columns.
- Assuming triggers scale linearly with volume.

## Bonus
### Trigger Registry Table
Metadata table listing purpose, owner, contact, last review date.

### Complexity Threshold
Refactor if trigger exceeds ~25 lines or has branching >2 levels.

### Consolidation Pass
Merge overlapping triggers to single guarded block.

### Migration Safety
Disable non-essential triggers during bulk backfill; run compensating verification after.
