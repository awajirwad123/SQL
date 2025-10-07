# Performance Tuning Overview

## Overview
SQL performance tuning is the systematic process of reducing response time, resource consumption, and contention while preserving correctness. Effective tuning aligns query patterns, schema design, indexing, statistics, and workload management. It is iterative: Observe → Measure → Hypothesize → Change → Verify.

## Core Concepts
- **Workload Classification**: OLTP (short, selective, high concurrency) vs OLAP (scans, aggregations, batch). Different tuning levers.
- **Bottleneck Dimensions**: CPU, I/O, Memory (cache), Network, Lock/Contention.
- **Execution Plan**: Optimizer-produced blueprint of operator tree with estimated costs & cardinalities.
- **Cardinality Estimation**: Predicted row counts drive plan shape; stale/missing stats cause suboptimal joins or scans.
- **Access Paths**: Full scan, index seek, index scan, index-only scan, bitmap scan (DW), sampling.
- **Physical Join Algorithms**: Nested Loop, Hash Join, Merge Join (see join module for depth).
- **Cost-Based Optimization**: Choose lowest estimated cost path subject to heuristics & constraints.
- **Parameterization & Caching**: Reuse plans for efficiency; can cause parameter sniffing issues.
- **Feedback Loop**: Query store / plan cache / pg_stat_statements to identify top offenders.
- **80/20 Rule**: Focus on top N queries by cumulative resource footprint.

## Interview-Focused Notes
1. Define performance tuning: systematic optimization of query and resource usage based on measured evidence.
2. First step? Capture baselines (execution time, logical reads, row counts, plan hash).
3. Common root causes: Missing/ineffective indexes, bad cardinality stats, non-SARGable predicates, parameter sniffing, over-fetching.
4. Why analyze execution plan? Reveals chosen operators & estimated vs actual rows.
5. Difference optimizing query vs database config: Query tuning (logical rewrite, indexing) usually precedes knob twisting (memory, parallelism).

## Quick Recall ✅
- Measure before change.
- Fix data access (SARGability) first.
- Cardinality accuracy → good plan selection.
- Eliminate unnecessary rows early.
- Index only what workload proves.

## Interview Traps & Confusions ⚠️
- Premature index proliferation.
- Ignoring actual vs estimated row mismatch.
- Assuming CTEs always materialize (version dependent).
- Blaming optimizer before verifying statistics.
- Focusing on micro-optimizations ignoring missing predicate index.

## Bonus
### Tuning Loop Checklist
1. Identify high-cost query (time / IO / waits).
2. Capture current plan + metrics.
3. Validate row estimates vs actuals.
4. Improve predicate SARGability / indexes / rewrite.
5. Re-run; compare metrics; regress? revert quickly.

### High-Level Heuristics
| Symptom | Investigation |
|---------|---------------|
| Slow occasional query | Parameter sniffing / cache aging |
| High logical reads | Missing covering index / wide scan |
| Blocking chain | Lock waits / long transactions |
| CPU bound | Complex expressions / row-by-row UDF |
| I/O bound | Table scans / random I/O / fragmentation |

### Golden Rule
Change one variable at a time; validate with plan + metrics.
