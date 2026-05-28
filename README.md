# Customer Analysis Prompt

Drop in a transaction file. Get back cohort retention curves, customer segmentation (whales / regulars / churned), revenue concentration, and trend analysis.

No SQL. No data team. No BI tool. An LLM and about twelve minutes.

This repo contains:

- **`prompt.md`** — the analysis prompt. Paste into Claude (or any capable LLM) along with your CSV.
- **`SKILL.md`** — a Claude Code skill for users who want to run this as a reusable, version-controlled workflow.
- **`sample-data.csv`** — anonymized example data so you can see what the output looks like before running it on your own numbers.

## Quick start

### Option 1 — Claude.ai (easiest)

1. Open [claude.ai](https://claude.ai)
2. Upload your transaction CSV (see [Input format](#input-format) below)
3. Paste the contents of `prompt.md`
4. Read the output. Ask follow-up questions.

### Option 2 — Claude Code (for repeat use)

1. Clone this repo
2. Drop your transaction CSV into the working directory
3. Ask Claude Code to "run the customer analysis skill on my data"
4. The skill follows `SKILL.md` and produces a spreadsheet, charts, and an executive summary

## Input format

The simplest version of your transaction data:

| customer_id | period | amount |
|---|---|---|
| acme_corp | 2025-01 | 4500 |
| acme_corp | 2025-02 | 4500 |
| beta_co | 2025-01 | 1200 |
| ... | ... | ... |

Three columns. One row per customer per period. That's it.

If your data is in a wide format (customers as rows, dates as columns, transaction sizes as cells), the prompt handles that too — just tell it which shape you're working with.

**What "period" means:** monthly is the default and works for most businesses. Weekly works if you have high enough transaction volume. Daily is usually too noisy unless you're a consumer app.

**What "amount" means:** net revenue is the cleanest input. Gross revenue minus refunds, chargebacks, discounts, and adjustments. If you only have gross, the prompt will tell you what assumptions it's making.

## What you get back

1. **Cohort retention curves.** For every group of customers who started in the same period, how many are still paying you 30/60/90/180 days later, and how much.
2. **Customer segmentation.** Whales (top decile by revenue), regulars (the middle), and churned (formerly active, now silent). Revenue share for each segment.
3. **Revenue concentration.** What percentage of revenue comes from your top 10%, 20%, 50% of customers. Whether that concentration is increasing or decreasing over time.
4. **Trend analysis.** Is average revenue per customer growing or shrinking? Are new cohorts retaining better or worse than old ones? Where is the curve bending?
5. **Anomaly flags.** Customers with unusual patterns (sudden drop-off, large reactivation, one-time spikes) called out so you can investigate.

## Will this work for my business?

**Honest answer: not for everyone.** This analysis assumes a specific shape of business. If yours doesn't match, the output will be misleading at best and useless at worst.

### This works well for:

- **B2B SaaS** with monthly or annual recurring billing
- **Subscription consumer products** (Netflix-shaped — recurring charges)
- **Usage-based platforms** that bill customers repeatedly (Twilio-shaped, AWS-shaped)
- **Professional services** with named clients who engage on repeated projects (agencies, consultancies, law firms)
- **B2B marketplaces** where you can track which buyer paid which seller over time
- **D2C with strong repeat purchase patterns** (subscription boxes, beauty, pet, supplements — anywhere customers come back monthly or quarterly)
- **Medical / dental / health practices** billing the same patients or referring providers across visits

The common thread: **identifiable customers, repeat transactions, multiple periods of history.**

### This does NOT work well for:

- **One-time-purchase businesses** with no expected repeat (engagement rings, mattresses, funeral services). Cohort retention is meaningless — every customer "churns" by design.
- **Anonymous-transaction retail** (a coffee shop, a gas station, most physical retail). If you can't link the same customer across transactions, you can't compute retention or segmentation.
- **Pure ad-supported media** where users aren't paying you. The relevant analysis is engagement-based, not revenue-based — different tool needed.
- **Project-based businesses with single large contracts** (construction, M&A advisory, custom enterprise installations). Too few data points per customer; cohort math doesn't stabilize.
- **Pre-revenue companies.** Nothing to analyze yet. Come back when you have transactions.
- **Businesses with fewer than ~30 customers or ~6 months of history.** The math works but the conclusions won't be statistically meaningful. You're better off reading customer interviews at this stage.

### Edge cases — proceed with caution:

- **Marketplaces where the "customer" is ambiguous** (is it the buyer? the seller? both?). The prompt can run on either side, but you have to pick one and know what you're measuring.
- **Businesses with heavy seasonality** (tax software, holiday retail). Standard retention windows will misread seasonal patterns as churn. Tell the prompt about the seasonality up front.
- **Businesses that switched pricing models mid-history.** Cohorts before and after the switch aren't comparable. Either restrict the window or analyze them separately.
- **Heavily contracted businesses** where annual prepays make monthly activity look bursty. The prompt can normalize, but you need to flag this so it doesn't read normal billing cadence as churn.

If you're not sure whether your business fits, run it on `sample-data.csv` first to see what the output looks like, then think about whether the same questions make sense for your business.

## Limitations and caveats

- **The LLM can be wrong.** Spot-check the numbers against a known truth (last quarter's revenue, a customer count you trust) before sharing the output with anyone whose opinion matters.
- **This isn't a substitute for a real BI setup at scale.** If you're past Series B, hire an analyst. This tool is for the stage where hiring an analyst is overkill but flying blind is worse.
- **Interpretation is the hard part.** The prompt produces numbers. Deciding what those numbers mean for your business is still your job. The blog post linked below covers the most common interpretation traps.
- **Privacy:** the prompt runs against whatever LLM you point it at. If your customer data is sensitive, use a deployment that meets your data handling requirements (Claude for Work, an API call with zero-retention, or a self-hosted model). Don't paste PHI or PII into a free consumer chat.

## See also

- [Blog post: "Your spreadsheet already knows whether your business is working"](https://andrewlochemes1.substack.com/p/your-spreadsheet-already-knows-whether) — the why-this-matters writeup, including the most common ways founders misread their own metrics
- [Sample output](./sample-output.md) — what the analysis looks like when run on `sample-data.csv`

## Contributing

If you run this on your own data and find a case where the prompt produced something misleading, open an issue. The prompt gets sharper every time someone breaks it.

## License

MIT. Use it, fork it, ship it.
