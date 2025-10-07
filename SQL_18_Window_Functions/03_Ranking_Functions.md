# Ranking Functions

## Overview
Ranking window functions assign ordinal or bucket positions within a partition. They support pagination, top-N per group, tie handling, and distribution analyses.

## Functions
- `ROW_NUMBER()`: Unique sequential number per row (deterministic only with ORDER BY + unique tiebreaker).
- `RANK()`: Like competition ranking (1,2,2,4) – gaps after ties.
- `DENSE_RANK()`: No gaps (1,2,2,3).
- `NTILE(n)`: Divides ordered partition into n buckets (roughly equal size), output bucket index (1..n).

### Examples
```sql
SELECT o.*, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC) AS rn
FROM orders o;

SELECT p.*, RANK() OVER (ORDER BY price DESC) AS price_rank
FROM products p;

SELECT p.*, DENSE_RANK() OVER (ORDER BY category, price) AS cat_price_rank
FROM products p;

SELECT user_id, NTILE(4) OVER (ORDER BY total_spend DESC) AS quartile
FROM user_spend;
```

## Use Cases
- Pagination per category
- Top seller per region (ROW_NUMBER filter = 1)
- Price/tier grouping (NTILE quartiles)
- Tie-sensitive leaderboards (RANK vs DENSE_RANK choice)

## Interview-Focused Notes
1. Difference RANK vs DENSE_RANK? RANK leaves gaps; DENSE_RANK doesn’t.
2. Why ROW_NUMBER for dedupe? Keep first row per group & discard rest.
3. NTILE caveat? Buckets differ by at most one row; not exact percentile.
4. Determinism requirement? Add secondary ORDER BY columns for stable ranking.
5. Filtering top-N per group pattern? Wrap ranking in subquery and filter on rank/row_number.

## Quick Recall ✅
- ROW_NUMBER for unique order + dedupe.
- DENSE_RANK compresses gaps.
- RANK shows competitive placement.
- NTILE approximates distribution buckets.
- Always specify ORDER BY for deterministic results.

## Interview Traps & Confusions ⚠️
- Using RANK when contiguous ranking expected.
- Assuming NTILE(4)=quartile boundaries exactly (approx).
- Omitting ORDER BY causing arbitrary ordering.
- Relying on non-unique ORDER BY leading to unstable output.
- Using ranking as substitute for true percentage (use percent_rank / cume_dist / percentile_cont).

## Bonus
### Top 3 Products per Category
```sql
SELECT * FROM (
  SELECT p.*, ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rn
  FROM product_sales p
) x WHERE rn <= 3;
```

### Tie-Aware Podium
```sql
SELECT * FROM (
  SELECT athlete, RANK() OVER (ORDER BY finish_time) AS r
  FROM race_results
) r WHERE r <= 3;
```

### De-Duplicate Keeping Latest
```sql
SELECT * FROM (
  SELECT t.*, ROW_NUMBER() OVER (PARTITION BY natural_key ORDER BY updated_at DESC) AS rn
  FROM staging t
) s WHERE rn = 1;
```

### Bucket Spend into Deciles
```sql
NTILE(10) OVER (ORDER BY total_spend DESC) AS spend_decile
```