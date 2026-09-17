# Growth OS

Growth OS is Wellio's marketing memory: one repo holding what the market has said, the machines that act on it, the job spec of every agent, and the results those machines have produced. It exists to close one loop, what goes out to the market comes back as a measured result and changes what goes out next.

## What has run

Cycle zero processed 803 closed AU new-business deals from the Sales Pipeline AU on 2026-09-17. 704 of them had early conversation text and were tagged by a language model against hook codebook 1.0, with zero tagging errors; the other 99 had none, so the model was never called on them. Those 704 are the in-scope population; the in-scope base win rate is 38% (271 won, 433 lost). The 99 are excluded because their company has no file in the SDR-history cache, and deals outside that cache win far more often than deals inside it, so the cache is not a random sample and the finding below cannot be generalised to the excluded deals.

The weekly pull-and-tag job is installed and scheduled for Sunday 21:00. Its first real run, on 2026-09-17, pulled 503 deals, 565 meetings, 1,101 calls and 167 notes, then exited without producing a digest: the HubSpot private app does not hold the emails read scope, and the pull fails on that object.

Source: customer-truth/what-the-market-is-telling-us.md; agents/weekly-pull-and-tag.md

## The one finding

No hook in the cycle-zero data clears the promotion bar, at least 20 mentions and a volume-adjusted difference of at least 10 points across at least two volume strata. Once conversation volume is controlled for, no wording separates the deals that won from the deals that lost. The strongest signal is structural rather than verbal: deals that won closed in a median 29 days, against 73 days for deals that lost, and win rate falls as the logged conversation grows (43%, 43%, 36%, 32% by early-text quartile). One tag shows a nominal association against winning: time_saved has a volume-adjusted difference of -12.3 points on 95 deals. The promotion rule only ever promotes a hook, it does not flag a warning, so this is reported rather than promoted; and with 18 tags tested it does not survive multiple-comparison correction, so it is a lead to check, not a finding.

Source: customer-truth/what-the-market-is-telling-us.md

## Read next

1. `results/README.md`: what has been built, and the baseline numbers behind it.
2. `customer-truth/README.md`: what schools say when they buy, and the full finding.
3. `decisions/README.md`: what has been decided, and what is still open.

## The loop

Observe (weekly read-only pull), learn (tag and rank on a branch), decide (Caelum merges), act (stamped sequences and briefs), measure (scorecard), then back to observe.

Spec: `docs/superpowers/specs/2026-09-16-growth-os-design.md`.

## What is not yet measured

`results/metric-definitions.md` lists every metric this role would be measured on and whether it is measurable today. The gap that matters most is hook attribution: which hook a school responded to is not measurable, because no message has ever been stamped with a hook, persona or variant on send.

Source: results/metric-definitions.md
