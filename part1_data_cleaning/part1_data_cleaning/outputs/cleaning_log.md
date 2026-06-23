# Cleaning Log — raw_orders.xlsx

## Issues Found

### 1. Text Inconsistencies
- **segment**: 16 unique variants for 4 values (Consumer, Small Business, Corporate, Home Office). Issues included leading/trailing spaces, all-caps, all-lowercase, and double internal spaces.
- **region**: 16 variants for 4 values (North, South, East, West). Same issues.
- **order_status**: 10 variants for 3 values (Completed, Cancelled, Returned).
- **payment_status**: 8 variants for 4 values (Paid, Refunded, Pending, Failed).
- **ship_mode**: 12 variants for 4 values (Second Class, Same Day, First Class, Standard Class).
- **category**: 13 variants for 3 values (Office Supplies, Furniture, Technology).
- **sub_category**: 23 variants for 13 unique sub-categories.
- **customer_name**, **state**, **city**: Leading/trailing spaces and case inconsistencies.

### 2. Missing Values
- `region`: 26 missing values
- `ship_mode`: 22 missing values
- `discount`: 18 missing values (stored as string, required numeric conversion)

### 3. Duplicate Records
- **Exact duplicate rows**: 20 rows were completely identical across all 21 columns.
- **Duplicate order_id with conflicting data**: 63 rows shared an order_id but had different field values.

### 4. Date Format Issues
- Both `order_date` and `ship_date` contained 4 mixed formats:
  - `21 Jul 2024`
  - `08/31/2024`
  - `28-11-2024`
  - `2024-05-24`
- 94 records had ship_date earlier than order_date (invalid shipping records).

### 5. Discount Issues
- 16 records had negative discount values (invalid).
- 18 records had missing discount values.
- No discounts exceeded 1.0 (100%).

### 6. Calculation Mismatches
- 57 records had a mismatch between reported `sales` and calculated `quantity × unit_price × (1 − discount)`.
- 41 records had a mismatch between reported `profit` and calculated `sales − cost`.

---

## Cleaning Actions Performed

| Action | Details |
|---|---|
| Text standardisation | Applied TRIM + PROPER + double-space removal to all 10 text columns |
| Missing region | Filled with "Unknown" |
| Missing ship_mode | Filled with "Unknown" |
| Missing discount | Treated as 0 where all other sales fields were valid |
| Date parsing | All 4 date formats parsed and standardised to DD/MM/YYYY |
| Exact duplicates | 20 rows removed (keep first occurrence) |
| Conflicting order_ids | 63 rows flagged in data_quality_flag column; not deleted |
| Negative discounts | Flagged as invalid; cleaned_discount set to NaN |
| Ship before order | 94 records flagged in data_quality_flag |

---

## Business Rules Applied

| Rule | Action Taken |
|---|---|
| Missing region | Filled as Unknown and flagged |
| Missing ship_mode | Filled as Unknown and flagged |
| Missing discount | Treated as 0 only if all other sales fields are valid |
| Negative discount | Flagged as Invalid in data_quality_flag |
| Discount > 1 | None found; would have been flagged |
| Cancelled orders | Excluded from completed sales pivot summaries |
| Failed payments | Excluded from completed sales pivot summaries |
| Refunded/Returned orders | Separately summarised in pivot and quality report |
| Ship date before order date | Flagged as invalid shipping record |

---

## Calculated Columns Added

| Column | Formula Logic |
|---|---|
| cleaned_discount | Standardised discount; negative → NaN; missing → 0 |
| calculated_sales | quantity × unit_price × (1 − cleaned_discount) |
| calculated_profit | calculated_sales − cost |
| profit_margin | calculated_profit / calculated_sales |
| shipping_delay_days | ship_date − order_date (days) |
| order_month | Month extracted from order_date |
| order_year | Year extracted from order_date |
| data_quality_flag | Clean / Warning with reason(s) |

---

## Assumptions Made

1. When discount is missing and all other sales fields are present and valid, discount is assumed to be 0.
2. For conflicting duplicate order_ids, the first occurrence is treated as the primary record; others are flagged but retained for review.
3. Date parsing uses MM/DD/YYYY as default where format is ambiguous (e.g., 08/05/2024 → August 5, not May 8).
4. Profit margin is calculated on `calculated_sales`, not the originally reported `sales`, to ensure consistency.
5. "Completed sales" for pivot summaries means order_status = Completed AND payment_status = Paid.

---

## Records Removed
- **20 exact duplicate rows** removed from cleaned_orders.xlsx.

## Records Flagged
- **188 records** flagged with Warning status (various reasons including duplicate order_id, negative discount, ship before order, missing region/ship_mode).
- **724 records** are Clean.

---

## Limitations

1. Date ambiguity: For dates like `05/06/2024`, it is impossible to determine definitively whether this is May 6 or June 5 without additional context.
2. Conflicting order_ids were not resolved — a business SME review is needed to determine the correct record.
3. Sales and profit mismatches were noted but original values were retained — the root cause (rounding, returns, manual adjustments) is unknown.
4. Customer name cleaning used PROPER case which may incorrectly capitalise names like "McDonald" as "Mcdonald".
