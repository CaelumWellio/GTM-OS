# Growth OS

Growth OS is Wellio's marketing memory: one repo holding what the market has said, the machines that act on it, the job spec of every agent, and the results those machines have produced. It exists to close one loop, what goes out to the market comes back as a measured result and changes what goes out next.

## What has run

Cycle one processed 844 closed AU new-business deals from the Sales Pipeline AU, rerun on 2026-09-17 after the HubSpot private app was granted the sales-email-read and crm.objects.contacts.read scopes and the 95 company timelines missing from the SDR-history cache were backfilled read-only. 842 of them are in scope, 99.8% coverage, up from 87.7% in cycle zero; the in-scope base win rate is 43% (365 won, 477 lost). Only 2 deals are still excluded, because their company has no file in the SDR-history cache.

The weekly pull-and-tag job is installed, scheduled for Sunday 21:00, and now runs end to end. Its first digest, 2026-W38, is on branch `auto/weekly` for review.

Source: customer-truth/what-the-market-is-telling-us.md; agents/weekly-pull-and-tag.md

## The one finding

No hook in the cycle-one data clears the promotion bar, at least 20 mentions and a volume-adjusted difference of at least 10 points across at least two volume strata. Once conversation volume is controlled for, no wording separates the deals that won from the deals that lost. The strongest signal is structural rather than verbal: deals that won closed in a median 27 days, against 73 days for deals that lost, and win rate falls as the logged conversation grows (48%, 48%, 40%, 37% by early-text quartile). The most negative well-powered tag is time_saved, with a volume-adjusted difference of -13.0 points on 108 deals. The promotion rule only ever promotes a hook, it does not flag a warning, so this is reported rather than promoted; and with 18 tags tested it does not survive multiple-comparison correction, so it is a lead to check, not a finding. The nearest thing to a positive signal is social_media_ban, at n=19, one deal short of the minimum sample; it lines up with the Five Engines strategy's existing bet on the social media ban as a hook, and is worth testing deliberately.

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
