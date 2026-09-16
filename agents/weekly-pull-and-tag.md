# weekly-pull-and-tag

Status: dry-run. Since: 2026-09-17.

1. **Job in one sentence.** Every Sunday night, pull what changed in HubSpot in the last eight days, tag the early conversation text of any AU new-business deal that closed, recompute the hook summary, and write a digest to branch `auto/weekly`.
2. **Data source.** HubSpot production 20058914, read-only via `HUBSPOT_PROD_TOKEN`: deals in pipeline 41942802 with `hs_lastmodifieddate` in window; meetings, calls, notes, emails with `hs_lastmodifieddate` in window plus their company and deal associations; email bodies via `/crm/v3/objects/emails/batch/read`. Existing caches under `~/Documents/GTM Project/outputs/sdr_funnel/` and `outputs/sdr_history/`.
3. **Run schedule.** Sunday 21:00 local, launchd `com.wellio.growthos.weekly`. Manual: `bash scripts/growth_os/run_weekly.sh`.
4. **Filters.** Skip deals not in pipeline 41942802; skip `dealtype` outside {newbusiness, New Business - Primary}; skip events with empty body; skip `auto_reply` emails; skip anything already tagged for that deal id.
5. **Expected output.** `customer-truth/weekly/<ISO week>.md` in Growth OS on branch `auto/weekly`; `outputs/growth_os/delta/<week>/*.jsonl` and `outputs/growth_os/digests/<week>.md` in GTM Project; appended rows in `outputs/customer_truth/deals_tagged.jsonl`.
6. **Approval step.** Caelum reads the branch diff Monday and merges, edits or rejects. Nothing reaches `main` otherwise.
7. **Metric.** Digest delivered by 08:00 Monday with every closed deal of the week tagged or listed under "needs Caelum". Target: 100 percent of weeks.
8. **What breaks it.** Token expired or scope missing (check: portal assertion fails, job exits non-zero, log line). Mac asleep (check: digest file missing Monday; fallback is the manual command). Claude JSON parse failure (check: `error` field on the tagged row; digest lists it). Company with no timeline file (check: digest lists "no text"). Deal closed but `closedate` outside window (check: digest counts deals by close week).

Implementation: `scripts/growth_os/pull_delta.py`, `weekly_digest.py`, `run_weekly.sh`, launchd plist.
Corrections log: none yet.
