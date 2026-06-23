# Part 1: Business Data Cleaning, Validation & Excel Reporting

## Problem Summary
A retail company exported order-level sales data from multiple internal systems. The raw dataset contained issues including inconsistent text formatting, mixed date formats, duplicate records, missing values, invalid discounts, calculation mismatches, and order status inconsistencies. The goal was to produce a clean, validated, analysis-ready dataset along with summary reports for business review.

---

## Dataset Description
- **File**: `data/raw_orders.xlsx`
- **Rows**: 932 records
- **Columns**: 21 fields
- **Fields**: order_id, order_date, ship_date, customer_id, customer_name, segment, region, state, city, category, sub_category, product_name, ship_mode, quantity, unit_price, discount, sales, cost, profit, payment_status, order_status

---

## Tools Used
- Microsoft Excel (manual inspection, TRIM, PROPER, SUBSTITUTE, Text to Columns, Pivot Tables)
- Python (pandas, openpyxl, dateutil) for automated cleaning and report generation

---

## Cleaning Steps Performed

1. **Text Standardisation**: Applied TRIM + PROPER + double-space removal to 10 text columns (segment, region, state, city, category, sub_category, ship_mode, payment_status, order_status, customer_name).
2. **Missing Value Handling**: Filled missing `region` (26) and `ship_mode` (22) with "Unknown". Treated missing `discount` (18) as 0 where other sales fields were valid.
3. **Date Standardisation**: Parsed all 4 mixed date formats and standardised to DD/MM/YYYY. Added `shipping_delay_days` column.
4. **Duplicate Removal**: Removed 20 exact duplicate rows. Flagged 63 conflicting duplicate order_ids for review.
5. **Discount Validation**: Flagged 16 negative discounts as invalid. No discounts exceeded 100%.
6. **Calculated Columns**: Added 8 new columns (see below).
7. **Data Quality Flagging**: Every record tagged as Clean or Warning with specific reason(s).

---

## Business Rules Applied

| Rule | Action |
|---|---|
| Missing region | Filled as Unknown, flagged |
| Missing ship_mode | Filled as Unknown, flagged |
| Missing discount | Treated as 0 (valid fields), flagged |
| Negative discount | Flagged as Invalid |
| Cancelled orders | Excluded from completed sales summaries |
| Failed payments | Excluded from completed sales summaries |
| Refunded/Returned | Separately summarised |
| Ship date before order date | Flagged as invalid |

---

## Data Quality Issues Found

| Issue | Count |
|---|---|
| Missing region | 26 |
| Missing ship_mode | 22 |
| Missing discount | 18 |
| Exact duplicate rows | 20 |
| Duplicate order_ids (conflicting) | 63 |
| Negative discounts | 16 |
| Ship before order date | 94 |
| Sales calculation mismatch | 57 |
| Profit calculation mismatch | 41 |
| **Total Clean records** | **724** |
| **Total Warning records** | **188** |

---

## Calculated Columns Added

| Column | Description |
|---|---|
| cleaned_discount | Standardised discount value |
| calculated_sales | quantity × unit_price × (1 − cleaned_discount) |
| calculated_profit | calculated_sales − cost |
| profit_margin | calculated_profit / calculated_sales |
| shipping_delay_days | ship_date − order_date (days) |
| order_month | Month from order_date |
| order_year | Year from order_date |
| data_quality_flag | Clean or Warning with reason |

---

## Pivot Summary Reports

Six pivot reports created in `outputs/pivot_summary.xlsx`:

1. **Sales by Region** — Total sales, profit, order count, and profit margin % by region. Sorted by total sales descending.
2. **Sales by Category** — Sales and profit broken down by category and sub-category. Sorted by sales within each category.
3. **Orders by Ship Mode** — Order count and average shipping delay by ship mode. Sorted by order count descending.
4. **Profit by Segment** — Total sales, profit, and profit margin % by customer segment. Sorted by profit descending.
5. **Issues by Region** — Count of cancelled, returned, and failed payment orders by region.
6. **Monthly Sales Trend** — Total sales, profit, and order count by year and month.

---

## Key Business Insights

1. **94 orders had ship dates earlier than order dates** — this indicates data entry errors or system sync issues that need investigation.
2. **57 sales calculation mismatches** — reported sales figures do not match quantity × unit_price × discount logic, suggesting possible manual overrides or system discrepancies.
3. **20.1% of orders (188/932) were flagged** — a significant portion of data has quality issues that could affect reporting accuracy.
4. **Cancelled and failed payment orders** account for a portion of records and must be excluded from revenue reporting.

---

## Assumptions and Limitations

1. Missing discount treated as 0 when all other sales fields are valid.
2. Conflicting duplicate order_ids retained and flagged — not deleted without business confirmation.
3. Date ambiguity resolved using MM/DD/YYYY as default where format is unclear.
4. Original reported sales and profit values retained alongside calculated versions for comparison.
5. PROPER case applied to names may cause minor formatting issues (e.g., "Mcdonald" instead of "McDonald").

---

## Screenshots Included

| File | Shows |
|---|---|
| `screenshots/raw_data_preview.png` | Raw dataset before cleaning |
| `screenshots/cleaned_data_preview.png` | Cleaned dataset with calculated columns |
| `screenshots/pivot_summary_1.png` | Sales and profit by region pivot |
| `screenshots/pivot_summary_2.png` | Monthly sales trend pivot |
