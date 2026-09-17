# Term 4 outbound case: design

Date: 18 September 2026. Status: approved, ready for an implementation plan.
Owner: Caelum. Sponsor: Head of Marketing. Final audience: the founders.
Predecessor spec: `2026-09-16-growth-os-design.md` (the Growth OS repo and the customer-truth machine).

## 1. Why this exists

A simulated cold read of the repo by a sceptical founder persona returned a clear verdict: the work earns trust on character, because it reports a null result honestly, and fails on purpose, because it never says what to do. The exact words were "he has done the hard part and then stopped one inch short of the part I pay for", and "put him in a room with me for an hour, no slides, and let him tell me what he would do with outbound in the next ninety days and what number he will sign up to".

This project produces that. The repo is the credential. The plan is the argument.

## 2. Decisions already taken

| Question | Decision |
|---|---|
| Deliverable shape | One short plan document sent ahead of the meeting. The repo is linked evidence, not an attachment. |
| What the 90 days target | Term 4 AU outbound, the window where 70 percent of 2025 revenue landed. |
| The number committed to | Qualified meetings booked. Measurable today from the lead pipeline. |
| Routing | Head of Marketing reviews, then carries it to the founders. Framed as a proposal Caelum would run if given the role, never as a change to anyone's team today. |
| Package shape | Option A of three: one plan, repo as evidence. |
| Out of scope | Deal-size decomposition, new market entry, the EDM engine build, anything that writes to HubSpot. |

## 3. Absolute constraints

- **HubSpot is read-only. No writes, no deletes, no changes of any kind, ever.** Allowed HTTP is GET, and POST only to paths ending `/search` or `/batch/read`. Production portal 20058914 only, asserted before any call. This is the user's standing instruction and overrides any convenience.
- No sends, no sequence enrolments, no property creation, no list changes.
- Founder-facing files carry no school name, no staff name, no email address. The existing four checks (citations, de-identification, read-only, people) run before anything is published.
- Python 3.9, standard library only. Scripts live in `~/Documents/GTM Project/scripts/growth_os/`. Large outputs stay under `outputs/`. No file outside those two paths is modified.
- Prose: no em-dashes, sentence case, every figure carries a source and an n.
- Aggregate by sequence, persona and market. Never by individual rep.

## 4. What gets built

Four pieces, in dependency order. The first two produce the evidence, the third makes the argument, the fourth fixes the front door.

### 4.1 Response-speed triage

**The question.** The founder asked whether the 27 versus 73 day cycle-length gap is cause or effect. Cycle length is largely effect, because lost deals linger before anyone marks them lost. The usable question is whether responsiveness observable early predicts the outcome.

**What is already measured**, on 842 in-scope closed deals, base win rate 43.3 percent:

| Days from deal creation to the school's first reply | Win rate | n |
|---|---|---|
| 0 to 3 | 46.9% | 559 |
| 4 to 14 | 31.3% | 67 |
| 15 or more | 16.3% | 49 |
| No reply in the early window | 44.3% | 167 |

Supporting: schools replying more than once per email sent win 61.0 percent (n=59). Wellio sending 11 or more emails in the early window wins 37.7 percent against 48.4 percent at three to five sends.

**The blocking anomaly.** The no-reply group wins 44.3 percent, higher than both slow-reply groups. Until that is explained the finding cannot become advice, because the obvious rule ("deprioritise schools that have not replied") would be wrong for a group that wins at the base rate. Hypothesis to test: those deals close through routes that leave no early email trail, such as Free Preview or Contact Sales inbound, referral, diocese or conference. The diagnostic splits the no-reply group by `hs_analytics_source`, `campaign_source`, dealtype and segment, and compares against the replied group.

**Output.** A new file `customer-truth/response-speed.md` holding the question, the table, the anomaly and its resolution, the stated rule, and the limits. One `Source:` line per section. Where the anomaly resolves as expected, the rule is conditional: it applies to outbound-sourced deals only, and names the sources it excludes.

