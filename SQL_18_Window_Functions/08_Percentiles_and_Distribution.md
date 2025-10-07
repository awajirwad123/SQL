# Percentiles & Distribution Functions

## Overview
Distribution-oriented window functions compute relative standing or quantile boundaries within partitions: ranking proportions, cumulative distributions, and exact/approximate percentiles.

## Functions
- `PERCENT_RANK()` – (rank - 1) / (rows_in_partition - 1)
- `CUME_DIST()` – cumulative distribution: number of rows with value <= current / total rows.
- `NTILE(n)` – bucket index (approximate quantile grouping).
- `percentile_cont(p) WITHIN GROUP (ORDER BY col)` – Continuous percentile (interpolates).
- `percentile_disc(p) WITHIN GROUP (ORDER BY col)` – Discrete percentile (returns actual row value).
- Approximate percentile functions (`APPROX_PERCENTILE`, `PERCENTILE_CONT` with quantile arrays in some engines, t-digest / kll algorithms) depending on platform.

### Examples
```sql
SELECT
  val,
  PERCENT_RANK() OVER (ORDER BY val) AS pr,
  CUME_DIST()    OVER (ORDER BY val) AS cd,
  NTILE(4)       OVER (ORDER BY val) AS quartile
FROM metrics;

-- Exact median (continuous)
SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY val) AS median FROM metrics;

-- Discrete 95th
SELECT percentile_disc(0.95) WITHIN GROUP (ORDER BY latency_ms) AS p95 FROM latency_samples;
```

## Use Cases
- Percentile latency reporting (p50, p90, p95, p99)
- Customer spend percentile classification
- Quantile-based bucket pricing
- Detect outliers (values above p99)

## Interview-Focused Notes
1. PERCENT_RANK start & end? First row 0, last row 1 (unless single row → undefined denominator; returns 0 per engine).
2. CUME_DIST vs PERCENT_RANK difference? CUME_DIST counts current rank group fully; PERCENT_RANK uses rank-1 formula.
3. percentile_cont vs percentile_disc? Continuous may interpolate, discrete chooses actual row.
4. NTILE vs percentile functions? NTILE groups into n buckets approximating quantiles, not exact boundaries.
5. Approximate percentile advantage? Faster on large data with acceptable error margin.

## Quick Recall ✅
- Use percentile_disc for actual value; percentile_cont for interpolated.
- PERCENT_RANK 0..1 endpoints; CUME_DIST can jump above theoretical percentile due to ties.
- NTILE rough cut; use precise percentile functions for accuracy.
- Pre-aggregate then percentile for performance.
- Outlier detection pairs with distribution metrics.

## Interview Traps & Confusions ⚠️
- Treating NTILE quartiles as exact Q1/Q3.
- Using percent_rank to assign deciles (inaccurate for grouping).
- Ignoring tie impact on cumulative distribution.
- Not casting to numeric causing integer division issues.
- Percentile on skewed distribution misinterpreted (long tail illusions).

## Bonus
### Multiple Percentiles Single Pass (Postgres)
```sql
SELECT percentile_cont(ARRAY[0.5,0.9,0.95,0.99]) WITHIN GROUP (ORDER BY latency_ms) AS pcts
FROM latency_samples;
```

### Approximate (Engine Example)
`SELECT approx_percentile(val, 0.95) FROM big_table;` (Presto/Trino style)

### Outlier Flag
```sql
WITH p AS (
  SELECT percentile_cont(0.99) WITHIN GROUP (ORDER BY val) AS p99 FROM metrics
)
SELECT m.*, CASE WHEN m.val > p.p99 THEN 1 END AS is_outlier
FROM metrics m CROSS JOIN p;
```

### Weighted Percentiles
Some engines support `percentile_cont` over ordered by value repeated weight times or custom aggregate.
