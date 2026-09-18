# What each event returned

Generated 2026-09-18T06:24:21 UTC. 12 of 151 events have a matched campaign, together 182 deals and 74 wins worth $525,439.

## What this can and cannot see

Every figure comes from the Australian sales pipeline, which is the only one cached here, so UK and international events appear in the catalogue and cannot be measured at all. A campaign is an assignment made when the deal was created, not proof the event caused it. Costs are the planned 2026 figures from the workbook while returns span 2025 and 2026, and amounts are recorded deal values rather than recognised revenue.

Source: `events/data/events.csv` and `events/data/campaign-aliases.csv`; joined to ~/Documents/GTM Project/outputs/sdr_funnel/deals.jsonl and leads.jsonl; workbook snapshot in outputs/growth_os/

## The returns

| Event | Cost | Deals | Won | Revenue | Cost per win | Return | Note |
|---|---|---|---|---|---|---|---|
| NSWSPC Conference | 12,000 | 75 | 30 | 182,468 | 400 | 15.21x |  |
| WASSEA | 9,350 | 20 | 13 | 119,640 | 719 | 12.8x |  |
| ASA | 17,600 | 18 | 7 | 66,745 | 2,514 | 3.79x |  |
| QSPA | 7,150 | 7 | 5 | 46,889 | 1,430 | 6.56x |  |
| CaSPA National Conference | 16,500 | 12 | 4 | 37,724 | 4,125 | 2.29x |  |
| HICES |  | 3 | 1 | 19,656 |  |  |  |
| NSWSDPA | 25,000 | 6 | 2 | 19,271 | 12,500 | 0.77x |  |
| EARCOS Annual Leadership Conference | 1,300 | 6 | 4 | 13,816 | 325 | 10.63x |  |
| VASSP 2026 |  | 6 | 2 | 11,140 |  |  |  |
| QASSP School Leadership Conference | 0 | 12 | 5 | 7,089 |  |  |  |
| SAWLIS | 1,000 | 3 | 1 | 1,000 | 1,000 | 1.0x |  |
| PESA |  | 14 | 0 | 0 |  |  |  |

Source: `events/data/events.csv` and `events/data/campaign-aliases.csv`; joined to ~/Documents/GTM Project/outputs/sdr_funnel/deals.jsonl and leads.jsonl; workbook snapshot in outputs/growth_os/

## Where the sheet and HubSpot disagree

| Event | Sheet claims | HubSpot wins | Difference |
|---|---|---|---|
| NSWSPC Conference | 29 | 30 | -1 |
| WASSEA | 15 | 13 | +2 |
| ASA | 10 | 7 | +3 |
| QSPA | 6 | 5 | +1 |
| HICES | 2 | 1 | +1 |
| NSWSDPA | 5 | 2 | +3 |
| CaSPAQ | 6 | 0 | +6 |
| PESA | 8 | 0 | +8 |
| VCSSDPA | 1 | 0 | +1 |

Source: `events/data/events.csv` and `events/data/campaign-aliases.csv`; joined to ~/Documents/GTM Project/outputs/sdr_funnel/deals.jsonl and leads.jsonl; workbook snapshot in outputs/growth_os/

## Exceptions to resolve

| Event | Issue | Detail |
|---|---|---|
| ASA | sheet and HubSpot disagree | sheet 10, HubSpot 7 won |
| CaSPAQ | claims deals but has no campaign alias | 6 claimed |
| HICES | sheet and HubSpot disagree | sheet 2, HubSpot 1 won |
| NSWSDPA | sheet and HubSpot disagree | sheet 5, HubSpot 2 won |
| NSWSPC Conference | sheet and HubSpot disagree | sheet 29, HubSpot 30 won |
| PESA | sheet and HubSpot disagree | sheet 8, HubSpot 0 won |
| QSPA | sheet and HubSpot disagree | sheet 6, HubSpot 5 won |
| VCSSDPA | claims deals but has no campaign alias | 1 claimed |
| WASSEA | sheet and HubSpot disagree | sheet 15, HubSpot 13 won |
| (none) | campaign looks like an event but has no alias | AU 2026 - APPA Cold: 1 deals |
| (none) | campaign looks like an event but has no alias | AU 2026 - APPA: 1 deals |

Each of these is a naming problem rather than a measurement problem, and each is fixed by a row in `events/data/campaign-aliases.csv` or by a consistent campaign name at the point the campaign is created.

Source: `events/data/events.csv` and `events/data/campaign-aliases.csv`; joined to ~/Documents/GTM Project/outputs/sdr_funnel/deals.jsonl and leads.jsonl; workbook snapshot in outputs/growth_os/
