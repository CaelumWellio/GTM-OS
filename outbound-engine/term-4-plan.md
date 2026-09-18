# The 90-day Term 4 plan

## The number

Start with what Term 4 actually produced last year, measured from the lead records rather than modelled from send volume. In the equivalent window, 18 September to 16 December 2025, AU schools booked 315 meetings and 156 of those went on to qualify, a qualification rate of 49.5 percent. This year is running at a similar level: qualified AU meetings from February to August 2026 average 44 a month, which is about 131 in a 90-day window. Those two numbers, 156 and 131, are the board.

I am not committing to that total, and you should know why before you read anything else. Of the 156 qualified meetings last Term 4, 105 were booked on a call and 51 by email. Calls are two thirds of how meetings get booked, they are made by the SDR team, and nothing in this plan changes them. Committing to the total would be committing to other people's work and taking credit for the season.

What I commit to is the email-sourced part, which is the only part the three moves below touch: **56 qualified meetings sourced from email in the 90 days to mid-December, against the 51 the same window produced last year.** The 156 and the 131 are context. The 56 is the commitment.

The arithmetic, in full, with every assumption named as one:

- **The base is 51.** Email sourced 51 of the 156 qualified AU meetings in the 18 September to 16 December window last year. That is counted off the lead records, one lead at a time, not derived from anything.
- **One measured number cuts against that base, and you should see it now rather than in January.** Across February to August 2026, email sourced 61 of 305 qualified AU meetings, about 9 a month, about 26 in a 90-day window. Email's share of qualified meetings has fallen from 33 percent last Term 4 to 20 percent this year, as SDR effort has moved to the phone. Using 51 as the base is therefore a bet that Term 4 behaves like Term 4 did last year. If it does not, this is where I miss, and I will miss by a lot.
- **Move one is worth 5, and it is an assumption, not a measurement.** The derivation and the condition it depends on are set out under move one below. If the check described there fails, move one is worth nothing and the commitment falls back to 51.
- **Move two is worth nothing in this number.** It acts on deals after a meeting is already booked, so it cannot add meetings. Its value is in wins, and that value is an assumption too.
- **Move three is worth nothing this term.** It is the recording change that lets the next 90 days end in an answer.

51 plus 5 is 56. There is no range. The previous version of this plan carried one, 52 to 77, where the downside was my work failing and the upside was my own double counting turning out smaller than I had assumed. An upside whose mechanism is a bookkeeping correction is not an upside, so it is gone. 56 is a single number, the downside is named above and it is real, and I could not put an honest mechanism behind a symmetric upside, so I am not claiming one.

What 56 is worth. Of qualified leads, 99 percent get an AU deal, and 36 percent of those deals are won. So 56 qualified meetings is about 20 wins, roughly $148,000 at the 2026 average recorded deal of $7,394. For scale, the whole board at last Term 4's 156 qualified works out near $410,000, and most of that is the phone, not me.

Source: computed from `~/Documents/GTM Project/outputs/sdr_funnel/leads.jsonl` joined to company market in `lead_company_context.json`, AU only, using `date_first_entered_meeting_booked` and `hs_v2_date_entered_qualified_stage_id_233247981`; `results/baseline-2026-09.md`; `markets/au.md`.

## Why this window

In 2025, 204 of 284 wins landed on or after 4 September, 70 percent of the year, worth $1,654,495 of the $2,348,300 full year. 2026 to 3 September has 85 wins worth $628,515, against 80 worth $693,805 in the same window of 2025: the same number of deals for less money. The year is decided in the next 90 days.

Source: `results/baseline-2026-09.md`.

## The three moves

**One. Move the send volume off the weakest large sequence.** The automated email sequence that HubSpot records under id 299760938 is the largest AU sequence by send volume. Its name did not resolve when the inventory was built, so it has to be found by that id inside the portal rather than by name. Over the 14 months the email cache covers it made 1,030 sends to 134 schools and produced 3 meetings and 1 win. I want to pause it and put its list and its send budget onto hand-written email, meaning an SDR writing to a named school individually instead of enrolling that school in an automated sequence.

What it is worth, and what has to be true for it to be worth anything. Pro-rated from 14 months down to 90 days, that sequence is about 28 schools. Hand-written email has produced 59 meetings across 244 schools, 24 percent. If those 28 schools book at that rate, that is about 7 booked meetings, and at the 66 percent rate at which email-booked meetings qualified in the Term 4 window last year, 51 of 77, that is about 5 qualified. Those 5 are the increment in the number above. Every step of it is an assumption about where redirected volume goes.

And it rests on exactly the comparison this work has said all along is not a fair one. Two sequences, or a sequence set against hand-written email, differ in list, timing and targeting, and a slow sequence reads badly here because a meeting only counts if it lands within 30 days of the first send. I am not going to disclaim that method and then lean on it for my number. For the 5 to hold, two things have to be true: the 134 schools on that sequence have to be broadly like the 244 worked by hand, in sector, size, state and whether we had contacted them before; and the sequence's poor showing has to survive a longer window. So before any volume moves I will put the two lists side by side on those fields, and re-read that sequence's meetings at 60 days instead of 30. That is a day of work and it happens in week one. If the lists differ materially, or the longer window closes the gap, I will say so, move one is worth nothing, and the commitment drops to 51. Me with the Sales Manager, first fortnight.