**Method file.** The diagnostic is a script, `scripts/growth_os/response_speed.py`, re-runnable, reading only local caches. Its method is recorded in `customer-truth/method.md` alongside the existing pass.

### 4.2 AU sequence inventory

**Why it is possible now.** The cached email objects carry `hs_sequence_id`. There are 62,167 sequence-tagged emails across 401 distinct sequences spanning 30 June 2025 to 3 September 2026, and 61,367 of them carry a company association. Nothing new needs pulling.

**Coverage honesty.** Sequence-tagged emails are 36.5 percent of the 170,422 cached. The remaining 63.5 percent are manual sends. The inventory therefore describes the sequenced portion of outbound only, and says so.

**Per sequence, computed:** sends, distinct schools touched, schools that replied (an inbound non-auto-reply email from that company within 14 days of a send), reply rate, meetings booked (a lead for that company entering the meeting-booked stage within 30 days of the first send), deals created (a Sales Pipeline AU deal created for that company within 60 days of the first send), deals won, and recorded amount won.

**Attribution is touch-based, not causal**, and every output states this. A school touched by several sequences is credited to each, so column totals exceed reality; the file reports that explicitly rather than apportioning. A control group is included: schools contacted in the window with no sequence-tagged email at all, measured the same way, so sequence performance can be read against the unsequenced baseline.

**Sequence names.** Emails carry only the id. Names are fetched read-only with `GET /automation/v4/sequences` if the token's `automation.sequences.read` scope allows it. If that call fails, the inventory reports ids and says names were unavailable, rather than guessing.

**Shape of the output.** 401 sequences is a long tail. `outbound-engine/sequence-inventory.md` reports the top 20 by send volume as a table, the aggregate for everything below that as one row, the control group, and the three to five sequences whose reply rate is furthest from the median in each direction. The full 401-row table goes to `outputs/growth_os/sequence_inventory.csv` in GTM Project, referenced by path.

**Thin-cell rule.** The same discipline as customer-truth: a sequence with fewer than 20 schools touched is marked thin and no conclusion is drawn from it.

### 4.3 The 90-day Term 4 plan

**File.** `outbound-engine/term-4-plan.md` in the Growth OS repo. Two pages, roughly 900 words. It is the document sent ahead.

**Required structure**, in this order:

1. **The number, in the first two lines.** Qualified meetings booked in the 90 days to mid-December, stated as a commitment, with the baseline it is measured against.
2. **Why this window.** One paragraph on Term 4, using the recorded figure that 204 of 284 2025 wins (70 percent) landed on or after 4 September.
3. **The three moves.** Each gets: what it is, the evidence from this repo that motivates it, what it is worth at current conversion, who does it, and when. The moves are drawn from the two analyses above, not chosen in advance of them; the plan is written after the evidence exists.
4. **What I need.** Specific, small, and addressed to named roles rather than people: access, approvals, and one decision.
5. **What I will stop doing.** Credibility depends on this being real. The renewals book hand-over is the honest answer.
6. **How you will know weekly.** The measurement, where it comes from, and the fact that it is already automated.
7. **What this plan does not claim.** Explicitly: no hook has been proven, attribution does not exist yet, and the analysis is retrospective because running anything live before the decision was not permitted.

**Deriving the number.** It must be arithmetic from figures in the repo, shown in the plan, not asserted. The chain: sequenced schools available in Term 4, times the reply rate the inventory measures, times the meeting rate the inventory measures, adjusted for the response-speed rule's effect on where effort goes. The plan states the assumption behind each step and the figure it comes from. A target that cannot be derived this way is not used.

**Tone.** The founder's complaint was that the language was written for its author. The plan uses no statistical vocabulary at all: no strata, no quartile, no multiple-comparison correction, no volume-adjusted. Where a statistical idea is load-bearing it is said in plain words, for example "we compared like with like by grouping deals by how much conversation was logged".

### 4.4 The rewritten front door

Both founder-facing pages are rewritten against the specific failures the cold read named.

