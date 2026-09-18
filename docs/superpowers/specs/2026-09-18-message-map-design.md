# Message map: design

Date: 18 September 2026. Status: approved, ready for an implementation plan.
Owner: Caelum. Sponsor: Head of Marketing. Final audience: the founders.
Predecessor specs: `2026-09-16-growth-os-design.md`, `2026-09-18-term4-outbound-case-design.md`.

## 1. Why this exists

A simulated founder cold read of the Term 4 pack passed on comprehension and then named what was still missing. The sharpest sentence was that the whole pack "is about measurement, and the role is about revenue", followed by: what is missing is "one piece of evidence that he can decide what we say to schools and not merely count what happened after we said it."

That is this document. It is the judgement half. The machine assembles the evidence; the opinion is written by hand, because the opinion is the thing being demonstrated.

## 2. The constraint that shapes everything

The hook analysis came back null. Across 842 closed deals no angle clears the evidence bar once like is compared with like, and the nearest thing to a positive signal was one mention short of the minimum sample.

Slicing by sector does not rescue it. Eighteen angles across five sectors is ninety comparisons, so the strong-looking cells are mostly chance. Checked directly: Catholic price mentions look plus 18.5 points on 43 deals, independent competitor mentions plus 12.1 on 33, and neither survives a fair reading of how many comparisons were run.

So this document cannot say "use angle X because it wins". It says three other things instead, and each is honest about its footing:

1. **One negative is robust enough to assert.** Talking about saving teachers time goes with losing in every cut: minus 13.0 points adjusted overall on 108 deals, minus 26.4 in primary, minus 29.1 in the unknown sector, minus 18.0 in independent. A consistent negative across independent cuts is worth more than a positive that appears in one.
2. **What schools say in their own words is evidence even when it does not predict.** Language that recurs is not proof that it wins, but it is the vocabulary the buyer already uses, and writing in it is a defensible judgement rather than a measured one.
3. **Anything asserted about what to say must be falsifiable inside one term**, which is what the three bets are for.

## 3. What gets built

One document, `outbound-engine/message-map.md`, and one module that produces its evidence sections.

### 3.1 Role taxonomy

Raw job titles are messy: 26,389 of 39,291 cached contacts carry one, and the top values run principal 5,083, deputy principal 4,833, assistant principal 3,432, chaplain 614, counsellor 597, head of wellbeing 397, associate principal 382, wellbeing leader 345, guidance officer 338, alongside noise such as "ms", "mr" and "it".

Four roles, collapsed by matching the lowercased title:

| Role | Matches on |
|---|---|
| `leadership` | principal, deputy principal, assistant principal, associate principal, acting principal, head of school, headmaster, headmistress, director of |
| `wellbeing` | wellbeing, chaplain, counsellor, guidance, psychologist, pastoral, student services, welfare |
| `curriculum` | curriculum, head of department, head teacher, learning, teaching, faculty, coordinator where it is not a wellbeing coordinator |
| `operations` | business manager, finance, bursar, it, ict, administration, registrar |

Anything else is `other` and is reported as a count, never dropped silently. The mapping table is rendered into the document so a reader can argue with it. A title matching more than one family resolves in the order wellbeing, curriculum, operations, leadership, so that "wellbeing coordinator" is wellbeing rather than curriculum, and "deputy principal, wellbeing" is wellbeing rather than leadership.

### 3.2 Layer one, deal level, full population

For each of the 842 in-scope closed deals, which roles actually engaged. A role counts as engaged when at least one inbound email in that deal's early window came from a sender whose contact record carries a title in that family. Silence and outbound sends do not count; this measures who wrote back.

Reported per role: deals where the role engaged, win rate when engaged, win rate when not, the difference, and the same thin rule as everywhere else, fewer than 20 deals is thin and carries no conclusion. Also reported: which concerns that role raises, as the distribution of school-voiced tags across deals where the role engaged.

This is an association on closed deals, not proof, and the document says so.

### 3.3 Layer two, quote level, the language bank

612 school-voiced tagged quotes can be traced to a named role, by matching the quote's `event_id` to an email object and the sender to a titled contact. The spread is deputy principal 83, head of wellbeing 70, wellbeing leader 61, assistant principal 37, principal 34, and a tail.

