# 5. NULL Handling

Q: What is NULL?
A: Unknown / missing value—not equal to anything, including another NULL.

Q: Test for NULL?
A: IS NULL / IS NOT NULL (never = NULL).

Q: Aggregation nuance?
A: COUNT(col) skips NULL; COUNT(*) counts all rows.

Q: Safe substitution?
A: COALESCE(col, fallback_value).

Q: Classic trap?
A: WHERE id NOT IN (subquery) with NULL inside subquery → returns no rows; use NOT EXISTS instead.
