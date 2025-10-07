# Statistical Functions

## Overview
Statistical (aggregate & window) functions surface distributional characteristics: averages, variance, standard deviation, percentiles, medians, ranking, and correlation—core to analytics and performance dashboards.

## Core Concepts
- Population vs sample functions: `VAR_POP`, `VAR_SAMP`, `STDDEV_POP`, `STDDEV_SAMP`.
- Percentiles & quantiles: `PERCENTILE_CONT`, `PERCENTILE_DISC`, vendor-specific `APPROX_PERCENTILE` / `NTILE`.
- Ranking windows: `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`.
- Correlation/Covariance: `CORR(x,y)`, `COVAR_POP`, `REGR_SLOPE` (Postgres / Oracle).
- Approximation functions for big data (HyperLogLog, t-digest percentile approximations) in warehouses.

**Examples:**
```sql
-- Basic dispersion
SELECT
  AVG(amount) AS mean_amount,
  STDDEV_POP(amount) AS std_pop,
  STDDEV_SAMP(amount) AS std_sample
FROM payments;

-- Median (continuous percentile)
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount) AS median_amount
FROM payments;  -- PG/Oracle

-- Top 3 per category
SELECT * FROM (
  SELECT category, product_id, revenue,
         ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
  FROM product_sales
) q
WHERE rn <= 3;

-- 95th percentile latency
SELECT PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY duration_ms)
FROM api_latency;
```

## Interview-Focused Notes
1. "Difference between RANK and DENSE_RANK?" – Gaps with RANK after ties; DENSE_RANK no gaps.
2. "Median vs average?" – Median robust to outliers; average influenced heavily.
3. "Approximate percentile tradeoff?" – Speed & memory vs exact precision.
4. "When use VAR_POP vs VAR_SAMP?" – Population when dataset is full population; sample when your data is a sample representing larger population.
5. "Why window over self-join?" – Window functions avoid row explosion and are more performant.

## Quick Recall ✅
- Median via `PERCENTILE_CONT(0.5)` (if supported) otherwise simulate with window sorting.
- Use `ROW_NUMBER` for deterministic tie-breaking.
- Approximate functions exist in warehouses (e.g., `APPROX_PERCENTILE`).
- Population vs sample divides by N vs (N-1).
- `NTILE(4)` splits into quartiles.

## Interview Traps & Confusions ⚠️
- Using AVG of averages (weighting issue) instead of recomputing from raw values.
- Misinterpreting percentiles (95th ≠ 95% of values below; rather value below which 95% lie).
- Large window sorts causing temp spill (optimize partitioning / indexing order).
- Forgetting ORDER BY requirement inside percentile clause.

## Bonus
### Manual Median (Odd/Even Compatible)
```sql
WITH ordered AS (
  SELECT amount,
         ROW_NUMBER() OVER (ORDER BY amount) AS rn,
         COUNT(*) OVER () AS cnt
  FROM payments
)
SELECT AVG(amount)::numeric AS median
FROM ordered
WHERE rn IN ( (cnt+1)/2, (cnt+2)/2 );
```

### Running Percentile (Cumulative Distribution)
```sql
CUME_DIST() OVER (PARTITION BY region ORDER BY revenue) AS cdf
```

### Z-Score Segmentation
```sql
CASE WHEN ABS(zscore) > 3 THEN 'Outlier' ELSE 'Normal' END
```

### Correlation Example
```sql
SELECT CORR(page_views, purchases) FROM user_daily_metrics; -- PG
```

### Rank vs Dense Rank Example
```sql
RANK() OVER (ORDER BY score DESC) vs DENSE_RANK() ...
```
