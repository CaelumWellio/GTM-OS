# Agents

Every agent has a spec in `_spec-template.md` format before it runs. Status table:

| Agent | Implementation | Status | Since |
|---|---|---|---|
| weekly-pull-and-tag | scripts/growth_os/run_weekly.sh | running | 2026-09-17 |

The weekly job is scheduled (launchd, Sunday 21:00) and ran end to end for the first time on 17 September 2026: it pulled deals, meetings, calls, notes and emails, tagged the 26 AU new-business deals that closed in the window with zero errors, passed all four checks, and committed the digest to the auto/weekly branch for review.

Specced and not yet built, in Five Engines build order: inbound responder, reschedule defence, attempt governor, always-on research, Baseline analyst, referral queue, renewal risk reader, proposal assembler.

Source: `~/Documents/GTM Project/outputs/growth_os/delta/2026-W38/manifest.json`; `customer-truth/weekly/2026-W38.md`; `agents/weekly-pull-and-tag.md`.