Per role, the document carries up to eight verbatim quotes, de-identified exactly as everywhere else: school names, staff names and email addresses scrubbed, the existing quote blocklist respected, and each labelled with the role, sector and outcome.

**The coverage limit is material and must be stated prominently.** Only 689 of 3,426 school-voiced quotes, 20 percent, came by email; the rest came from meeting notes and rep notes that carry no sender, so they cannot be attributed. The language bank therefore shows what schools write, not what they say in a room, and written language skews towards logistics and scheduling where spoken language carries more of the reasoning.

### 3.4 The point of view, hand-written

Per role: what they actually say, what to say back, and what to stop saying. Short, opinionated, and explicit about which parts rest on measurement and which on judgement. The one assertion the evidence carries on its own is the negative about time saving.

### 3.5 Three bets, hand-written

Each bet gets: what we would send, to whom, what would count as it paying off, how we would know, and by when. Each must be settleable inside one term. The stamp proposed in the Term 4 plan is what makes them settleable, and the document says so plainly, because without it none of the three can ever be resolved.

Three is the cap. A fourth would be a wish list rather than a commitment.

## 4. Architecture

New module `~/Documents/GTM Project/scripts/growth_os/role_voice.py`, following the shape of the existing analysis modules: pure functions, a `summarise`, a `render`, a `main` writing JSON and markdown. Tests in `tests/test_role_voice.py` on hand-built fixtures.

It reuses `common.py` for paths, parsing, outcome and window rules, and `write_truth.py` for `deidentify`, `scrub_people` and `school_label`, plus the quote blocklist already in place. It introduces no dependency.

Inputs, all local: `outputs/customer_truth/deals_early_text.jsonl`, `deals_tagged.jsonl`, `outputs/sdr_funnel/emails.jsonl`, and the cached contacts inside `outputs/sdr_history/<company_id>.json`. Outputs: `outputs/growth_os/role_voice.json` and the evidence sections of `outbound-engine/message-map.md`.

The module renders the evidence sections only. The point of view and the bets are written by hand into the same file, below the generated content, under headings the module does not touch. The module must therefore be safe to re-run: it regenerates its own sections and leaves the hand-written ones alone. It does this by writing the generated block between two sentinel comment lines and replacing only what lies between them when the file already exists.

## 5. Constraints

- **HubSpot is read-only. No writes, no deletes, no changes, ever.** This module makes no network calls at all.
- Python 3.9, standard library only. Module docstring first, then `from __future__ import annotations`.
- Every `## ` section ends with a `Source:` line; the citations check covers the file.
- No school name, no staff name, no email address in the rendered file. The existing four checks must pass.
- Thin rule: fewer than 20 deals in a cell means thin and no conclusion.
- No em-dashes, sentence case headings, no statistical vocabulary in the hand-written sections.
- Roles only. Never an individual, and never a rep.

## 6. Verification

- The full suite passes, currently 169 tests.
- All four checks print OK before any commit.
- Every figure in the hand-written sections traces to `role_voice.json` or to a file already in the repo. A reviewer spot-checks at least four.
- The generated block regenerates cleanly over an existing file without disturbing the hand-written sections. Tested.
- A cold read of the finished document by a fresh founder persona, five minutes, judged against one bar: can the reader say what we should say differently to a school, and what would prove it wrong.

## 7. Sequence

| Step | Deliverable | Gate |
|---|---|---|
| 1 | `role_voice.py` with tests, evidence sections generated | Thin cells marked, coverage limits stated, sentinel regeneration works |
| 2 | The point of view, hand-written | Only the time-saving negative asserted as measured; everything else labelled judgement |
| 3 | Three bets, hand-written | Each settleable inside one term, each naming what would falsify it |
| 4 | Linked from the Term 4 plan and the README | All figures agree across the pack |

## 8. Out of scope

Rewriting the SDRs' current copy, which is the Sales Manager's. Any new HubSpot property. Events and new market entry, which the cold read also named as missing and which need their own work. The fall in average deal value, which is a pricing and mix question rather than a messaging one.
