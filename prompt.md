# Customer Analysis Prompt

> Copy everything below the line and paste it into a fresh Claude conversation, along with your transaction CSV. Claude will run the analysis and produce a report.

---

You are an expert revenue analyst. I'm going to give you a customer transaction file. Your job is to produce a complete customer analysis covering cohort retention, customer segmentation, revenue concentration, and trend analysis.

## Step 1 — Understand the data before computing anything

Before you produce any numbers, examine the file and tell me:

1. **Shape:** is it long format (one row per customer-period with columns like `customer_id`, `period`, `amount`) or wide format (customers as rows, periods as columns)?
2. **Period grain:** monthly, weekly, daily, quarterly?
3. **Time range:** earliest period, latest period, total number of periods
4. **Customer count:** how many unique customers
5. **Total revenue:** sum across all transactions
6. **Data quality flags:** any of the following present?
   - Negative amounts (refunds, chargebacks)
   - Zero-amount rows
   - Customers appearing under multiple slightly different names (likely the same entity)
   - Periods with no transactions at all (gaps in the timeline)
   - Very recent periods that might be partial (current month not yet complete)

Pause here. Show me what you found. If anything looks ambiguous, ask me one clarifying question before proceeding. **Do not start the analysis until I confirm the data interpretation is correct.**

## Step 2 — Confirm the analysis scope

Once the data is understood, propose:

- Which periods to **include** in the analysis (recommend excluding partial current periods, and flag if early periods have suspiciously low activity that suggests the data window predates the real start of the business)
- How to handle **refunds and adjustments** (net out by default; ask if the user wants gross figures shown separately)
- How to handle **duplicate-looking customer names** (propose merges; let me confirm before merging)
- What **retention threshold** defines an "active" customer in a period (default: any positive transaction in the period; flag if user might want a minimum revenue threshold for noisier datasets)

Get my confirmation on these choices. Default to sensible answers and only ask when there's a real judgment call.

## Step 3 — Run the analysis

Produce all four analyses below. Show the computed tables AND interpret what they mean. Do not just dump numbers — for each section, lead with a one-sentence headline finding, then show the supporting data.

### 3.1 — Revenue overview

- Total revenue by period (table + simple chart if you can render one)
- Period-over-period growth rate
- Customer count by period (new, retained, churned, total active)
- Trend direction and any inflection points worth flagging

### 3.2 — Customer segmentation

Classify every customer into one of these segments based on their behavior over the analysis window:

- **Whales** — top 10% by total revenue
- **Regulars** — middle 40% by total revenue, with consistent activity
- **Small accounts** — bottom 40% by total revenue, with consistent activity
- **Churned** — previously active but no activity in the last 60-90 days (use the longer window if data is sparse)
- **One-time** — single period of activity, no return

Report:

- Customer count and revenue share for each segment
- Revenue concentration: what share of total revenue comes from the top 10%, top 20%, top 50% of customers
- Whether concentration is **increasing or decreasing** over time (split the analysis window in half and compare)
- Name the top 10 customers by revenue (so I can sanity-check the results)

### 3.3 — Cohort retention

For every cohort (customers grouped by the period of their first transaction), compute:

- **N+1 retention** (still active one period later)
- **N+3 retention** (still active three periods later)
- **N+6 retention** (still active six periods later)
- **N+12 retention** (if data window allows)

Critical: only compute a retention milestone for cohorts that have had time to reach it. A cohort that started 2 months ago cannot have a N+6 figure. Show "n/a" or "insufficient window" rather than computing a misleading number.

Also produce an **apples-to-apples view**: restricted to cohorts that have had time to reach all milestones, so cohort-to-cohort comparison is fair.

**Flag explicitly** if newer cohorts are retaining worse than older ones — this is the single most important finding the analysis can produce, and it must be called out clearly if present.

### 3.4 — Per-customer trajectory

For customers who have been active for at least 6 periods:

- Are they spending more, the same, or less than 6 periods ago?
- Identify any individual customers with notable trajectories (sharp expansion, sharp contraction, sudden churn after high engagement, reactivation after a gap)
- Aggregate: what share of long-tenured customers are growing vs. shrinking?

### 3.5 — Anomalies and flags

Surface any of the following you notice:

- One-time large transactions that distort headline revenue
- Customers with unusual patterns (large reactivation, suspicious bursts)
- Periods with anomalous spikes or drops
- Concentration risks (e.g., a single customer accounting for >20% of revenue)
- Cohort behavior that contradicts the narrative the top-line numbers suggest

## Step 4 — Executive summary

End with a plain-English summary, no more than 6 bullet points, covering:

1. Is the business growing? At what rate?
2. Is growth healthy or are there underlying concerns?
3. What is the single most important pattern in the data
4. Three things that would be useful to investigate further (with specific follow-up questions)
5. What's missing from this data that would sharpen the analysis (e.g., customer industry, plan tier, acquisition channel, contract type)

## Step 5 — Offer follow-ups

After delivering the analysis, offer to:

- Drill into any specific cohort, segment, or customer
- Re-run with different definitions (different churn threshold, different segment cutoffs)
- Produce a one-page version suitable for a board deck
- Export the underlying tables as a spreadsheet

---

## Rules of engagement

- **Do not invent data.** If something isn't in the file, say so. Never fabricate customer names or numbers to make the output look fuller.
- **Be skeptical of your own headline findings.** Before reporting a conclusion, ask yourself whether it could be an artifact of the data window, partial periods, or segmentation choices. Caveat where appropriate.
- **Show your work on judgment calls.** When you classify a customer as "churned" or pick a retention threshold, briefly explain why so I can override if needed.
- **Default to the longer window when computing retention.** A flattering short-window number is worse than a sobering long-window number, because it sets up disappointment downstream.
- **Be direct about bad news.** If retention is degrading, revenue concentration is risky, or growth is artificial, say so plainly. The point of the analysis is to find these things — softening them defeats the purpose.

Ready when you are. Upload the file and confirm you'd like me to begin.
