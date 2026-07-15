# loyalty-rewards-causal-analysis
# Does Loyalty Reward Engagement Actually Drive Spend? A Causal Analysis

## Overview

Retail loyalty programs (Woolworths Rewards, Flybuys, and similar point-based systems) assume that giving customers a reward for reaching a spend threshold increases their future spending. This project tests that assumption properly, using a public dataset with the same reward-threshold mechanic (Starbucks Rewards app data), since real transaction-level data from Australian grocery loyalty programs is not publicly available. The same method applies directly to programs like Flybuys or Woolworths Rewards.

The goal was not just to measure whether reward completers spend more, but to find out whether the reward *causes* that spend, or whether the naive numbers are misleading.

## The naive finding (and the flaw in it)

A simple comparison showed customers who completed a reward offer spent roughly 6x more on average than those who didn't ($111.56 vs $18.62 per customer).

Digging into how "offer completed" is actually defined revealed a problem: some offers are marked complete simply because a customer's spend crossed a threshold, whether or not they ever saw the offer. About 15% of all completions happened without the customer viewing the offer first, meaning the causality ran backwards for a meaningful chunk of the data. Naturally high-spending customers were getting credit for "responding to" an offer they never saw.

## Correcting for the bias

Splitting customers into three groups based on actual offer exposure showed a clearer picture:

| Group | Avg. spend |
|---|---|
| No real completion | $18.62 |
| Blind completion (never viewed) | $60.84 |
| Viewed, then completed | $114.72 |

Blind completers spending over 3x the baseline confirmed part of the original 6x gap was reverse causality, not a marketing effect. But the gap between blind and viewed completers suggested a real, if smaller, incremental effect.

A time-normalized before/after comparison (spend per day, before vs. after each customer's first offer view, controlling for uneven observation windows) supported this: spend per day roughly tripled after first offer exposure ($1.32 → $4.30/day).

## Formal causal estimate (DoWhy)

To move from a defensible association to a formal causal estimate, I built a causal graph (treatment: viewed-then-completed an offer; outcome: post-exposure spend per day; confounders: age, income, membership tenure) and used DoWhy's backdoor adjustment method.

**Result: an estimated $3.53 in additional daily spend attributable to offer engagement**, after controlling for age, income, and membership tenure.

This estimate was tested with two refutation checks:
- **Random common cause test:** adding a meaningless random variable left the estimate virtually unchanged ($3.53 → $3.53, p = 0.98) — the result is stable, not fragile.
- **Placebo treatment test:** replacing the real treatment with a randomly assigned fake one collapsed the effect to ~0 (−0.002) — confirming the original effect isn't a modeling artifact.

## Business impact

Total rewards paid out across the dataset: **$132,093**
Estimated extra revenue generated: **$905,064**
**Return: approximately $6.85 in extra revenue for every $1 spent on rewards.**

## Honest limitations

- This is an observational study, not a randomized experiment. A true controlled test (randomly withholding offers from a control group) would be needed to confirm the exact effect size.
- Confounders were limited to age, income, and membership tenure — unmeasured factors (e.g., general app engagement) could still play a role.
- The dataset is a public analogue (Starbucks), not actual Woolworths/Flybuys data, though the reward-threshold mechanic is directly comparable.

## Methods used

Python, pandas, DoWhy (causal graphs, backdoor adjustment, refutation testing), chi-square testing, time-normalized cohort comparison.

## Files

- `analysis.ipynb` — full analysis notebook
- `README.md` — this write-up
