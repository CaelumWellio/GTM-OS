# How fast a school replies

Generated 2026-09-17T23:54:30 UTC. 842 closed deals, base win rate 43.4%.

## The question

Deals that won closed in a median 27 days against 73 for deals that lost. That gap is mostly an effect, because a deal that is going nowhere sits open until somebody closes it. The useful question is whether anything visible early, while the deal is still live, predicts the outcome.

Source: ~/Documents/GTM Project/outputs/growth_os/response_speed.json; computed from outputs/customer_truth/deals_early_text.jsonl and outputs/sdr_history/; method in customer-truth/method.md

## How fast the school replies

| Days from deal creation to the school's first reply | Win rate | Deals | Note |
|---|---|---|---|
| 0 to 3 | 46.9% | 559 |  |
| 4 to 14 | 31.3% | 67 |  |
| 15 or more | 16.3% | 49 |  |
| no reply in the early window | 44.3% | 167 |  |

Source: ~/Documents/GTM Project/outputs/growth_os/response_speed.json; computed from outputs/customer_truth/deals_early_text.jsonl and outputs/sdr_history/; method in customer-truth/method.md

## How hard we pushed

| Wellio emails sent in the early window | Win rate | Deals | Note |
|---|---|---|---|
| 0-2 | 46.5% | 99 |  |
| 3-5 | 48.4% | 223 |  |
| 6-10 | 43.9% | 244 |  |
| 11+ | 37.7% | 276 |  |

Source: ~/Documents/GTM Project/outputs/growth_os/response_speed.json; computed from outputs/customer_truth/deals_early_text.jsonl and outputs/sdr_history/; method in customer-truth/method.md

## The group that does not fit

167 deals had no school reply at all in the early window, and they win 44.3%, which is higher than the deals that replied slowly. That has to be explained before any rule is drawn, because a rule that deprioritises silence would be wrong for this group.

| Source of the deal | Win rate | Deals |
|---|---|---|
| OFFLINE | 44.8% | 163 |
| DIRECT_TRAFFIC | 50.0% | 2 |
| Unknown | 0.0% | 1 |
| ORGANIC_SEARCH | 0.0% | 1 |

Deal source does not explain it: 97.6% of the no-reply group and 97.0% of the group that replied both came in as OFFLINE, so that field carries no signal here. Deal type works against the anomaly rather than for it, since the no-reply group leans slightly more toward New Business - Primary, and Primary deals win less often, not more.
Campaign is where the explanation lives. 83 of the 167 no-reply deals (49.7%) sit in campaigns a school can join without ever sending an early email: inbound Free Preview and Contact Sales requests, principal-association conferences, and Catholic feeder-school or diocese-and-order networks such as Edmund Rice. Those 83 deals win 51.8%. The other 84 no-reply deals, the ones that came from ordinary outbound and ABM campaigns and still never answered, win only 36.9%, well below the base rate and closer to the slow-reply groups above than to deals that answered quickly. So the rule below is conditional on how the deal arrived, not a blanket read on silence.

Source: ~/Documents/GTM Project/outputs/growth_os/response_speed.json; computed from outputs/customer_truth/deals_early_text.jsonl and outputs/sdr_history/; method in customer-truth/method.md

## The rule

If a deal came from ordinary cold outreach or an ABM campaign, and the school sends no reply at all in the early weeks, treat that the same as a slow, dragging conversation: follow up again, flag it to a manager, and do not assume it just needs more time. Do not apply that warning to a deal that came in through a Free Preview or Contact Sales request, a principal conference, or a Catholic feeder-school or diocese connection, since schools on those paths regularly buy without ever emailing back early, and silence there is normal, not a warning sign.

Source: ~/Documents/GTM Project/outputs/growth_os/response_speed.json; computed from outputs/customer_truth/deals_early_text.jsonl and outputs/sdr_history/; method in customer-truth/method.md

## Limits

- Cells below 20 deals are marked thin and carry no conclusion.
- Replies are counted from cached company timelines, so a reply by phone or in person is invisible here.
- Auto-replies are excluded, and the early window ends when the deal first reached Evaluating.
- This is an association measured on closed deals, not a controlled test.

Source: ~/Documents/GTM Project/outputs/growth_os/response_speed.json; computed from outputs/customer_truth/deals_early_text.jsonl and outputs/sdr_history/; method in customer-truth/method.md
