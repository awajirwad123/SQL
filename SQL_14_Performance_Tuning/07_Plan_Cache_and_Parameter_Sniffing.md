# Plan Cache & Parameter Sniffing

## Overview
Plan caching stores compiled execution plans for reuse. Parameter sniffing occurs when the optimizer compiles a plan using the initial parameter values; later executions with different distributions may perform poorly. Managing plan stability vs adaptability is key.

## Core Concepts
- **Compilation vs Execution**: Optimization phase uses sniffed parameter values for cardinality estimates.
- **Good Scenario**: Parameters representative → optimal reused plan.
- **Bad Scenario**: Atypical initial parameter (very selective or very unselective) leads to suboptimal reused plan.
- **Mitigations**:
  - Recompile hints / option (force new plan) sparingly.
  - Optimize for specific value / unknown (engine-specific options).
  - Plan guides or forced plan (SQL Server Query Store, Oracle SQL Plan Baselines).
  - Parameterization strategy (simple vs forced).
  - Use dynamic SQL for very skewed patterns (safely parameterized to avoid injection) to encourage tailored plans.
- **Histogram Sensitivity**: Out-of-range values degrade estimate quality.
- **Plan Eviction**: Cache size pressure causes recompile churn.

### Example (SQL Server Hint)
```sql
SELECT * FROM orders WHERE customer_id = @cust_id
OPTION (OPTIMIZE FOR UNKNOWN);
```

### Force Recompile (When Narrow Use Case)
```sql
SELECT * FROM orders WHERE customer_id = @cust_id
OPTION (RECOMPILE);
```

## Interview-Focused Notes
1. Define parameter sniffing: Compilation using initial parameter values affecting plan reused by others.
2. Why problematic? Different parameter distributions need different access paths (seek vs scan).
3. When use OPTIMIZE FOR UNKNOWN? To base estimates on average density not skewed first value.
4. Drawback of RECOMPILE? Higher CPU compilation overhead each execution.
5. Alternative to blanket hints? Targeted indexing, better stats, or query pattern segmentation.

## Quick Recall ✅
- Sniffing = first-parameter shaped plan reused.
- Skew + caching = potential mismatch.
- RECOMPILE = fresh plan each time (costly).
- OPTIMIZE FOR UNKNOWN = generic plan.
- Forced plan can stabilize regressions.

## Interview Traps & Confusions ⚠️
- Overusing recompile causing CPU spikes.
- Forcing plan ignoring future data growth.
- Treating symptom (sniffing) not root cause (skew + missing stats).
- Using dynamic SQL unsafely (injection risk).
- Ignoring plan cache pollution (many single-use queries).

## Bonus
### Identify Single-Use Plans (SQL Server)
Query Store / DMVs: high compile count, low reuse.

### Parameter Bucketization
Rewrite to categorize parameter ranges (e.g., small, medium, large) with dedicated logic.

### Postgres Note
Postgres generic vs custom plans threshold: uses custom for first few, may switch to generic if cheaper overall.

### Hybrid Mitigation
Use OPTION (RECOMPILE) only on small, highly volatile subset queries.

### Cache Bloat Control
Normalize literals (parameterization) to reduce unique plan entries.
