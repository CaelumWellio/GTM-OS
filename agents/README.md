# Agents

Every agent has a spec in `_spec-template.md` format before it runs. Status table:

| Agent | Implementation | Status | Since |
|---|---|---|---|
| weekly-pull-and-tag | scripts/growth_os/run_weekly.sh | dry-run | 2026-09-17 |

The weekly job is scheduled (launchd, Sunday 21:00) but stays at dry-run: its first real run on 17 September 2026 pulled deals, meetings, calls and notes, then stopped because the HubSpot private app lacks the emails read scope. It becomes running once that scope is restored and a digest completes.

Specced and not yet built, in Five Engines build order: inbound responder, reschedule defence, attempt governor, always-on research, Baseline analyst, referral queue, renewal risk reader, proposal assembler.

Source: `~/Documents/GTM Project/outputs/growth_os/delta/2026-W38/manifest.json`; `agents/weekly-pull-and-tag.md`.