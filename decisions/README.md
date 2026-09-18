# Decisions

Format: date, decision, alternatives considered, why, who decided, what would reverse it. One file per decision, named `YYYY-MM-DD-slug.md`.

The reversal condition in `2026-09-16-first-machine.md` was met on 2026-09-17; the decision that followed is in `2026-09-17-first-machine-reversal-condition-met.md`.

Cycle one reran the customer-truth pass on the full cache once the missing HubSpot scopes were granted; see `2026-09-17-cycle-one-full-cache.md`.

## Cold read

Date: 2026-09-18.

- Comprehension bar passed. The reader could state the recommendation and the number without re-reading, and named a decision the plan changed for them.
- The same read found the central fault: the do-nothing baseline was modelled from the sequence inventory's email cache rather than measured, and was wrong by roughly three times.
- That is what prompted the correction of 18 September 2026, which rebased the number on meetings counted off the lead records and moved the commitment to the email-sourced portion.
- Four wording faults were raised in the same read and fixed: unexplained jargon, a sequence named only by a raw id, a quantity left vague, and one sentence that could not be understood on three attempts.

Source: `outbound-engine/term-4-plan.md`; `README.md`; `results/README.md`.

18 September 2026, message map. A founder persona read `outbound-engine/message-map.md` cold, alone, with five minutes. Passed the bar: named four things he would change that week, and could state what would falsify each of the three bets. He also found five weaknesses, all since fixed: the pivotal time-pressure figure was computed by hand rather than by the pipeline, the scepticism applied to the leadership number was not applied to the wellbeing one, the bets' drop conditions used words where they needed numbers, two of the three bets competed for the same list, and no figure anywhere said what a bet was worth. His verdict was that the view of what to say had arrived but the proof had not, since every bet depends on the send stamp landing first, and that the October and November outbound test should be handed over as an owned deliverable with thresholds agreed in advance.

Source: the cold read transcript, held in the session record; `outbound-engine/message-map.md`.

## Pending from Caelum

- The cost line for `README.md`. The front-door rewrite of 2026-09-18 was specified to end with one line stating the hours this work took. That figure has to come from Caelum and must not be estimated, so the line was left out of the page rather than filled with a guess. Add it to `README.md` once the number is known.

## Pending questions (from the 2026-09-15 brainstorm)

- Does primary attach travel with the role as ABM? Caelum.
- Outreach data access for UK sequences. Caelum, UK sales lead.
- Webinar recording location. Head of Marketing.
- Drive folder for case studies and kits. Head of Marketing.
- Decision deadline and case-study format. Head of Marketing.
- Numeric 2027 target for the role. Head of Marketing, founders.
- Evidence linking wellbeing to attendance and outcomes. To assemble.

## Week 2 gate

Date: 2026-09-17.

- Test suite: `python3 -m pytest scripts/growth_os/tests -q` from `~/Documents/GTM Project`, 100 passed.
- Checks: `python3 -m scripts.growth_os.checks`, OK (citations, de-identification, read-only, people, all clear).
- Plugin scope: `gtm-skills@gtm-plugins` confirmed absent from the user-level `~/.claude/settings.json` `enabledPlugins`; the plugin remains project-scoped to this repo only.
- Cold read timing: pending, Caelum to time.

Source: scripts/growth_os/tests; scripts/growth_os/checks.py; ~/.claude/settings.json.
