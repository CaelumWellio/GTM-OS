# The 90-day Term 4 plan

## The number

I commit to **56 qualified meetings sourced from email in the 90 days to mid-December, against the 51 the same window produced last year.**

Measured off the lead records, not modelled from send volume: in the equivalent window, 18 September to 16 December 2025, AU schools booked 315 meetings and 156 qualified, 49.5 percent. Of the 156, 105 came by call and 51 by email. 2026 is running at 44 qualified a month, near 131 per 90 days. Calls are two thirds of how meetings get booked, they are the SDR team's work, and nothing in this plan changes them, so the total is not mine to commit. The email half is the only part the three moves touch.

The arithmetic:

- **The base is 51**, email's share of the 156, counted off the lead records one lead at a time.
- **Move one is worth 5, an assumption, not a measurement.** If the check under move one fails, it is worth nothing and the commitment falls back to 51.
- **Moves two and three add nothing here.** Move two acts on deals after a meeting is already booked, so it cannot add meetings; its value is in wins, also an assumption. Move three is the recording change that lets the next 90 days end in an answer.

51 plus 5 is 56, a single number and not a range. The earlier 52 to 77 was widened only by my own double counting turning out smaller than I had assumed, and a bookkeeping correction is not an upside.

One measured number cuts against that base, and you should see it now rather than in January. Across February to August 2026, email sourced 61 of 305 qualified AU meetings, about 9 a month, about 26 per 90 days. Email's share of qualified meetings has fallen from 33 percent last Term 4 to 20 percent this year as SDR effort moved to the phone. Using 51 as the base is a bet that Term 4 repeats. If it does not, this is where I miss, and I miss by a lot.

What 56 is worth: 99 percent of qualified leads get an AU deal and 36 percent of those are won, so about 20 wins, roughly $148,000 at the 2026 average recorded deal of $7,394. The whole board at 156 qualified is near $410,000, and most of that is the phone, not me.

Source: computed from `~/Documents/GTM Project/outputs/sdr_funnel/leads.jsonl` joined to company market in `lead_company_context.json`, AU only, using `date_first_entered_meeting_booked` and `hs_v2_date_entered_qualified_stage_id_233247981`; `results/baseline-2026-09.md`; `markets/au.md`.

## Why this window

In 2025, 204 of 284 wins landed on or after 4 September, 70 percent of the year, worth $1,654,495 of the $2,348,300 full year. 2026 to 3 September has 85 wins worth $628,515, against 80 worth $693,805 in the same window of 2025: the same number of deals for less money. The year is decided in the next 90 days.

Source: `results/baseline-2026-09.md`.

## The three moves

**One. Move the send volume off the weakest large sequence.** The largest AU sequence by send volume, id 299760938 in the portal because its name did not resolve, made 1,030 sends to 134 schools over 14 months for 3 meetings and 1 win. Pause it, and put its list and send budget onto hand-written email, an SDR writing to a named school individually.

Pro-rated to 90 days that is about 28 schools. Hand-written email has produced 59 meetings across 244 schools, 24 percent. At that rate 28 schools give about 7 booked meetings, and at the 66 percent rate at which email-booked meetings qualified last Term 4, 51 of 77, about 5 qualified. Every step of that is an assumption about where redirected volume goes.

The comparison behind it is not a fair one: a sequence and hand-written email differ in list, timing and targeting, and a slow sequence reads badly because a meeting only counts if it lands within 30 days of the first send. So before any volume moves I put the two lists side by side on sector, size, state and whether we had contacted them before, and re-read that sequence's meetings at 60 days instead of 30. A day of work, week one. If the lists differ materially, or the longer window closes the gap, move one is worth nothing and the commitment drops to 51. Me with the Sales Manager, first fortnight.

**Two. A three-day clock on silence, but only where silence is a warning.** Any deal from cold outreach, or from a named-target list of schools we picked and worked deliberately, with no reply in three days enters a follow-up queue and is flagged to a manager at day ten. Deals from a Free Preview or Contact Sales request, a principal conference, or a Catholic feeder-school or diocese connection are exempt. Across 842 closed deals, a reply inside three days goes with a 46.9 percent win rate on 559 deals, fifteen days or more with 16.3 percent on 49. Silence itself is not bad: of the 167 that never replied early, the 83 on warm paths won 51.8 percent and the 84 from ordinary outbound 36.9 percent, under the 43.4 percent average. Which group a deal lands in is decided by matching words in its campaign name, such as "feeder" or "conference", so that split is a judgement I made, not a fact the system holds. If the silent outbound group reached 43.4 percent, that is about 5 more wins per 84 such deals, roughly $40,000, and that is an assumption. Me to build it, the SDRs to work it, live from week two.

**Three. Stamp every send with the angle it used.** Three fields filled in when a school is added to a sequence: which angle the email leads with, who it is aimed at, and which version of the wording. Then enough volume behind the social media ban angle to settle it. Nothing today records what was said, and across the 842 closed deals, compared only against deals carrying a similar amount of recorded conversation, no angle separated the deals we won from the deals we lost; the closest thing to a positive signal was the social media ban, in 19 deals where the bar set in advance was 20. This is the only way the next 90 days end in an answer rather than another backward look. Me with the portal owner, before the first send of October. The angles worth stamping, and why, are in [message-map.md](message-map.md).

Source: `outbound-engine/sequence-inventory.md`; `customer-truth/response-speed.md`; `customer-truth/what-the-market-is-telling-us.md`; `results/baseline-2026-09.md`; the Term 4 email split computed from `~/Documents/GTM Project/outputs/sdr_funnel/leads.jsonl`.

## What I need

Edit rights on AU sequences in the production portal; I have read-only today. Sales Manager sign-off to pause a sequence a rep owns and to hold one angle constant across a list for six weeks. And one decision from you: does outbound allocation, which schools get which sequence, sit with marketing or with each rep? Move one cannot start until that is answered. The other two can.

Source: `results/metric-definitions.md`; `outbound-engine/README.md`.

## What I will stop doing

I hand my renewals book over. The 2027 renewal cohort transfers to the CS team in the first fortnight, with a written handover per account. I keep only the renewals already at quote stage, until they sign, and take nothing new. This analysis was done on top of a full book, and that is not repeatable for 90 days.

Source: my own renewals book; no file in this repo records it.

## How you will know, weekly

Four numbers every Monday, read straight from HubSpot production 20058914: meetings booked from the lead's meeting-booked date, those meetings split by call or by email, qualified meetings from the qualified stage date, and wins from closed-won. I do not report meetings held, because 70 percent of past meetings still read as scheduled and the field cannot be trusted. The job already runs: it pulls the week's changes every Sunday at 21:00 and has a digest ready by 08:00 Monday. Its first run tagged 26 closed deals with no errors.

Source: `results/metric-definitions.md`; `agents/weekly-pull-and-tag.md`; `results/baseline-2026-09.md`.

## What this plan does not claim

No angle has been proven to win: across 842 closed deals none clears the bar, and the one that came closest is a lead to check, not a finding.

There is no attribution. No message has ever been stamped with which angle it used, so nothing here tells you what to say, only what happened.

All of it is backward-looking. Running anything live before this decision was not permitted, so every figure is read off leads and deals that already closed.

And the sequence inventory only sees schools that became leads, so every rate inside it flatters us. That is why the commitment is built on the lead records instead, and the only figure taken from the inventory is move one's 5.

Source: `customer-truth/what-the-market-is-telling-us.md`; `results/metric-definitions.md`; `outbound-engine/sequence-inventory.md`.