**Two. A three-day clock on silence, but only where silence is a warning.** Any deal that came from cold outreach, or from a named-target list of schools we picked and worked deliberately, with no reply in three days enters a follow-up queue and is flagged to a manager at day ten. Deals that arrived through a Free Preview or Contact Sales request, a principal conference, or a Catholic feeder-school or diocese connection are exempt. Across 842 closed deals, a reply inside three days goes with a 46.9 percent win rate on 559 deals, fifteen days or more with 16.3 percent on 49. Silence itself is not bad: of the 167 that never replied early, the 83 on warm paths won 51.8 percent and the 84 from ordinary outbound 36.9 percent, under the 43.4 percent average. Which of those two groups a deal lands in is decided by matching words in its campaign name, such as "feeder" or "conference", not by reading a stored field, so that split is a judgement I made and not a fact the system holds. None of this is in the number above, because it acts after a meeting is booked and so cannot add meetings. If the silent outbound group reached 43.4 percent, that is about 5 more wins per 84 such deals, roughly $40,000, and that is an assumption. Me to build it, the SDRs to work it, live from week two.

**Three. Stamp every send with the angle it used.** Three fields filled in at the moment a school is added to a sequence: which angle the email leads with, who it is aimed at, and which version of the wording it is. Then enough volume behind the social media ban angle to settle it. Here is why that is needed. Deals with more conversation recorded against them are more likely to be won whatever was actually said, so before comparing angles we sorted the 842 closed deals into groups by how much conversation each one had, then compared angles only within a group, against deals that had a similar amount of conversation. Inside those groups no angle separated the deals we won from the deals we lost. The closest thing to a positive signal was the social media ban, which came up in 19 deals where the bar set in advance was 20. Worth this term: nothing, and nothing for it is in the number. It is the only way the next 90 days end in an answer rather than another backward look. Me with the portal owner, before the first send of October. The angles worth stamping, and why, are set out in [message-map.md](message-map.md).

Source: `outbound-engine/sequence-inventory.md`; `customer-truth/response-speed.md`; `customer-truth/what-the-market-is-telling-us.md`; `results/baseline-2026-09.md`; the Term 4 email split computed from `~/Documents/GTM Project/outputs/sdr_funnel/leads.jsonl`.

## What I need

Edit rights on AU sequences in the production portal; I have read-only today. Sales Manager sign-off to pause a sequence a rep owns and to hold one angle constant across a list for six weeks. And one decision from you: does outbound allocation, meaning which schools get which sequence, sit with marketing or with each rep? Move one cannot start until that is answered. The other two can.

Source: `results/metric-definitions.md`; `outbound-engine/README.md`.

## What I will stop doing

I hand my renewals book over. The 2027 renewal cohort transfers to the CS team in the first fortnight, with a written handover per account. I keep only the renewals already at quote stage, until they sign, and take nothing new. I am not proposing to do both jobs: this analysis was done on top of a full book, and that is not repeatable for 90 days.

Source: my own renewals book; no file in this repo records it.

## How you will know, weekly

Four numbers every Monday, read straight from HubSpot production 20058914: meetings booked from the lead's meeting-booked date, those meetings split by whether they were booked on a call or by email, qualified meetings from the qualified stage date, and wins from closed-won. The split matters because the commitment is on the email half and the total is not mine to claim. I do not report meetings held, because 70 percent of past meetings still read as scheduled and the field cannot be trusted. The job already runs: every Sunday at 21:00 it pulls the week's changes and has a digest ready by 08:00 Monday, which I review before anything is filed. Its first run tagged 26 closed deals with no errors, so this is reporting that exists, not reporting I am promising to build.

Source: `results/metric-definitions.md`; `agents/weekly-pull-and-tag.md`; `results/baseline-2026-09.md`.

## What this plan does not claim

No angle has been proven to win: across 842 closed deals none clears the bar, and the one that came closest is a lead to check, not a finding.

There is no attribution. No message has ever been stamped with which angle it used, so nothing here tells you what to say, only what happened.

All of it is backward-looking. Running anything live before this decision was not permitted, so every figure is read off leads and deals that already closed. These are associations, not proof.

And the sequence inventory only sees schools that became leads, so every rate inside it flatters us. That is why the commitment is built on the lead records instead, where a booked meeting is a booked meeting and a qualified meeting is a qualified meeting, and why the only figure taken from the inventory is move one's 5, which is labelled an assumption and gated on the check described above.

Source: `customer-truth/what-the-market-is-telling-us.md`; `results/metric-definitions.md`; `outbound-engine/sequence-inventory.md`.
