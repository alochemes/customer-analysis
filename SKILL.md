---
name: customer-analysis
description: Use this skill when the user wants to analyze customer transaction data to produce cohort retention curves, customer segmentation (whales/regulars/churned), revenue concentration, and trend analysis. Triggers include phrases like "run the customer analysis", "analyze my customers", "compute cohort retention", "segment my customers by revenue", or when the user uploads a transaction CSV and asks for an analytical report. Do NOT use this skill for one-time-purchase businesses, anonymous-transaction retail, or pre-revenue companies — confirm the business fits the pattern before running.
---

# Customer Analysis Skill

## What this skill does

Takes a customer transaction file as input. Produces:

1. A revenue overview (totals, growth, customer counts by period)
2. Customer segmentation into whales / regulars / small accounts / churned / one-time
3. Cohort retention analysis at N+1, N+3, N+6, N+12 periods
4. Per-customer trajectory analysis (who's growing, who's shrinking)
5. Anomaly flags (concentration risks, one-time spikes, cohort degradation)
6. An executive summary

Deliverables: an Excel workbook with all tables, a markdown executive summary, and a few interactive HTML charts.

## When to use this skill

Use when the user has:
- A transaction-level dataset with identifiable customers and repeat transactions
- At least ~6 months of history and ~30 customers (otherwise the math doesn't stabilize)
- A B2B SaaS, subscription, marketplace, professional services, or repeat-purchase D2C business

Do NOT use this skill for:
- One-time-purchase businesses (mattresses, weddings, funerals)
- Anonymous retail where customers can't be linked across transactions
- Pre-revenue companies
- Project-based businesses with single large contracts and very few customers

If the data doesn't fit, stop and explain why before running.

## Workflow

Follow these steps in order. Do not skip the early steps even if the user is impatient — bad assumptions in step 1 corrupt everything downstream.

### Step 1 — Locate and inspect the data

Look in the working directory and `/mnt/user-data/uploads` for a transaction file. Common formats: `.csv`, `.xlsx`, `.tsv`. If there are multiple files, ask which one.

Once located, run a quick inspection:

```python
import pandas as pd

df = pd.read_csv(PATH)  # or read_excel
print(f"Shape: {df.shape}")
print(f"Columns: {df.columns.tolist()}")
print(df.head(10))
print(df.dtypes)
print(f"Nulls per column:\n{df.isna().sum()}")
```

Identify:
- The customer identifier column (might be named `customer_id`, `customer`, `account`, `practice`, `client`, etc.)
- The period or date column
- The amount column
- Format: long (one row per customer-period) or wide (customers as rows, periods as columns)

If wide format, pivot to long before proceeding:

```python
df = df.melt(id_vars=[customer_col], var_name="period", value_name="amount")
df = df[df["amount"].notna() & (df["amount"] != 0)]
```

### Step 2 — Surface data quality issues to the user

Before running any analysis, report:

- Time range covered (min and max period)
- Number of unique customers
- Total revenue (sum of amounts)
- Any negative amounts (refunds/chargebacks) — count and total
- Any customers with multiple very-similar names (run a fuzzy match for likely duplicates)
- Whether the most recent period appears partial (suspiciously low vs. prior periods)
- Whether the earliest periods appear suspiciously sparse (data window may predate the real business start)

Present these findings and ask the user to confirm:

1. Should we include or exclude the most recent partial period?
2. Are the likely duplicate customer names actually the same entity (propose merges)?
3. Should refunds net against revenue, or be analyzed separately?
4. What defines "active" in a period (default: any positive transaction; can be raised for noisy data)?

Wait for confirmation before proceeding. Default to sensible choices but get explicit sign-off on anything that's a real judgment call.

### Step 3 — Build the canonical dataset

Once decisions are confirmed, produce a single clean DataFrame with three columns: `customer_id`, `period`, `amount`. This is the input to all downstream analysis.

Save a copy to the working directory as `customer_data_clean.csv` so the user can audit the cleanup.

### Step 4 — Revenue overview

Compute and save to the workbook (sheet: "Revenue Overview"):

- Total revenue by period
- Period-over-period growth rate (%)
- Active customer count by period
- New customers per period (first-ever transaction)
- Churned customers per period (was active 1-3 periods ago, not active now)

Add a chart: revenue and active-customer count on a dual-axis line chart.

### Step 5 — Customer segmentation

Compute total revenue per customer over the full window. Segment:

- **Whales:** top 10% by revenue
- **Regulars:** customers in the 50-90 percentile range with activity in at least 50% of their possible periods
- **Small accounts:** bottom 50% by revenue, with activity in at least 25% of their possible periods
- **Churned:** previously active (≥3 active periods at some point), but no activity in the last 60-90 days of the window
- **One-time:** active in exactly one period

Save to the workbook (sheet: "Segmentation"):

- Customer count and revenue share by segment
- Top 10 customers by revenue (name + total + first period + last period)
- Revenue concentration table: top 10% / 20% / 50% revenue share
- Concentration trend: same concentration figures computed for the first half of the window vs. the second half. **Flag if concentration is increasing.**

### Step 6 — Cohort retention

A customer's cohort is the period of their first transaction.

For each cohort, compute:

- N (cohort size: number of customers in the cohort)
- N+1 active count and rate
- N+3 active count and rate
- N+6 active count and rate
- N+12 active count and rate (only if data window allows)

Critical:

- Only compute milestones for cohorts that have had time to reach them. Younger cohorts get "n/a" — do not compute a partial milestone that will be artificially flattering.
- Produce two views:
  - **Full view:** all cohorts, with n/a where appropriate
  - **Apples-to-apples view:** only cohorts old enough to have reached the longest milestone you're showing

Save to the workbook (sheet: "Cohort Retention").

Render the cohort retention curves as an interactive HTML chart in the working directory (one line per cohort, x-axis = months since first transaction, y-axis = retention rate).

**Compare newer cohorts to older cohorts.** If newer cohorts retain worse than older ones, this is the most important finding in the entire analysis. Call it out explicitly in the summary, with the specific numbers, and explain what it implies (growth is borrowed from the future).

### Step 7 — Per-customer trajectory

For customers active in at least 6 periods, compute:

- Average revenue in their first third of activity
- Average revenue in their last third of activity
- Trajectory: growing (>15% increase), flat (within ±15%), shrinking (>15% decrease)

Aggregate: what share of long-tenured customers fall in each bucket?

Surface notable individual cases:

- Largest expansion (% growth)
- Largest contraction (% decline)
- Customers who went silent after high engagement
- Customers who reactivated after a long gap

Save to the workbook (sheet: "Trajectories").

### Step 8 — Anomaly scan

Run these checks and produce a flags list:

- Any single customer >20% of total revenue (concentration risk)
- Any single period with revenue >2x the median (one-time spike)
- Any customer with a single anomalously large transaction relative to their own history
- Periods with suspiciously low active-customer counts
- Cohort retention degrading by more than 10 points between oldest and newest comparable cohorts

Save to the workbook (sheet: "Flags").

### Step 9 — Executive summary

Write a markdown file `executive_summary.md` in the working directory, no more than one page, covering:

1. **Headline:** is the business growing, and is the growth healthy?
2. **Top finding:** the single most important pattern (often cohort degradation or revenue concentration, but trust the data)
3. **Key numbers:** total revenue, period-over-period growth, top-decile revenue share, longest-window retention rate
4. **Three things to investigate further** (specific follow-up questions a founder should chase down)
5. **What's missing from this data** that would sharpen the analysis (customer industry, plan tier, channel, contract type)

Lead with the most important finding. Don't bury bad news.

### Step 10 — Present results

Use `present_files` to share:

1. `executive_summary.md`
2. The Excel workbook
3. The cohort retention HTML chart
4. The cleaned data CSV (so the user can audit)

Briefly walk the user through the top finding and offer to drill into any specific cohort, segment, or customer.

## Implementation notes

- **Always use pandas for the data manipulation.** No SQL, no spreadsheet formulas.
- **Use the `xlsx` skill** for producing the workbook — read `/mnt/skills/public/xlsx/SKILL.md` first for the formatting conventions and chart embedding.
- **For HTML charts:** use Chart.js or Plotly via standalone HTML files. Don't use matplotlib PNGs — the user can't interact with them.
- **Be defensive on dates.** Pandas can misread date columns; always check `df.dtypes` after loading and convert explicitly with `pd.to_datetime(..., errors='coerce')`.
- **Be defensive on amounts.** Currency strings (`"$1,234.56"`) need cleanup before they're numeric. Negative values in parentheses (`"(500)"`) are common in accounting exports.
- **Cache intermediate results** during a long analysis so the user can ask follow-up questions without recomputing from scratch.

## Rules of engagement

- **Do not invent data.** If something isn't in the file, say so.
- **Be skeptical of your own headline findings.** Before reporting, check whether it could be a data-window artifact, partial period, or segmentation choice. Caveat where appropriate.
- **Show your work on judgment calls.** When you classify a customer as churned or pick a threshold, explain why.
- **Default to the longer retention window.** A flattering short-window number is worse than a sobering long-window number.
- **Be direct about bad news.** If retention is degrading or growth is artificial, say so plainly.
