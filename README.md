# Tech Hub Kenya: September 2026 Sales Analysis

Day 2 project for the Squad Python Specialization. Tech Hub Kenya sells phones, laptops and accessories from five stores. This script reads the till export, cleans the messy numeric values, and reports total revenue, the best-selling products and the high-value sales that deserve a follow-up call. It then checks its own results against pandas.

**Constraint:** the reporting server has no third-party packages, so the core analysis uses only the Python standard library. pandas is imported only for the final verification step, which is skipped with a message if pandas is missing.

## Project structure

```
day2-sales-analytics/
├── sales_analysis.py
├── data/electronics_sales.csv
├── output/sales_report.txt
├── requirements.txt          # pandas only (for verification)
└── README.md
```

## How to run

Requires Python 3.9 or newer.

```bash
git clone https://github.com/YOUR-USERNAME/day2-sales-analytics.git
cd day2-sales-analytics
python -m venv .venv
.venv\Scripts\activate           # Windows (Mac/Linux: source .venv/bin/activate)
pip install -r requirements.txt   # pandas, needed only for the verification step
python sales_analysis.py
```

The script prints the row count and column names, the sales report and the pandas verification, and saves the report to `output/sales_report.txt`. The `output/` folder is created if it is missing.

## Approach

| Step | What the code does |
|---|---|
| Read | `read_sales()` uses `csv.DictReader` and prints the row count and column names. |
| Convert | `parse_number()` repairs values that can be repaired safely and returns `None` for the rest. `convert_rows()` keeps clean rows and a list of skipped rows, each with a reason. |
| Revenue | A row's revenue is quantity × unit price. |
| Group | `revenue_by(rows, field)` uses a plain dict (no `defaultdict` or `Counter`) and is reused for product, store and payment method. |
| Rank | `top_n()` uses `sorted()` with a key function. |
| Filter | `high_value()` is a list comprehension. Counts per store use a dict comprehension. |
| Statistics | `mean`, `median` and `minimum_maximum` are written by hand, and `minimum_maximum` does not use `min()` or `max()`. Each raises a `ValueError` for an empty list. |
| Verify | `verify_with_pandas()` recomputes the headline numbers with the same cleaning rules and prints MATCH or MISMATCH. |

## Cleaning rules

Quantity and unit price arrive as text. The rule is: **repair only what is unambiguous, skip everything else**.

| Raw value | Decision | Why |
|---|---|---|
| Surrounding whitespace | **Repair** | The meaning is unambiguous. |
| Thousands commas (`1,200`) | **Repair** | The comma is a separator, not a decimal point. |
| `KES` prefix (e.g. `KES 18999`) | **Repair** | The column is already in KES, so the prefix adds no information. |
| Blank, `N/A`, `none`, `null`, `-` | **Skip** | No value exists, and guessing one would invent revenue. |
| Number words (`two`) | **Skip** | Your reason: for example, a number word is free text rather than a number, and I chose not to interpret free text in a financial column. |
| Other non-numeric text, `nan`, `inf` | **Skip** | It cannot be converted to a real amount without guessing. |

**Rows skipped in this dataset (4 of 50, so 46 were used):**

| Sale | Raw value | Reason |
|---|---|---|
| S-3006 |  raw quantity (`two`?) | Invalid quantity |
| S-3030 |  raw value | Invalid unit price |
| S-3035 |  raw value | Invalid quantity |
| S-3039 |  raw value | Invalid unit price |

Skipped rows are excluded from every total, count and statistic, and are listed in the report with their reason.  One sentence: why is skipping a row better than replacing a missing price with 0 or an average?

**Trade-off:** rejecting `two` removes one sale from the totals. Comparing runs shows the effect: total revenue would be KES 6,398 higher (about 0.5%) if it were repaired to 2, and Xiaomi Redmi Buds would be KES 31,990 instead of KES 25,592.  Confirm this against the CSV row for S-3006. This trade-off favours accuracy over completeness: no revenue is counted that was not clearly recorded.

 **Zero or negative values:** state either "no quantity or price is zero or negative" or "rows with a quantity or price of zero or below are skipped", whichever your code actually does.

## Tie-breaking and thresholds

- **Ties** in rankings and sorted printouts are broken by revenue descending, then by product name A to Z, so the order is always deterministic.  Confirm this matches the key in your `top_n`.
- **High-value** means revenue at or above KES 50,000 (`>=`). The smallest qualifying sale is KES 53,996, so no sale equals exactly 50,000 and the `>=` versus `>` choice does not change the result.

## Findings

- Total revenue for September 2026 was **KES 1,310,812.00** from 46 usable sales.
- The top three products are the **HP 15 Laptop** (KES 389,994.00), the **Hisense 43in Smart TV** (KES 272,993.00) and the **Lenovo Tab M10** (KES 199,992.00). Together they bring in about 66% of revenue.
- **Five high-value sales (≥ KES 50,000) make up about 53% of revenue** while being roughly 11% of transactions. **Kisumu Mega Plaza** had the most (2); Nairobi CBD, Nakuru Westside and Westgate had one each. These are the sales to follow up with a call.
- Transaction revenue is very uneven. The median is **KES 11,097.00**, but the mean is **KES 28,495.91** because a few large laptop and TV sales pull it up. The median is the better picture of a typical sale. The smallest sale was KES 1,499.00 and the largest KES 259,996.00.

## pandas verification

The check reads the CSV as text, applies the same cleaning rules, and recomputes the headline numbers. Latest run:

```
=== pandas verification ===
Total revenue      : MATCH
Revenue by product : MATCH
Top 3              : MATCH
Mean / median      : MATCH
```

All checks match, so there are no discrepancies to explain.

## Assumptions

- Revenue is quantity × unit price. The file has no revenue column.
- All prices are in KES.
- Sale IDs are unique (state that you checked).
- All sale dates fall in September 2026 (state that you checked).
- The data is a single export, with no refunds or cancellations to net off.

## Tech used

Python 3 standard library (`csv`, `math`, `io`, `pathlib`) and pandas, for verification only.