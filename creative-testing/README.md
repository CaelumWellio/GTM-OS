# Creative testing

An experiment is any controlled comparison: subject lines, hooks to one persona, follow-up timing, page variants. Zero spend required.

## Minimum sample rule

A hook is promoted or demoted in `customer-truth/what-the-market-is-telling-us.md` only when it has been present in at least 20 closed deals and the win-rate difference against absence is at least 10 percentage points. Below that the cell is marked thin and nothing changes.

Why: the 2026-08-23 quote-harvest run showed a 53 percent lift on 30 schools regress to one point at 705. Source: `~/Documents/GTM Project/docs/revops/2026-08-23-quote-harvest-plan.md`.

Ledger: `ledger.csv` with columns hypothesis, variant_a, variant_b, metric, n_needed, n_reached, result, decision, date.
