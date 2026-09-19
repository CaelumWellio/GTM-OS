# Growth OS

This is the case for how we should spend the next ninety days of outbound, and the evidence I am basing it on.

## What I recommend

Back the Term 4 plan in `outbound-engine/term-4-plan.md`: 56 qualified meetings sourced from email in the ninety days to mid-December, against the 51 the same window produced last year. It asks for three things. Move the send volume off the weakest large sequence and onto hand-written email, meaning an SDR writing to a named school individually instead of enrolling that school in an automated sequence. Put a three-day clock on silence, but only for schools we contacted cold. And start recording, on every send, which angle it used, so that in ninety days we can answer the question this work could not. The view on what to say to schools, and the three bets that would settle it, is in [outbound-engine/message-map.md](outbound-engine/message-map.md).

Be clear about what is committed to and what is only context. In the equivalent window last year, 18 September to 16 December 2025, AU schools booked 315 meetings and 156 of those went on to qualify. Qualified meetings in 2026 are running at about 44 a month, roughly 131 in a ninety-day window. Those are the business numbers, measured off the lead records. But 105 of last Term 4's 156 qualified meetings were booked on a call and only 51 by email, so two thirds of the total is phone work by the SDR team that this plan does not touch. The commitment is therefore the email-sourced half: 51 is what that half produced last year, and 5 more come from the first move, an assumption about where redirected volume goes that is gated on a check the plan sets out. The second move adds no meetings at all, and the third adds none this term. There is no range, because the only upside the previous version could name was a bookkeeping correction. The named risk is that email's share of qualified meetings has fallen from 33 percent last Term 4 to 20 percent across 2026, so using 51 as the base assumes the season repeats.

The events intelligence sits alongside this in [events/](events/README.md): what the conference programme returned, a rubric for deciding the next booking, and one page in [events/2026-plan.md](events/2026-plan.md) on the three things I would change about the 2026 plan before the invoices are paid.

Three things are needed from you. Edit rights on AU sequences in the production portal, which I do not have today. Sales Manager sign-off to pause a sequence a rep owns. And one decision: does it sit with marketing or with each rep to choose which schools get which sequence? The first move cannot start until that is answered. In exchange I hand my renewals book to the CS team in the first fortnight, keeping only the renewals already at quote stage. This analysis was done on top of a full book and that is not repeatable for ninety days.

Source: `outbound-engine/term-4-plan.md`.

## What the evidence says

**Nothing we say predicts winning.** Across 842 closed AU deals, 365 won and 477 lost, no angle separates the deals we won from the deals we lost once deals are compared against other deals that had a similar amount of conversation recorded against them. The evidence bar was set before the work ran, at a minimum of 20 deals mentioning an angle and a ten point difference in win rate. Nothing cleared it. The closest thing to a positive signal was the social media ban, which came up in 19 deals and needed 20 to count, so it is a lead worth testing, not a finding. Those 842 are 99.8 percent of the 844 closed deals in the window; the other 2 have no cached history to read.

**How fast a school replies predicts a great deal.** A reply inside three days goes with a 46.9 percent win rate across 559 deals. Fifteen days or more goes with 16.3 percent across 49. But silence is only a warning sign for schools we contacted cold: of the 167 that never replied early, the 83 who arrived through Free Preview, Contact Sales, a principal conference or a diocese network won 51.8 percent, while the 84 from ordinary outbound won 36.9 percent against a 43.4 percent average. Which group a deal falls into is decided by matching words in its campaign name, not by reading a stored field, so that split is a judgement and not a fact the system holds.

**Our sequences vary enormously, and the spread is not proof.** The largest by volume made 1,030 sends to 134 schools and produced 3 meetings and 1 win, while email written by hand to 244 schools produced 59 meetings and 13 wins. Two sequences are never a fair comparison: different lists, different months, different targeting, and a slow sequence looks far worse at fourteen days than at sixty. The inventory also only sees schools that already became leads, so every rate in it flatters us. That is the single largest reason the plan no longer builds its number out of the inventory at all, and counts meetings off the lead records instead, using the inventory for one figure only, which it labels an assumption.

Source: `customer-truth/what-the-market-is-telling-us.md`; `customer-truth/response-speed.md`; `outbound-engine/sequence-inventory.md`.

## Read next

1. `outbound-engine/term-4-plan.md`: the plan, the arithmetic behind the 56, and what it does not claim.
2. `results/README.md`: what the work found, what changed because of it, and the baseline numbers.
3. `decisions/README.md`: what has been decided, and what is still waiting on someone.

Source: the three files named above, all in this repo.

## What this is not

Running anything live before this decision was not permitted, so every number here is read off leads and deals that already closed. That is why all of the work to date looks backwards, and why the plan is the forward half. These are associations, not proof.

There is also no attribution. No message we have ever sent was recorded with the angle it used, so nothing here tells you what to say, only what happened. The third move exists to fix that.

The repo is built to close one loop: what goes out to the market comes back as a measured result and changes what goes out next. Only the measuring half of that loop exists today. The weekly job that reads HubSpot and writes the week up now runs end to end, and the first digest is waiting for review, so the reporting behind the plan is reporting that already exists rather than reporting I am promising to build.

Source: `outbound-engine/term-4-plan.md`; `results/metric-definitions.md`; `agents/weekly-pull-and-tag.md`.