| Cold read finding | Required fix |
|---|---|
| No recommendation anywhere | The README's first section after the one-line description is the recommendation, pointing at the plan. |
| Opening paragraph is architecture, not argument | Replace it. Lead with what was found and what follows, not with what the repo is. |
| Jargon unknown to the reader | Remove strata, quartile, multiple-comparison correction, volume-adjusted, branch, scopes, superpowers from both pages. The full statistical account stays in `customer-truth/`, where a reader who wants it will go. |
| "Promotion bar" collides with promoting a person | Rename throughout the founder-facing pages to "the evidence bar". The code's `promotable()` keeps its name; this is a prose change. |
| The Results page contains no results | It reports outcomes: what the analyses found and what changed. What exists moves to a short "what has been built" line at the end. |
| Page two repeats page one | Deduplicate. Each page earns its own read. |
| Backward-looking with no explanation | One sentence stating that running anything live before the decision was not permitted, so the work to date is necessarily retrospective, and the plan is the forward half. |
| No cost stated | One line: the hours spent and what it displaced. Caelum supplies the figure; the file must not invent it. |
| The SDR pilot has no outcome | Either report its measured outcome, now that email data is readable, or state plainly that it has not run live and therefore has none. |

## 5. Architecture and file layout

New code, all in `~/Documents/GTM Project/scripts/growth_os/`:

| File | Responsibility |
|---|---|
| `response_speed.py` | Stage-1-style: reads cached timelines and deals, computes the speed table and the anomaly split, writes a summary JSON and renders `customer-truth/response-speed.md` |
| `sequence_inventory.py` | Reads cached emails, leads, deals and meetings, computes per-sequence metrics and the control group, writes the CSV and renders `outbound-engine/sequence-inventory.md` |
| `tests/test_response_speed.py`, `tests/test_sequence_inventory.py` | Unit tests on hand-built fixtures, following the existing pattern |

Both reuse `common.py` for paths, parsing, outcome and window rules, and `write_truth.py`'s `deidentify` and `scrub_people` for any quoted text. Neither introduces a new dependency. Outputs land in `outputs/growth_os/`.

The two prose deliverables, the plan and the rewritten pages, are hand-written into the Growth OS repo. They are not generated, because judgement is the product and generated argument reads like it.

## 6. Verification

- The existing four checks pass before any commit: citations, de-identification, read-only, people. The read-only check now also scans the module that issues HTTP calls.
- The full test suite passes. It stands at 131 tests.
- Every figure in the plan and in both rewritten pages traces to a file on disk. A reviewer spot-checks at least six, including every number in the plan's derivation chain.
- A fresh cold-read simulation is run against the rewritten pages and the plan, using the same founder persona and the same five-minute limit, and its verdict is recorded. The bar is that the reader can state the recommendation and the number without re-reading, and does not ask "what do I do on Monday".
- A HubSpot write audit: grep every new script for write verbs and non-allowlisted paths, and confirm no call outside GET and POST-to-search or batch-read exists.

## 7. Sequence

| Step | Deliverable | Gate |
|---|---|---|
| 1 | Response-speed analysis, anomaly resolved | Rule stated and conditional on the anomaly's resolution |
| 2 | Sequence inventory | Thin cells marked, control group present, attribution caveat stated |
| 3 | The Term 4 plan | Every number derived from steps 1 and 2, arithmetic shown |
| 4 | README and results page rewritten | Cold-read simulation passes |

Steps 1 and 2 are independent and may run in parallel. Step 3 cannot start before both are complete, because the plan is written from the evidence rather than towards a conclusion.

## 8. Open questions carried

- The hours figure for the cost line. Caelum supplies it; nothing invents it.
- Whether the Sales Manager sees the sequence inventory before the founders do. The routing decision covers the plan; the inventory names sequence performance and may warrant the same courtesy. Raised in the plan's "what I need" section rather than decided here.
- The social media ban near-miss sits at 19 mentions against a threshold of 20. Widening the window or including open deals could settle it. Not in this scope, recorded as the next analytical step.
