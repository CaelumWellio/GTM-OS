# Method: the customer-truth pass

## Question
What did the schools that bought say early in the conversation that the schools that did not buy never said? Which named pain points convert at a higher rate, at a larger deal size, in a shorter cycle?

## Population
Sales Pipeline AU (41942802) deals, `dealtype` in {newbusiness, New Business - Primary}, closed won or lost. Won means `hs_is_closed_won = true`. Open deals excluded from rates.

## Inputs
- `~/Documents/GTM Project/outputs/sdr_funnel/deals.jsonl` (986 deals, pulled 3 Sep 2026, production 20058914)
- `~/Documents/GTM Project/outputs/sdr_history/<company_id>.json` (5,513 company timelines, pulled late Aug 2026)
- `~/Documents/GTM Project/outputs/sdr_funnel/lead_company_context.json` (company name, state, governing body)

## Early window
From 30 days before deal creation until the deal first entered Evaluating (155744137), else Proposal Sent (88376591), else its close date. Text after that is excluded so the pass measures what was said early, not what was said to close.

## Text selection
Events of type email, meeting, note, call with a non-empty body and not an auto-reply. Priority: inbound emails (school voice), meeting notes, notes, calls, outbound emails. Each event capped at 3,000 characters, each deal at 12,000.

## Stages and commands (run from the GTM Project root)
1. `python3 -m scripts.growth_os.assemble_early_text` -> `outputs/customer_truth/deals_early_text.jsonl`
2. `python3 -m scripts.growth_os.tag_deals --model sonnet` -> `deals_tagged.jsonl` (resumable; `--force` retags)
3. `python3 -m scripts.growth_os.join_outcomes` -> `summary.json`, `summary.csv`
4. `python3 -m scripts.growth_os.write_truth --mode "cycle zero (retrospective)"` -> `what-the-market-is-telling-us.md` and a run entry below
5. `python3 -m scripts.growth_os.checks` -> must print OK before anything is committed to main

## Rules
- A tag counts only when the school raised it (speaker = school). Wellio-voiced mentions are stored, not counted.
- Cells with fewer than 20 present deals are thin. Nothing in the truth file is promoted from a thin cell.
- Quotes are verbatim, at most 200 characters, de-identified to school type and state in this repo.
- Codebook version and model are recorded on every tagged row.
- Rebuild from raw quarterly. Weekly runs append; they do not rewrite the truth file.
- Quotes are scrubbed per deal, using that deal's own school names, staff names and email addresses. The automated check verifies full names and email addresses across the founder-facing files, but deliberately does not scan for single first names, because surnames collide with ordinary words. A name belonging to someone who is not a contact on that deal would not be caught mechanically and is a read-before-publish item.

## Known limits
Meeting outcomes unrecorded (70 percent of past discovery meetings still SCHEDULED); a third of meetings untyped; call dispositions only from Feb 2026; loss labels unreliable (128 of 214 audited), notes used instead. Source: `~/Documents/GTM Project/docs/revops/2026-09-03-sdr-funnel-leak-analysis.md`.

## Run log
