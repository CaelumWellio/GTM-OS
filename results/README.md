# Results

What the work found, and what it changed.

## What we found

Nothing we say predicts winning: across 842 closed AU deals no angle separates winners from losers once each deal is compared only against deals that had a similar amount of conversation recorded against them, and the closest thing to a positive signal, the social media ban, appeared in 19 deals where 20 were needed to count.

How fast a school replies predicts a great deal: a reply inside three days goes with a 46.9 percent win rate on 559 deals and fifteen days or more with 16.3 percent on 49, but silence is a warning sign only for schools we contacted cold, since the 83 no-reply deals that arrived through Free Preview, Contact Sales, a conference or a diocese network won 51.8 percent against 36.9 percent for the 84 that came from ordinary outbound.

Our outbound sequences vary enormously and the spread is not proof: the largest by volume made 1,030 sends to 134 schools for 3 meetings and 1 win while email written by hand to 244 schools produced 59 meetings and 13 wins, and two sequences with different lists, months and targeting are never a fair comparison.

Source: `customer-truth/what-the-market-is-telling-us.md`; `customer-truth/response-speed.md`; `outbound-engine/sequence-inventory.md`.

## What changed as a result

There is now a recommendation on the table rather than an analysis: `outbound-engine/term-4-plan.md` commits to 56 qualified meetings sourced from email in the ninety days to mid-December, against the 51 the same window produced last year, and 5 of those 56 rest on one move whose effect is assumed, not measured, and is gated on a check before any volume moves. The total board is context rather than commitment: the equivalent window last year, 18 September to 16 December 2025, produced 315 booked AU meetings of which 156 qualified, and 2026 is running at about 44 qualified a month, roughly 131 in ninety days. Two thirds of that total is booked on the phone by the SDR team, which the plan does not touch, which is why the commitment is the email-sourced half.

Four things are now measured that were not. What Term 4 actually produces, counted off the lead records rather than modelled from send volume, and split by whether the meeting was booked on a call or by email. How quickly a school answers, set against whether the deal was won, and split by how the school reached us. What every AU outbound sequence has actually produced, in one table, sends against meetings and wins. And the weekly read of HubSpot, which now runs end to end on a schedule, so the numbers the plan reports every Monday come out of a job rather than a hand pull.

One thing was deliberately not changed. No angle was promoted into the messaging, because none cleared the evidence bar, and reporting a near miss as a finding would have been the easiest and worst mistake available here.

Source: `outbound-engine/term-4-plan.md`; `customer-truth/response-speed.md`; `outbound-engine/sequence-inventory.md`; `agents/weekly-pull-and-tag.md`; the Term 4 and 2026 meeting counts computed from `~/Documents/GTM Project/outputs/sdr_funnel/leads.jsonl` joined to company market in `lead_company_context.json`.

## The numbers behind it

`baseline-2026-09.md` holds the numbers already recorded, sourced and dated. Four headline figures, all recorded deal amounts in HubSpot production 20058914, not recognised ARR: AU new business 1 January to 3 September 2025, 80 deals won, $693,805; the same window in 2026, 85 deals won, $628,515; full year 2025, 284 deals won, $2,348,300; and of that full year, 204 deals, 70 percent of them, won on or after 4 September, worth $1,654,495. The last of those is why the plan is a ninety-day plan.

The plan's own baseline sits alongside those and is counted off the lead records, not off deals: in the 18 September to 16 December 2025 window, 315 AU meetings booked and 156 qualified, a 49.5 percent qualification rate, of which 105 were booked on a call and 51 by email. Across February to August 2026, 305 qualified AU meetings, about 44 a month, of which 61 came from email.

Source: `results/baseline-2026-09.md`; the Term 4 and 2026 meeting counts computed from `~/Documents/GTM Project/outputs/sdr_funnel/leads.jsonl` joined to company market in `lead_company_context.json`, AU only, using `date_first_entered_meeting_booked` and `hs_v2_date_entered_qualified_stage_id_233247981`.

## What is measured, and what is not

`metric-definitions.md` lists every metric this role would be measured on, its definition, where it lives in HubSpot, and whether it can be read today. Meetings booked, pipeline created and pipeline won can be read. Meetings held cannot, because 70 percent of past meetings still read as scheduled. The gap that matters most is attribution: which angle a school responded to cannot be measured at all, because no message has ever been recorded with its angle at the moment it was sent.

Source: `results/metric-definitions.md`.

## What has been built

- The Five Engines strategy: the September 2026 GTM strategy artifact, both critique rounds merged.
- The Sales Signals pilot and SDR outreach skill: 20 schools gated on prospect status, the first email per school written from a pinned note and sent by a human.
- The primary attach spine and scraping pipeline: 202 K-12 schools, 201 scraped, a junior-school point of contact named at 99.
- The customer-truth machine: one full backward-looking run over the closed AU deals, each one tagged against a fixed list of angles with zero tagging errors, plus the weekly job that repeats it.

Source: `~/Documents/GTM Project/docs/gtm-strategy-2026-09/10-third-critique-response.md`; `~/Documents/GTM Project/docs/revops/2026-09-05-sdr-outreach-pilot-assessment.md`; `~/Documents/GTM Project/outputs/primary_attach_2026-09-05/`; `customer-truth/what-the-market-is-telling-us.md`.
