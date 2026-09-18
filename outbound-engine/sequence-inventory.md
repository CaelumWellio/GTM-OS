# What the AU outbound sequences did

Generated 2026-09-18T00:31:41 UTC. 225 sequences, 12459 sequenced sends of 35055 outbound emails (35.5%), all within the AU scope described below.

## What this covers, and what it does not

Only 35.5 percent of outbound emails carry a sequence id. The rest are written by hand, so this describes the sequenced part of outbound and nothing else. A school touched by several sequences is counted under each of them, so the columns add up to more than reality. A reply counts when it arrives within 14 days of the first send to that school, a meeting within 30 days, and a deal within 60 days. This is touch attribution, not proof that the sequence caused anything.

This inventory is scoped to AU. 1568 companies have a market of au, 936 have a different known market (chiefly UK, where outbound runs through Outreach, not HubSpot sequences), and 89 are known companies with no market recorded. Email rows outside the AU set were dropped before anything below was computed: 25691 had no company association at all, 20475 belonged to a company with a different market, and 600 belonged to a known company with no market recorded (49 distinct companies).

The largest exclusion is a different kind of gap, not just another missing field: 74757 email rows, across 5986 distinct companies, went to a school that does not appear in the company file at all. That file contains exactly the companies that have ever been associated with a HubSpot Lead object, so a school that was emailed but never became a lead is invisible to this whole inventory: it cannot be scoped into AU, cannot enter the control group, and is counted nowhere below. In plain terms, this file only ever sees schools that became leads. Every reply rate, meeting rate and win rate reported here is therefore conditioned on that, and overstates true cold-outbound performance: the real volume of cold outbound that never turned into a lead, and so never progressed, is far larger than any number in this file suggests.

Sequence names did not resolve on this run, so sequences below are identified only by their HubSpot id; look that id up inside HubSpot to find the sequence's name.

Source: ~/Documents/GTM Project/outputs/growth_os/sequence_inventory.csv and sequence_inventory.json; computed from outputs/sdr_funnel/emails.jsonl, leads.jsonl and deals.jsonl

## The twenty largest sequences

| Sequence | Sends | Schools | Reply rate (14d) | Reply rate (60d) | Meetings | Deals won | Note |
|---|---|---|---|---|---|---|---|
| 299760938 | 1030 | 134 | 6.7% | 23.1% | 3 | 1 |  |
| 289426598 | 1007 | 110 | 27.3% | 63.6% | 29 | 12 |  |
| 291806797 | 816 | 55 | 34.5% | 56.4% | 4 | 3 |  |
| 290985046 | 459 | 67 | 32.8% | 55.2% | 10 | 11 |  |
| 285582734 | 354 | 44 | 54.5% | 65.9% | 9 | 5 |  |
| 309028247 | 352 | 34 | 20.6% | 29.4% | 3 | 0 |  |
| 307913077 | 345 | 109 | 22.0% | 32.1% | 18 | 1 |  |
| 304709532 | 324 | 112 | 21.4% | 37.5% | 21 | 0 |  |
| 300679565 | 278 | 150 | 46.0% | 80.7% | 0 | 0 |  |
| 298986013 | 243 | 97 | 29.9% | 46.4% | 24 | 2 |  |
| 291376944 | 223 | 71 | 45.1% | 66.2% | 38 | 7 |  |
| 306127916 | 207 | 48 | 8.3% | 12.5% | 0 | 0 |  |
| 306976000 | 200 | 71 | 22.5% | 31.0% | 2 | 0 |  |
| 273996434 | 184 | 80 | 56.2% | 75.0% | 40 | 9 |  |
| 281340465 | 181 | 44 | 27.3% | 47.7% | 10 | 3 |  |
| 290240094 | 144 | 48 | 35.4% | 64.6% | 21 | 6 |  |
| 274749313 | 136 | 57 | 38.6% | 59.7% | 11 | 7 |  |
| 284064136 | 125 | 32 | 50.0% | 71.9% | 16 | 6 |  |
| 289256817 | 122 | 22 | 13.6% | 36.4% | 4 | 0 |  |
| 283130368 | 114 | 39 | 35.9% | 69.2% | 17 | 6 |  |
| everything else, 205 sequences | 5615 | 2258 | 35.3% | 57.8% | 295 | 108 | |

Source: ~/Documents/GTM Project/outputs/growth_os/sequence_inventory.csv and sequence_inventory.json; computed from outputs/sdr_funnel/emails.jsonl, leads.jsonl and deals.jsonl

## Against the schools we emailed without a sequence

| Group | Schools | Reply rate (14d) | Reply rate (60d) | Meetings | Deals won |
|---|---|---|---|---|---|
| no sequence, written by hand | 244 | 41.4% | 62.7% | 59 | 13 |

Source: ~/Documents/GTM Project/outputs/growth_os/sequence_inventory.csv and sequence_inventory.json; computed from outputs/sdr_funnel/emails.jsonl, leads.jsonl and deals.jsonl

## Limits

- A sequence touching fewer than 20 schools is marked thin and carries no conclusion.
- Performance is reported by sequence, never by the person who sent it.
- Replies by phone or in person are invisible here.
- The windows above are choices, not facts; a different window would move the numbers.
- This inventory only sees schools that became leads; every rate in this file is an overstatement of true cold-outbound performance, not an estimate of it.
- Two sequences with similar send volumes are not a controlled comparison: their lists, time periods and targeting can all differ. A difference between two sequences is a question to investigate, not a conclusion to report.
- Reply rate depends heavily on the window chosen: sequence 299760938 replies at 6.7% within 14 days but 23.1% within 60 days, the same underlying behaviour measured differently. A slow-cadence sequence can look ten times worse than it is if the window is too short.
- 34.9% of incoming emails and 7.1% of outgoing emails in the underlying cache carry no company association at all and are invisible to every reply rate here; every reply rate in this file is a floor, not a true rate.

Source: ~/Documents/GTM Project/outputs/growth_os/sequence_inventory.csv and sequence_inventory.json; computed from outputs/sdr_funnel/emails.jsonl, leads.jsonl and deals.jsonl
