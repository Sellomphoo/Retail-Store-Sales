# Retail Store Sales — Data Cleaning, Analysis & Dashboard

## Overview

The **Dirty Retail Store Sales** dataset contains 12,575 rows of synthetic data representing sales transactions from a retail store. The dataset spans eight product categories, each with 25 items at static prices, and is deliberately "dirty" — containing missing and inconsistent values — to simulate real-world sales data.

This project takes that raw dataset through cleaning, validation, calculation, and visualization, producing a dashboard summarizing revenue, quantity, discount behavior, purchase location, and payment method across categories.

## Key characteristics of the raw dataset

- **Missing data**: columns including `Item`, `Price Per Unit`, `Quantity`, and `Total Spent` contain missing values, simulating real-world inconsistencies.
- **Item probabilities**: items within each category appear with different frequencies rather than uniformly.
- **Transaction uniqueness**: `Transaction ID` is always unique and present; `Customer ID` is always valid, drawn from 25 unique customers.
- **Dynamic dirtiness**: some transactions are only partially complete, requiring inference to recover missing values (e.g. deducing `Item` from `Price Per Unit` and `Total Spent`).

## Problem statement

Given a 12,575-row transactional dataset with missing and inconsistent values, the goal was to:

1. Clean and validate the data without fabricating information that couldn't be reasonably inferred.
2. Verify that cleaned/calculated figures were correct before using them in analysis.
3. Build a dashboard summarizing revenue, sales volume, discount behavior, and channel/payment patterns by category.
4. Present findings honestly, including flagging where the data shows little variation rather than forcing a narrative onto it.

## Cleaning process

### Numeric columns: `Price Per Unit`, `Quantity`, `Total Spent`

These three columns have a fixed relationship: `Total Spent = Price Per Unit × Quantity`. Where exactly one of the three was blank, it was recovered by calculation rather than estimation:

- Blank `Total Spent` → `=Price Per Unit × Quantity`
- Blank `Quantity` → `=Total Spent ÷ Price Per Unit`
- Blank `Price Per Unit` → `=Total Spent ÷ Quantity`

Formulas were wrapped in `IFERROR`/`ISNUMBER` checks to avoid `#VALUE!` errors from stray blank or text-formatted cells. Rows with **two or more** of these three values missing could not be back-calculated and were left as-is rather than guessed, since fabricating either value would have quietly corrupted revenue totals.

### Text column: `Item`

The raw `Item` values followed the pattern `Item_<number>_<ABBREVIATION>` (e.g. `Item_10_PAT`). Since only the abbreviation was needed, `TEXTAFTER(cell, "_", -1)` was used to extract everything after the final underscore, wrapped in `IFERROR` to leave already-blank cells untouched.

For rows where `Item` was blank outright, the initial plan was to infer it from `Category`. This was tested against the actual 12,575-row dataset using a PivotTable (Category → Item, Min/Max price) and confirmed: **each Category maps to exactly one Item abbreviation** across the full dataset (e.g. Patisserie → PAT, Milk Products → MILK). This meant a direct `VLOOKUP` against a Category → Item table was a safe, non-fabricated way to fill blanks — unlike a scenario with multiple items per category, where guessing would not have been justified.

## Validation / Calculations sheet

A separate **Calculations** sheet was built to independently verify every figure appearing on the dashboard, rather than trusting PivotTable output at face value. This step caught real errors:

- **Payment Method Per Category** and **Category Sold On Discount**: several `COUNTIFS` formulas referenced a single fixed cell in the raw data sheet instead of the column header in the Calculations sheet itself, producing duplicated values across categories (e.g. Milk Products showing identical counts for Cash, Credit Card, and Digital Wallet) and grand totals that didn't reconcile to 12,575.
- **Revenue Per Month**: a `SUMIF` formula had the same issue, causing March/April/May and January/September/October to return identical (incorrect) totals.

**Fix**: criteria arguments were corrected to reference the row/column labels within the Calculations sheet itself (using mixed `$` references so the formula could be copied across the full table), rather than a fixed cell in the raw data. After the fix, all totals reconciled to 12,575 transactions.

This step reinforced a core principle followed throughout the project: **a pivot or summary number is only trustworthy once it's been independently cross-checked**, not assumed correct because it renders cleanly.

## Dashboard

The dashboard summarizes:

- Total revenue, and revenue by category
- Total quantity sold, and quantity by category
- Discount usage (TRUE/FALSE) by category
- Purchase location split (Online vs. In-store)
- Payment method split (Cash / Credit Card / Digital Wallet) by category
- Revenue by month and by year

### Color palette

A consistent color system was applied across all charts so the dashboard reads as one coherent design rather than independently styled charts:

| Meaning | Color |
|---|---|
| Primary metric / Online / Discount = TRUE | Navy blue |
| Secondary metric / In-store | Warm orange |
| Highlighted top performer | Gold |
| De-emphasized comparison (e.g. Discount = FALSE) | Light blue-gray |
| KPI call-out numbers | Deep teal |

Card and page backgrounds were set to neutral off-white/light gray rather than mismatched pastel fills, so color is used to carry meaning rather than decoration.

## Findings

- **Revenue is unusually evenly distributed across categories.** Total revenue is R1,552,071 across 8 categories, with every category falling in a tight R180,000–R208,000 band (Butchers highest, Milk Products lowest) — only a ~15% spread top to bottom.
- **Quantity sold doesn't perfectly track revenue.** Furniture leads in units sold (8,462) but is only mid-pack in revenue, while Butchers has fewer transactions but the highest revenue — implying a meaningfully higher average price per unit for Butchers.
- **Discount usage is close to a 50/50 split in every single category**, with no category discounted disproportionately more or less than others.
- **Purchase location is close to even** (Online 6,354 vs. In-store 6,221).
- **Payment method is close to a three-way even split** (Cash, Credit Card, Digital Wallet), both overall and within each category individually.
- Taken together, the uniformity across nearly every dimension suggests this dataset was synthetically generated with balanced/randomized distributions rather than reflecting organic customer behavior. This is a legitimate finding in itself — the near-total absence of variation is worth stating explicitly rather than forcing charts to imply patterns the data doesn't support.

## Caveats and limitations

- **No profit data.** The dataset includes revenue (`Total Spent`) but no cost/COGS column, so profitability cannot be calculated — all revenue figures should be read as sales volume, not profit.
- **Rows with two or more missing numeric values** (`Price Per Unit`, `Quantity`, `Total Spent`) were left incomplete rather than estimated, since there was no reliable way to recover them without fabricating data.
- **Monthly/yearly revenue figures** depend on `Transaction Date` parsing; 2025 revenue appears low relative to other years because the dataset's 2025 records are partial (year in progress), not because of a genuine sales decline.
- This dataset is synthetic and intended for practicing data cleaning and analysis — findings above describe patterns in this dataset, not real market behavior.

## Tools used

- Microsoft Excel (formulas, PivotTables, Power Query concepts, native charts)
- Manual verification via a dedicated Calculations sheet

## Dataset link
- https://www.kaggle.com/datasets/ahmedmohamed2003/retail-store-sales-dirty-for-data-cleaning/data
