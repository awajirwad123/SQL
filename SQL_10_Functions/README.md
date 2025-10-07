# SQL Functions Module (SQL_10_Functions)

This module covers function categories and specific string functions crucial for interviews and practical SQL engineering:

## Files
1. `01_Date_Functions.md` – Extraction, truncation, arithmetic, timezone, bucketing.
2. `02_String_Functions.md` – Case, substring, trim, search, replace, normalization.
3. `03_Numeric_Functions.md` – Arithmetic precision, rounding, safe division.
4. `04_Statistical_Functions.md` – Variance, STDDEV, percentiles, ranking windows.
5. `05_JSON_Functions.md` – Semi-structured navigation, indexing, validation.
6. `06_Conversion_Functions.md` – CAST/CONVERT/TO_*, safe parsing patterns.
7. `07_Datatype_Functions.md` – Type introspection, UUIDs, arrays, domains.
8. `08_LTRIM_Function.md` – Left-side trimming specifics.
9. `09_UPPER_Function.md` – Case normalization & indexing strategies.
10. `10_RTRIM_Function.md` – Right-side trimming, fixed-width cleanup.

## How to Use
- Skim categories 1–7 for conceptual breadth.
- Use 8–10 for quick recall of trimming & case normalization nuances.
- Pair with previous joins and constraints modules for holistic querying capability.

## Suggested Next Deep Dives
- LOWER / TRIM unified patterns
- Window function advanced (LAG, LEAD, percent rank)
- Error handling & TRY_* functions across dialects

## Practice Prompts
1. Normalize and deduplicate emails ignoring case & whitespace.
2. Compute month-over-month growth using date truncation.
3. Extract customer id from JSON payload and index it.
4. Calculate 95th percentile latency per service daily.
5. Convert raw text currency to DECIMAL safely and aggregate by week.

Use these as flashcards before interviews.
