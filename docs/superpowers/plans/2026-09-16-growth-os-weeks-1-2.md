# Growth OS Weeks 1 and 2 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up the Growth OS repo and run the customer-truth machine end to end: cycle zero over 813 closed AU new-business deals, then the first automated weekly delta and digest on a branch.

**Architecture:** Two repos with one direction of flow. GTM Project (`~/Documents/GTM Project`) is the workshop: it already holds the HubSpot caches and gets a new `scripts/growth_os/` package that assembles early-conversation text per deal, tags it with headless Claude, joins tags to outcomes, and renders findings. Growth OS (`~/Documents/growth-os`) is the memory: it receives only rendered markdown, curated quotes and small summaries, each with a `Source:` line. Automation commits to branch `auto/weekly`; Caelum merges to `main`.

**Tech Stack:** Python 3.9 standard library only (no `requests`; use `urllib` like the existing pull script), pytest 8 via `python3 -m pytest`, headless Claude Code (`claude -p --output-format json`), git, launchd.

## Global Constraints

- Read-only against HubSpot. Production portal `20058914` only. Every script that touches the API asserts the portal ID before doing anything else. Allowed HTTP: `GET`, and `POST` only to paths ending in `/search`, `/batch/read`. No `PATCH`, `PUT`, `DELETE`, `/batch/create`, `/batch/update`, `/batch/archive`.
- Token stays in `~/.hubspot_prod_env` (variable `HUBSPOT_PROD_TOKEN`). Never written to any file in either repo.
- No file in GTM Project is moved or renamed. New files only under `scripts/growth_os/` and `outputs/growth_os/`, `outputs/customer_truth/`.
- Nothing founder-facing contains a school name. Founder-facing means every `.md` under `growth-os/customer-truth/` except `method.md`, everything under `growth-os/results/`, and `growth-os/README.md`.
- No student data enters Growth OS. Only deal, company and engagement text.
- Python 3.9 syntax: `from __future__ import annotations`, `Optional[...]`, no `match`, no `X | Y` at runtime.
- Prose style in every markdown file: no em-dashes, sentence case, numbers carry a source and an n.
- Commits: GTM Project files commit to the home-directory repo (`git -C ~`), which is the existing practice. Growth OS files commit to `~/Documents/growth-os`. Every commit message ends with `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`.
- Paths in code come from `common.py` constants, never hard-coded elsewhere. Environment overrides `GTM_PROJECT` and `GROWTH_OS` exist for tests.

## Domain facts the code depends on (verified 16 Sep 2026)

- Sales Pipeline AU id `41942802`. Stage ids: `88376587` Discovery, `88376590` Demo, `155744137` Evaluating, `88376591` Proposal Sent, `94980881` Proposal Signed, `94980882` Invoice Sent, `261917770` Handover Complete, `88376592` Invoice Paid, `88376593` Closed lost. Won is `hs_is_closed_won == "true"` (358 deals), not a stage test, because won deals sit in four different stages.
- `outputs/sdr_funnel/deals.jsonl`: 986 rows, 813 closed (358 won, 455 lost), 173 open. `dealtype` values: `newbusiness` 786, `New Business - Primary` 186, plus 11 upsell or null rows to exclude. `company_ids` is sometimes a JSON list and sometimes a string like `"['20469693501']"`; parse both.
- `outputs/sdr_history/<company_id>.json`: 5,513 files. Key `timeline` is a list of events with fields `type` (email, call, task, meeting, note), `id`, `ts` (ISO, no timezone), `direction` (`EMAIL` outbound, `INCOMING_EMAIL` inbound, `OUTBOUND`/`INBOUND` for calls), `subject`, `body` (max ~6,400 chars), `outcome`, `activity_type`, `disposition`, `auto_reply`. 714 of 813 closed deals have a company file; 713 have more than 200 chars of text in their early window; median 39,709 chars, so text must be capped before tagging.
- `outputs/sdr_funnel/lead_company_context.json`: `companies` dict keyed by company id with `name`, `state`, `governing_body`, `market`; `pipelines` dict maps pipeline id to name.
- Headless Claude: `claude -p '<prompt>' --output-format json --model sonnet` reads stdin, returns JSON whose `result` field holds the model text, usually wrapped in a ```json fence. Binary at `/Users/cleonard1998/.local/bin/claude`.
- `gh` is not installed. The GitHub step offers Homebrew or the web UI.

## File structure

Growth OS (`~/Documents/growth-os`):

| Path | Responsibility |
|---|---|
| `.gitignore` | Blocks env files, JSONL, outputs, OS junk |
| `CLAUDE.md` | Read order, plugin file mapping, rules for Claude in this repo |
| `README.md` | The five-minute read; rewritten at end of week 2 |
| `customer-truth/README.md`, `hook-codebook.md`, `method.md`, `what-the-market-is-telling-us.md` (generated), `weekly/` (generated), `quotes/README.md` | Truth and its method |
| `markets/README.md`, `au.md`, `uk.md`, `europe.md`, `canada.md`, `international.md`, `need-data-sources.md` | Market files |
| `outbound-engine/README.md` | What exists and what does not |
| `content-engine/README.md`, `creative-testing/README.md`, `creative-testing/ledger.csv` | Templates and the sample rule |
| `agents/README.md`, `_spec-template.md`, `weekly-pull-and-tag.md` | Spec format and the first spec |
| `results/README.md`, `metric-definitions.md`, `baseline-2026-09.md` | Founder-facing numbers |
| `decisions/README.md` and three dated entries | Decision log |

GTM Project (`~/Documents/GTM Project/scripts/growth_os/`):

| File | Responsibility |
|---|---|
| `common.py` | Paths, stage constants, parsing helpers, outcome and window rules, segment rule |
| `assemble_early_text.py` | Stage 1: one record per deal with capped, speaker-labelled early text |
| `tag_deals.py` | Stage 2: tag each record with headless Claude; resumable |
| `join_outcomes.py` | Stage 3: per-tag win rate, amount, cycle; thin flags |
| `write_truth.py` | Stage 4: de-identified markdown into Growth OS, run log into method.md |
| `pull_delta.py` | Weekly read-only pull of changed deals and engagements |
| `weekly_digest.py` | Tag the week's closed deals, recompute summary, write the digest |
| `checks.py` | Citation, de-identification and read-only checks; exit non-zero on failure |
| `run_weekly.sh` | Sunday job: pull, digest, commit to `auto/weekly` |
| `tests/` | pytest suite with fixtures in `tmp_path` |

Outputs: `~/Documents/GTM Project/outputs/customer_truth/{deals_early_text.jsonl, deals_tagged.jsonl, summary.json, summary.csv}` and `outputs/growth_os/{delta/<week>/, digests/, logs/}`.

---

## Week 1

### Task 1: Repo hygiene and skeleton

**Files:**
- Create: `~/Documents/growth-os/.gitignore`
- Create: `~/Documents/growth-os/CLAUDE.md`
- Create: `~/Documents/growth-os/README.md`
- Create: one `README.md` in each of `customer-truth/`, `customer-truth/quotes/`, `markets/`, `outbound-engine/`, `content-engine/`, `creative-testing/`, `agents/`, `results/`, `decisions/`
- Create: `decisions/2026-09-16-standalone-repo.md`, `decisions/2026-09-16-first-machine.md`, `decisions/2026-09-16-plugin-scope.md`

**Interfaces:**
- Produces: the folder layout every later task writes into; `CLAUDE.md` read order that `write_truth.py` and `weekly_digest.py` rely on when a human opens the repo.

- [ ] **Step 1: Write `.gitignore`**

```gitignore
# secrets and env
*.env
.hubspot_prod_env
.env*

# data never lives here
*.jsonl
outputs/
*.csv.bak

# OS and editor
.DS_Store
.vscode/
*.swp
```

Note: `creative-testing/ledger.csv` and `outbound-engine/lists/*.csv` are allowed; only `.jsonl` and `outputs/` are blocked.

- [ ] **Step 2: Write `CLAUDE.md`**

```markdown
# Working in Growth OS

This repo is Wellio's marketing memory. Read in this order before doing anything:

1. `README.md` (what this is, what has run)
2. `decisions/README.md` (current decisions and pending questions)
3. The folder README for the area you are working in

## Rules

- Findings live in files with a `Source:` line. Chat output that is not filed does not exist.
- Never copy raw data in. Cite GTM Project paths (`~/Documents/GTM Project/...`) or URLs.
- Never write to HubSpot from this repo. Read-only, production portal 20058914.
- Founder-facing files carry no school names. Use school type and state.
- No em-dashes. Sentence case. Every number has a source and an n.
- Automated runs commit to `auto/*` branches. Only Caelum merges to `main`.

## Plugin file mapping (gtm-skills)

The gtm-skills plugin is enabled at project scope. Its skills expect a bootstrap workspace.
Do not run its `bootstrap` skill. When a skill asks for these files, use these instead:

| Plugin expects | Use |
|---|---|
| `strategy/brand.md` | `customer-truth/language.md` and `customer-truth/buyer-and-champion.md` |
| `about/me.md` | `content-engine/voice.md` |
| `PROGRESS.md` | `decisions/README.md` |
| `content/` | `content-engine/briefs/` |

Skills that produce emoji headings or Title Case must be overridden to this repo's style.

## Where the machines live

Scripts: `~/Documents/GTM Project/scripts/growth_os/`. Run from the GTM Project root after `source ~/.hubspot_prod_env`.
Method for the customer-truth pass: `customer-truth/method.md`.
```

- [ ] **Step 3: Write the root `README.md` (status version; rewritten in Task 15)**

```markdown
# Growth OS

Wellio's marketing memory. One repo that holds what the market has told us, the machines that act on it, the job spec of every agent, and the results.

**Status, 16 September 2026:** scaffold. Nothing has run yet. The first machine (the customer-truth pass) runs in the week of 23 September.

## Read this in five minutes

1. `results/README.md`: what has been built and the numbers behind it.
2. `customer-truth/README.md`: what schools say when they buy.
3. `decisions/README.md`: what has been decided and what is still open.

## The loop

Observe (weekly read-only pull) -> Learn (tag and rank on a branch) -> Decide (Caelum merges) -> Act (stamped sequences and briefs) -> Measure (scorecard) -> back to Observe.

Spec: `docs/superpowers/specs/2026-09-16-growth-os-design.md`.
```

- [ ] **Step 4: Write the nine folder READMEs**

`customer-truth/README.md`:
```markdown
# Customer truth

What schools said, in their words, when they bought and when they did not.

- `what-the-market-is-telling-us.md`: the approved finding. Generated by the pass, merged by Caelum.
- `hook-codebook.md`: the tags and their trigger phrases.
- `method.md`: how the pass runs and the log of every run.
- `weekly/`: automated digests, one per ISO week, written to branch `auto/weekly`.
- `quotes/`: curated, de-identified quotes by hook.

Status: not yet run.
```

`customer-truth/quotes/README.md`:
```markdown
# Quotes

De-identified quotes by hook, school type and state. Populated by `write_truth.py` and curated by hand. No school names.
```

`markets/README.md`:
```markdown
# Markets

One file per market, same headings. The axis that decides channel choice is compliance versus discretionary: the UK buys wellbeing because PSHE is a curriculum requirement (about $2,000 average deal), AU buys on discretion (about $10,900 average via Free Preview and Contact Sales in 2026). Europe and Canada must be classified on this axis before anything else.

Source: brainstorm 2026-09-15 Q13; Five Engines 01-hubspot-findings.md.
```

`outbound-engine/README.md`:
```markdown
# Outbound engine

What exists: the Sales Signals pilot (20 schools, dry-run, zero HubSpot writes, awaiting Sales Manager), the sdr-outreach skill, the primary attach spine (202 K-12 schools, scraped junior-school contacts at 99).
What does not exist: any marketing-owned EDM engine. SDRs self-author sequences. AU runs in HubSpot, UK in Outreach. Meeting rates are not measured.

Source: `~/Documents/GTM Project/docs/revops/2026-09-05-sdr-outreach-pilot-assessment.md`; brainstorm Q6.

Reporting rule: sequence performance is aggregated by sequence, persona and market. Never by rep.
```

`content-engine/README.md`:
```markdown
# Content engine

Rule: a brief starts from a tagged hook in `customer-truth/`, never from a topic. Empty until the codebook has run.
Plugin skills behind this folder: webinar-content-and-events, content-strategy-and-planning, linkedin, seo-and-aeo-strategy.
```

`creative-testing/README.md`:
```markdown
# Creative testing

An experiment is any controlled comparison: subject lines, hooks to one persona, follow-up timing, page variants. Zero spend required.

## Minimum sample rule

A hook is promoted or demoted in `customer-truth/what-the-market-is-telling-us.md` only when it has been present in at least 20 closed deals and the win-rate difference against absence is at least 10 percentage points. Below that the cell is marked thin and nothing changes.

Why: the 2026-08-23 quote-harvest run showed a 53 percent lift on 30 schools regress to one point at 705. Source: `~/Documents/GTM Project/docs/revops/2026-08-23-quote-harvest-plan.md`.

Ledger: `ledger.csv` with columns hypothesis, variant_a, variant_b, metric, n_needed, n_reached, result, decision, date.
```

`agents/README.md`:
```markdown
# Agents

Every agent has a spec in `_spec-template.md` format before it runs. Status table:

| Agent | Implementation | Status | Since |
|---|---|---|---|
| weekly-pull-and-tag | scripts/growth_os/run_weekly.sh | specced | 2026-09-16 |
```

`results/README.md`:
```markdown
# Results

One page for a founder. Populated at the end of week 2 with the baseline and the first finding. Until then: nothing has run, and this page says so.
```

`decisions/README.md`:
```markdown
# Decisions

Format: date, decision, alternatives considered, why, who decided, what would reverse it. One file per decision, named `YYYY-MM-DD-slug.md`.

## Pending questions (from the 2026-09-15 brainstorm)

- Does primary attach travel with the role as ABM? Caelum.
- Outreach data access for UK sequences. Caelum, UK sales lead.
- Webinar recording location. Head of Marketing.
- Drive folder for case studies and kits. Head of Marketing.
- Decision deadline and case-study format. Head of Marketing.
- Numeric 2027 target for the role. Head of Marketing, founders.
- Evidence linking wellbeing to attendance and outcomes. To assemble.
```

- [ ] **Step 5: Write the three decision entries**

`decisions/2026-09-16-standalone-repo.md`:
```markdown
# Standalone repo

Date: 2026-09-16
Decision: Growth OS is its own repository at `~/Documents/growth-os`, not a folder inside GTM Project.
Alternatives: a folder inside GTM Project; restructuring GTM Project into the six-folder layout.
Why: the repo must be clean enough to open in front of a founder; the home directory is the git root for GTM Project, which makes its history unshowable; restructuring would break skill paths.
Decided by: Caelum.
Reversed if: GTM Project moves to its own repo and the two are merged deliberately.
```

`decisions/2026-09-16-first-machine.md`:
```markdown
# First machine: customer-truth pass

Date: 2026-09-16
Decision: the first working machine is the customer-truth pass over closed AU new-business deals, run retrospectively (cycle zero), then weekly.
Alternatives: outbound engine from the Signals pilot (blocked on Sales Manager); Europe and Canada market map (rests on an unvalidated hook).
Why: runnable now on cached data with zero HubSpot writes; tests the hook hypothesis; is the differentiator nobody else can build.
Decided by: Caelum, on recommendation.
Reversed if: the pass cannot separate winners from losers at n >= 20 on any tag, in which case the outbound engine inventory becomes the first machine.
```

`decisions/2026-09-16-plugin-scope.md`:
```markdown
# gtm-skills plugin at project scope

Date: 2026-09-16
Decision: install `gtm-skills@gtm-plugins` v2.6 at project scope in this repo only. Do not run its bootstrap skill.
Alternatives: user-level install; no plugin.
Why: its 54 skills trigger aggressively and would intrude on renewals and exec-report sessions; bootstrap would create a second workspace beside these folders.
Decided by: Caelum.
Reversed if: the plugin proves useful across the day job, in which case enable at user scope with trigger review.
```

- [ ] **Step 6: Commit**

```bash
cd ~/Documents/growth-os && git add -A && git commit -m "scaffold: gitignore, CLAUDE.md, folder READMEs, first three decisions

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

Expected: one commit; `git status --short` empty.

---

### Task 2: Codebook v1 and the first agent spec

**Files:**
- Create: `~/Documents/growth-os/customer-truth/hook-codebook.md`
- Create: `~/Documents/growth-os/agents/_spec-template.md`
- Create: `~/Documents/growth-os/agents/weekly-pull-and-tag.md`

**Interfaces:**
- Produces: the tag list `TAGS` that `tag_deals.py` copies verbatim (Task 5). Any change to the codebook bumps `CODEBOOK_VERSION`.

- [ ] **Step 1: Write `hook-codebook.md`**

```markdown
# Hook codebook

Version: 1.0 (2026-09-16). Every tagged output records the codebook version.

A tag is applied only when the school raised the idea or agreed to it in their own words. Wellio-voiced mentions are recorded with `speaker: wellio` and excluded from win-rate tables.

## Leader hooks

| Tag | Definition | Trigger phrases |
|---|---|---|
| oversight_visibility | Leader wants to see who has delivered what, across classes or campuses | "see which classes", "visibility", "oversight", "know what's being taught", "consistency across" |
| trackable_outcomes | Wants wellbeing measured over time, data to show change | "track", "measure", "data on wellbeing", "see improvement", "check-in results" |
| standardised_program | Wants one coherent program instead of ad hoc lessons | "whole-school", "consistent program", "scope and sequence", "everyone doing the same" |
| board_reporting | Needs something to put in front of the board, council, diocese or department | "report to the board", "for the diocese", "annual report", "show the principal" |
| staff_wellbeing_retention | Concern for teacher load or staff turnover | "staff burnout", "teacher wellbeing", "retention", "workload" |
| attendance_behaviour | Links wellbeing to attendance, behaviour or suspensions | "attendance", "behaviour", "suspensions", "engagement in class" |
| parent_community | Parent or community pressure or communication | "parents are asking", "community", "parent feedback" |

## Teacher hooks

| Tag | Definition | Trigger phrases |
|---|---|---|
| no_prep | Lessons ready to deliver without preparation | "no prep", "ready to go", "just open and teach", "don't have time to plan" |
| evidence_based | Wants psychology-backed or research-based content | "evidence-based", "psychology", "research", "backed by" |
| curriculum_mapped | Needs alignment to ACARA, state syllabus or PDHPE | "mapped to", "curriculum", "syllabus", "outcomes", "PDHPE", "ACARA", "V9" |
| time_saved | Time as the explicit benefit | "saves time", "hours", "off my plate" |
| student_engagement | Students respond to or enjoy the lessons | "students engaged", "kids liked", "relevant to them" |

## Commercial and situational

| Tag | Definition | Trigger phrases |
|---|---|---|
| price_budget | Price or budget raised as a factor | "budget", "cost", "price", "funding", "too expensive" |
| timing_term | Timing tied to term, year or planning cycle | "next year", "Term 4", "planning for", "not this term" |
| competitor_named | Another program or platform named | any product name, "we use", "currently with" |
| compliance_requirement | Wellbeing or PSHE as a requirement or mandate | "required", "mandated", "department expects", "compliance" |
| pilot_trial_request | Asks to trial before committing | "trial", "pilot", "try it first", "free preview" |
| social_media_ban | The 2026 social media age ban or online safety as a driver | "social media ban", "eSafety", "online safety" |

## Other

`other`: free text for anything that does not fit. The method requires reading a sample of `other` before any truth file is written, and promoting recurring themes to tags in the next codebook version.

## Change log

- 1.0, 2026-09-16: initial tags from brainstorm Q14 hypotheses plus loss-reason categories (Timing, No Interest, Price) from Five Engines findings.
```

- [ ] **Step 2: Write `agents/_spec-template.md`**

```markdown
# <agent name>

Status: specced | dry-run | running | retired. Since: YYYY-MM-DD.

1. **Job in one sentence.**
2. **Data source.** Object or path, read scope, portal.
3. **Run schedule.**
4. **Filters.** What it must skip.
5. **Expected output.** File, format, where it lands.
6. **Approval step.** Who sees it before anything acts on it.
7. **Metric.** The one number that says it worked.
8. **What breaks it.** Known failure modes and the check that catches each.

Implementation: script, skill or scheduled task that does the work.
Corrections log: dated entries added by Caelum at stage 3 of the loop.
```

- [ ] **Step 3: Write `agents/weekly-pull-and-tag.md`**

```markdown
# weekly-pull-and-tag

Status: specced. Since: 2026-09-16.

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
```

- [ ] **Step 4: Commit**

```bash
cd ~/Documents/growth-os && git add -A && git commit -m "customer-truth: hook codebook v1; agents: spec template and weekly-pull-and-tag spec

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 3: `common.py` with tests

**Files:**
- Create: `~/Documents/GTM Project/scripts/growth_os/__init__.py` (empty)
- Create: `~/Documents/GTM Project/scripts/growth_os/common.py`
- Create: `~/Documents/GTM Project/scripts/growth_os/tests/__init__.py` (empty)
- Create: `~/Documents/GTM Project/scripts/growth_os/tests/test_common.py`

**Interfaces:**
- Produces, used by every later task:
  - `GTM: Path`, `GROWTH_OS: Path`, `FUNNEL: Path`, `HISTORY: Path`, `OUT: Path`, `GOS_OUT: Path`
  - `PROD_PORTAL_ID = 20058914`, `SALES_PIPELINE_AU = "41942802"`, `STAGES: dict`, `EVALUATING`, `PROPOSAL_SENT`, `NEW_BUSINESS_TYPES: set`
  - `ids(v) -> List[str]`
  - `parse_ts(s) -> Optional[datetime]` (naive UTC)
  - `read_jsonl(path) -> List[dict]`, `write_jsonl(path, rows)`, `append_jsonl(path, rows)`
  - `outcome(deal) -> str` in {"won","lost","open"}
  - `early_window(deal) -> Tuple[Optional[datetime], Optional[datetime]]`
  - `segment(company: dict) -> str` in {"Catholic","Government","Independent","Unknown"}
  - `is_new_business(deal) -> bool`
  - `iso_week(d: date) -> str` like `2026-W39`
  - `load_companies() -> dict`

- [ ] **Step 1: Write the failing tests**

`scripts/growth_os/tests/test_common.py`:
```python
from __future__ import annotations
import datetime as dt
import json
from scripts.growth_os import common as c


def test_ids_parses_list_and_string_forms():
    assert c.ids(["1", 2]) == ["1", "2"]
    assert c.ids("['20469693501']") == ["20469693501"]
    assert c.ids("None") == []
    assert c.ids(None) == []
    assert c.ids("") == []


def test_parse_ts_handles_z_and_naive():
    assert c.parse_ts("2025-09-25T01:00:45.255Z") == dt.datetime(2025, 9, 25, 1, 0, 45, 255000)
    assert c.parse_ts("2026-04-22T05:47:47") == dt.datetime(2026, 4, 22, 5, 47, 47)
    assert c.parse_ts("None") is None
    assert c.parse_ts(None) is None


def test_outcome_uses_closed_won_flag_not_stage():
    won = {"hs_is_closed": "true", "hs_is_closed_won": "true", "dealstage": "261917770"}
    lost = {"hs_is_closed": "true", "hs_is_closed_won": "false", "dealstage": "88376593"}
    open_ = {"hs_is_closed": "false", "hs_is_closed_won": "false", "dealstage": "155744137"}
    assert c.outcome(won) == "won"
    assert c.outcome(lost) == "lost"
    assert c.outcome(open_) == "open"


def test_early_window_ends_at_evaluating_then_proposal_then_close():
    base = {"createdate": "2025-03-01T00:00:00Z", "closedate": "2025-06-01T00:00:00Z",
            "hs_v2_date_entered_155744137": "None", "hs_v2_date_entered_88376591": "None"}
    s, e = c.early_window(base)
    assert s == dt.datetime(2025, 1, 30)
    assert e == dt.datetime(2025, 6, 1)
    with_prop = dict(base, **{"hs_v2_date_entered_88376591": "2025-04-15T00:00:00Z"})
    assert c.early_window(with_prop)[1] == dt.datetime(2025, 4, 15)
    with_eval = dict(with_prop, **{"hs_v2_date_entered_155744137": "2025-04-01T00:00:00Z"})
    assert c.early_window(with_eval)[1] == dt.datetime(2025, 4, 1)


def test_segment_rule():
    assert c.segment({"governing_body": "Victorian Catholic Education Authority"}) == "Catholic"
    assert c.segment({"governing_body": "NSW Department of Education"}) == "Government"
    assert c.segment({"governing_body": "Anglican Schools Commission"}) == "Independent"
    assert c.segment({"governing_body": None}) == "Unknown"
    assert c.segment({}) == "Unknown"


def test_is_new_business():
    assert c.is_new_business({"dealtype": "newbusiness"})
    assert c.is_new_business({"dealtype": "New Business - Primary"})
    assert not c.is_new_business({"dealtype": "existingbusiness"})
    assert not c.is_new_business({"dealtype": None})


def test_iso_week():
    assert c.iso_week(dt.date(2026, 9, 27)) == "2026-W39"
    assert c.iso_week(dt.date(2026, 1, 1)) == "2026-W01"


def test_jsonl_roundtrip(tmp_path):
    p = tmp_path / "x.jsonl"
    c.write_jsonl(p, [{"a": 1}, {"b": "two"}])
    c.append_jsonl(p, [{"c": 3}])
    assert c.read_jsonl(p) == [{"a": 1}, {"b": "two"}, {"c": 3}]
    assert c.read_jsonl(tmp_path / "missing.jsonl") == []
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_common.py -q 2>&1 | tail -3
```

Expected: `ModuleNotFoundError: No module named 'scripts.growth_os'` (or import error). Create the two empty `__init__.py` files first if pytest cannot collect; the failure must then be on `common`.

- [ ] **Step 3: Write `common.py`**

```python
from __future__ import annotations
"""Shared constants and helpers for the Growth OS machines. Standard library only."""
import ast
import datetime as dt
import json
import os
import pathlib
from typing import Iterable, List, Optional, Tuple

GTM = pathlib.Path(os.environ.get("GTM_PROJECT", str(pathlib.Path.home() / "Documents" / "GTM Project")))
GROWTH_OS = pathlib.Path(os.environ.get("GROWTH_OS", str(pathlib.Path.home() / "Documents" / "growth-os")))
FUNNEL = GTM / "outputs" / "sdr_funnel"
HISTORY = GTM / "outputs" / "sdr_history"
OUT = GTM / "outputs" / "customer_truth"
GOS_OUT = GTM / "outputs" / "growth_os"

PROD_PORTAL_ID = 20058914
SALES_PIPELINE_AU = "41942802"
STAGES = {
    "88376587": "Discovery",
    "88376590": "Demo",
    "155744137": "Evaluating",
    "88376591": "Proposal Sent",
    "94980881": "Proposal Signed",
    "94980882": "Invoice Sent",
    "261917770": "Handover Complete",
    "88376592": "Invoice Paid",
    "88376593": "Closed lost",
}
EVALUATING = "155744137"
PROPOSAL_SENT = "88376591"
NEW_BUSINESS_TYPES = {"newbusiness", "New Business - Primary"}
WINDOW_LEAD_DAYS = 30


def ids(v) -> List[str]:
    """company_ids arrives as a list or as the string repr of a list. Return a list of str."""
    if isinstance(v, list):
        return [str(x) for x in v]
    if v in (None, "", "None"):
        return []
    try:
        parsed = ast.literal_eval(str(v))
    except (ValueError, SyntaxError):
        return []
    if isinstance(parsed, (list, tuple)):
        return [str(x) for x in parsed]
    return [str(parsed)]


def parse_ts(s) -> Optional[dt.datetime]:
    """ISO timestamp with or without Z to a naive UTC datetime. None for empty or 'None'."""
    if s in (None, "", "None"):
        return None
    try:
        d = dt.datetime.fromisoformat(str(s).replace("Z", "+00:00"))
    except ValueError:
        return None
    if d.tzinfo is not None:
        d = d.astimezone(dt.timezone.utc).replace(tzinfo=None)
    return d


def read_jsonl(path) -> List[dict]:
    p = pathlib.Path(path)
    if not p.exists():
        return []
    with p.open() as f:
        return [json.loads(line) for line in f if line.strip()]


def write_jsonl(path, rows: Iterable[dict]) -> None:
    p = pathlib.Path(path)
    p.parent.mkdir(parents=True, exist_ok=True)
    with p.open("w") as f:
        for r in rows:
            f.write(json.dumps(r, ensure_ascii=False) + "\n")


def append_jsonl(path, rows: Iterable[dict]) -> None:
    p = pathlib.Path(path)
    p.parent.mkdir(parents=True, exist_ok=True)
    with p.open("a") as f:
        for r in rows:
            f.write(json.dumps(r, ensure_ascii=False) + "\n")


def outcome(deal: dict) -> str:
    if str(deal.get("hs_is_closed_won")) == "true":
        return "won"
    if str(deal.get("hs_is_closed")) == "true":
        return "lost"
    return "open"


def early_window(deal: dict) -> Tuple[Optional[dt.datetime], Optional[dt.datetime]]:
    """Text counts as 'early' from 30 days before deal creation until the deal first reached
    Evaluating, else Proposal Sent, else its close date."""
    created = parse_ts(deal.get("createdate"))
    start = created - dt.timedelta(days=WINDOW_LEAD_DAYS) if created else None
    end = (parse_ts(deal.get(f"hs_v2_date_entered_{EVALUATING}"))
           or parse_ts(deal.get(f"hs_v2_date_entered_{PROPOSAL_SENT}"))
           or parse_ts(deal.get("closedate")))
    return start, end


def segment(company: dict) -> str:
    gb = (company or {}).get("governing_body")
    if not gb:
        return "Unknown"
    g = str(gb).lower()
    if "catholic" in g:
        return "Catholic"
    if "department" in g or "government" in g:
        return "Government"
    return "Independent"


def is_new_business(deal: dict) -> bool:
    return deal.get("dealtype") in NEW_BUSINESS_TYPES


def iso_week(d: dt.date) -> str:
    y, w, _ = d.isocalendar()
    return f"{y}-W{w:02d}"


def load_companies() -> dict:
    ctx = json.load((FUNNEL / "lead_company_context.json").open())
    return ctx.get("companies", {})
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_common.py -q 2>&1 | tail -3
```

Expected: `8 passed`.

- [ ] **Step 5: Commit**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: common constants, parsers, outcome and early-window rules with tests

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 4: `assemble_early_text.py` (stage 1)

**Files:**
- Create: `~/Documents/GTM Project/scripts/growth_os/assemble_early_text.py`
- Create: `~/Documents/GTM Project/scripts/growth_os/tests/test_assemble.py`

**Interfaces:**
- Consumes: `common.*`
- Produces: `outputs/customer_truth/deals_early_text.jsonl`, one record per deal:
  ```
  {"deal_id": str, "dealname": str, "outcome": "won"|"lost"|"open", "amount": float|None,
   "createdate": str, "closedate": str, "cycle_days": int|None, "source": str, "campaign": str,
   "dealtype": str, "segment": str, "state": str, "company_ids": [str], "company_names": [str],
   "window_start": str, "window_end": str, "n_events": int, "chars": int,
   "segments": [{"event_id": str, "ts": str, "kind": str, "speaker": "school"|"wellio", "text": str}]}
  ```
  Functions: `classify(ev) -> Tuple[str, str]` (kind, speaker), `select_segments(events) -> List[dict]`, `build_record(deal, companies, load_history) -> Optional[dict]`, `main(argv)`.

- [ ] **Step 1: Write the failing tests**

`scripts/growth_os/tests/test_assemble.py`:
```python
from __future__ import annotations
import json
from scripts.growth_os import assemble_early_text as a


def ev(type_, direction, body, ts="2025-03-10T00:00:00", id_="e1", auto="False"):
    return {"type": type_, "id": id_, "ts": ts, "direction": direction, "subject": "s",
            "body": body, "outcome": "None", "activity_type": "None", "disposition": "None", "auto_reply": auto}


def test_classify_speaker():
    assert a.classify(ev("email", "INCOMING_EMAIL", "x")) == ("inbound_email", "school")
    assert a.classify(ev("email", "EMAIL", "x")) == ("outbound_email", "wellio")
    assert a.classify(ev("meeting", "None", "x")) == ("meeting", "wellio")
    assert a.classify(ev("note", "None", "x")) == ("note", "wellio")
    assert a.classify(ev("call", "OUTBOUND", "x")) == ("call", "wellio")


def test_select_segments_prioritises_school_voice_and_caps():
    events = [ev("email", "EMAIL", "w" * 5000, id_="out"),
              ev("email", "INCOMING_EMAIL", "s" * 5000, id_="in"),
              ev("meeting", "None", "m" * 5000, id_="mt")]
    segs = a.select_segments(events, cap_total=8000, cap_event=3000)
    assert [s["event_id"] for s in segs] == ["in", "mt"]
    assert all(len(s["text"]) <= 3000 for s in segs)
    assert sum(len(s["text"]) for s in segs) <= 8000


def test_select_segments_skips_empty_and_auto_replies_and_tasks():
    events = [ev("task", "None", "do it"), ev("email", "INCOMING_EMAIL", "", id_="empty"),
              ev("email", "INCOMING_EMAIL", "out of office", id_="oo", auto="True"),
              ev("note", "None", "real note", id_="n")]
    assert [s["event_id"] for s in a.select_segments(events)] == ["n"]


def test_build_record_windows_and_enriches(tmp_path):
    deal = {"id": "d1", "dealname": "Test College - Y7 Essentials", "hs_is_closed": "true",
            "hs_is_closed_won": "true", "amount": "9388.75", "createdate": "2025-03-01T00:00:00Z",
            "closedate": "2025-05-01T00:00:00Z", "hs_v2_date_entered_155744137": "2025-04-01T00:00:00Z",
            "hs_v2_date_entered_88376591": "None", "hs_analytics_source": "OFFLINE",
            "campaign_source": "2024 CASPA Campaign", "dealtype": "newbusiness", "company_ids": "['c1']"}
    companies = {"c1": {"name": "Test College", "state": "NSW", "governing_body": "Catholic Schools NSW"}}
    timeline = [ev("email", "INCOMING_EMAIL", "before window", ts="2025-01-01T00:00:00", id_="old"),
                ev("email", "INCOMING_EMAIL", "in window", ts="2025-03-15T00:00:00", id_="in"),
                ev("note", "None", "after evaluating", ts="2025-04-10T00:00:00", id_="late")]

    def load_history(cid):
        return {"timeline": timeline} if cid == "c1" else None

    rec = a.build_record(deal, companies, load_history)
    assert rec["deal_id"] == "d1"
    assert rec["outcome"] == "won"
    assert rec["amount"] == 9388.75
    assert rec["cycle_days"] == 61
    assert rec["segment"] == "Catholic" and rec["state"] == "NSW"
    assert rec["company_names"] == ["Test College"]
    assert [s["event_id"] for s in rec["segments"]] == ["in"]
    assert rec["n_events"] == 1 and rec["chars"] == len("in window")


def test_build_record_returns_record_with_no_segments_when_no_history():
    deal = {"id": "d2", "dealname": "X", "hs_is_closed": "true", "hs_is_closed_won": "false",
            "amount": "None", "createdate": "2025-03-01T00:00:00Z", "closedate": "2025-05-01T00:00:00Z",
            "dealtype": "newbusiness", "company_ids": []}
    rec = a.build_record(deal, {}, lambda cid: None)
    assert rec["segments"] == [] and rec["amount"] is None and rec["segment"] == "Unknown"
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_assemble.py -q 2>&1 | tail -3
```

Expected: `ModuleNotFoundError` on `assemble_early_text`.

- [ ] **Step 3: Write `assemble_early_text.py`**

```python
from __future__ import annotations
"""Stage 1 of the customer-truth pass: one record per deal holding capped, speaker-labelled
early-conversation text drawn from the cached company timelines. Reads local caches only."""
import argparse
import datetime as dt
import json
import sys
from typing import Callable, List, Optional, Tuple

from . import common as c

PRIORITY = {"inbound_email": 0, "meeting": 1, "note": 2, "call": 3, "outbound_email": 4}
CAP_TOTAL = 12000
CAP_EVENT = 3000


def classify(ev: dict) -> Tuple[str, str]:
    t = ev.get("type")
    if t == "email":
        if ev.get("direction") == "INCOMING_EMAIL":
            return "inbound_email", "school"
        return "outbound_email", "wellio"
    return t or "unknown", "wellio"


def select_segments(events: List[dict], cap_total: int = CAP_TOTAL, cap_event: int = CAP_EVENT) -> List[dict]:
    usable = []
    for ev in events:
        kind, speaker = classify(ev)
        if kind not in PRIORITY:
            continue
        body = (ev.get("body") or "").strip()
        if not body or body == "None" or str(ev.get("auto_reply")) == "True":
            continue
        usable.append((PRIORITY[kind], ev.get("ts") or "", kind, speaker, ev, body))
    usable.sort(key=lambda x: (x[0], x[1]))
    out, total = [], 0
    for _, ts, kind, speaker, ev, body in usable:
        text = body[:cap_event]
        if total + len(text) > cap_total:
            break
        out.append({"event_id": str(ev.get("id")), "ts": ts, "kind": kind, "speaker": speaker, "text": text})
        total += len(text)
    out.sort(key=lambda s: s["ts"])
    return out


def default_load_history(company_id: str) -> Optional[dict]:
    p = c.HISTORY / f"{company_id}.json"
    if not p.exists():
        return None
    with p.open() as f:
        return json.load(f)


def build_record(deal: dict, companies: dict, load_history: Callable[[str], Optional[dict]]) -> Optional[dict]:
    start, end = c.early_window(deal)
    cids = c.ids(deal.get("company_ids"))
    events = []
    for cid in cids:
        h = load_history(cid)
        if not h:
            continue
        for ev in h.get("timeline", []):
            t = c.parse_ts(ev.get("ts"))
            if t is None or start is None or end is None:
                continue
            if start <= t <= end:
                events.append(ev)
    segs = select_segments(events)
    created, closed = c.parse_ts(deal.get("createdate")), c.parse_ts(deal.get("closedate"))
    amount_raw = deal.get("amount")
    try:
        amount = float(amount_raw) if amount_raw not in (None, "None", "") else None
    except ValueError:
        amount = None
    first_company = companies.get(cids[0], {}) if cids else {}
    return {
        "deal_id": str(deal.get("id")),
        "dealname": deal.get("dealname") or "",
        "outcome": c.outcome(deal),
        "amount": amount,
        "createdate": deal.get("createdate") or "",
        "closedate": deal.get("closedate") or "",
        "cycle_days": (closed - created).days if created and closed else None,
        "source": deal.get("hs_analytics_source") or "",
        "campaign": deal.get("campaign_source") or "",
        "dealtype": deal.get("dealtype") or "",
        "segment": c.segment(first_company),
        "state": first_company.get("state") or "",
        "company_ids": cids,
        "company_names": [companies.get(x, {}).get("name") for x in cids if companies.get(x, {}).get("name")],
        "window_start": start.isoformat() if start else "",
        "window_end": end.isoformat() if end else "",
        "n_events": len(segs),
        "chars": sum(len(s["text"]) for s in segs),
        "segments": segs,
    }


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="Assemble early-conversation text per AU new-business deal.")
    ap.add_argument("--include-open", action="store_true", help="also include open deals (default closed only)")
    ap.add_argument("--out", default=str(c.OUT / "deals_early_text.jsonl"))
    args = ap.parse_args(argv)
    deals = c.read_jsonl(c.FUNNEL / "deals.jsonl")
    companies = c.load_companies()
    recs, skipped = [], {"pipeline": 0, "dealtype": 0, "open": 0}
    for d in deals:
        if str(d.get("pipeline")) != c.SALES_PIPELINE_AU:
            skipped["pipeline"] += 1
            continue
        if not c.is_new_business(d):
            skipped["dealtype"] += 1
            continue
        if c.outcome(d) == "open" and not args.include_open:
            skipped["open"] += 1
            continue
        recs.append(build_record(d, companies, default_load_history))
    c.write_jsonl(args.out, recs)
    with_text = sum(1 for r in recs if r["chars"] > 200)
    print(f"deals in: {len(deals)}  records: {len(recs)}  with >200 chars: {with_text}  skipped: {skipped}")
    print(f"WROTE {args.out}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_assemble.py -q 2>&1 | tail -3
```

Expected: `5 passed`.

- [ ] **Step 5: Run stage 1 on the real cache**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.assemble_early_text
```

Expected output shape: `deals in: 986  records: ~800  with >200 chars: ~700  skipped: {...}` and `WROTE .../outputs/customer_truth/deals_early_text.jsonl`. Record the exact three numbers; Task 7 writes them into `method.md`.

- [ ] **Step 6: Commit**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: stage 1 assembles capped, speaker-labelled early text per deal

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 5: `tag_deals.py` (stage 2)

**Files:**
- Create: `~/Documents/GTM Project/scripts/growth_os/tag_deals.py`
- Create: `~/Documents/GTM Project/scripts/growth_os/tests/test_tag.py`

**Interfaces:**
- Consumes: `deals_early_text.jsonl` records from Task 4.
- Produces: `outputs/customer_truth/deals_tagged.jsonl`, one row per deal:
  ```
  {"deal_id": str, "codebook_version": "1.0", "model": str, "tagged_at": iso,
   "tags": [{"tag": str, "speaker": "school"|"wellio", "quote": str, "event_id": str}],
   "other": [str], "error": str|None}
  ```
  Functions: `TAGS: List[str]`, `CODEBOOK_VERSION`, `build_prompt() -> str`, `record_to_input(rec) -> str`, `parse_json(raw) -> dict`, `run_claude(prompt, stdin_text, model) -> str`, `tag_record(rec, runner=run_claude, model="sonnet") -> dict`, `main(argv)`.

- [ ] **Step 1: Write the failing tests**

`scripts/growth_os/tests/test_tag.py`:
```python
from __future__ import annotations
import json
from scripts.growth_os import tag_deals as t


def test_tags_match_codebook_v1():
    assert t.CODEBOOK_VERSION == "1.0"
    assert "oversight_visibility" in t.TAGS and "no_prep" in t.TAGS and "social_media_ban" in t.TAGS
    assert len(t.TAGS) == 18


def test_parse_json_strips_fences_and_prose():
    raw = "Here you go:\n```json\n{\"tags\": [], \"other\": []}\n```"
    assert t.parse_json(raw) == {"tags": [], "other": []}
    assert t.parse_json('{"tags":[{"tag":"no_prep"}],"other":[]}')["tags"][0]["tag"] == "no_prep"


def test_parse_json_raises_on_garbage():
    try:
        t.parse_json("not json at all")
    except ValueError:
        return
    raise AssertionError("expected ValueError")


def rec(segments):
    return {"deal_id": "d1", "outcome": "won", "segments": segments}


def test_tag_record_validates_and_normalises():
    fake_out = json.dumps({"tags": [
        {"tag": "oversight_visibility", "speaker": "school", "quote": "we need to see who has done what", "event_id": "e1"},
        {"tag": "made_up_tag", "speaker": "school", "quote": "x", "event_id": "e1"},
        {"tag": "no_prep", "speaker": "robot", "quote": "y", "event_id": "e2"}],
        "other": ["wants Indigenous perspectives content"]})
    calls = []

    def runner(prompt, stdin_text, model):
        calls.append((prompt, stdin_text, model))
        return fake_out

    out = t.tag_record(rec([{"event_id": "e1", "ts": "", "kind": "inbound_email", "speaker": "school", "text": "..."}]),
                       runner=runner, model="sonnet")
    assert out["deal_id"] == "d1" and out["codebook_version"] == "1.0" and out["model"] == "sonnet"
    assert out["error"] is None
    assert [x["tag"] for x in out["tags"]] == ["oversight_visibility", "no_prep"]
    assert out["tags"][1]["speaker"] == "wellio"
    assert out["other"] == ["wants Indigenous perspectives content"]
    assert "d1" in calls[0][1] and "oversight_visibility" in calls[0][0]


def test_tag_record_records_error_after_retry():
    n = {"calls": 0}

    def bad_runner(prompt, stdin_text, model):
        n["calls"] += 1
        return "nope"

    out = t.tag_record(rec([{"event_id": "e1", "ts": "", "kind": "note", "speaker": "wellio", "text": "x"}]), runner=bad_runner)
    assert out["tags"] == [] and out["error"] and n["calls"] == 2


def test_tag_record_skips_claude_when_no_text():
    def boom(prompt, stdin_text, model):
        raise AssertionError("should not be called")

    out = t.tag_record(rec([]), runner=boom)
    assert out["tags"] == [] and out["error"] == "no_text"
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_tag.py -q 2>&1 | tail -3
```

Expected: import error on `tag_deals`.

- [ ] **Step 3: Write `tag_deals.py`**

```python
from __future__ import annotations
"""Stage 2 of the customer-truth pass: tag each deal's early text against the codebook using
headless Claude Code. Resumable: already-tagged deal ids are skipped unless --force."""
import argparse
import datetime as dt
import json
import re
import subprocess
import sys
from typing import Callable, List, Optional

from . import common as c

CODEBOOK_VERSION = "1.0"
TAGS = [
    # leader
    "oversight_visibility", "trackable_outcomes", "standardised_program", "board_reporting",
    "staff_wellbeing_retention", "attendance_behaviour", "parent_community",
    # teacher
    "no_prep", "evidence_based", "curriculum_mapped", "time_saved", "student_engagement",
    # commercial and situational
    "price_budget", "timing_term", "competitor_named", "compliance_requirement",
    "pilot_trial_request", "social_media_ban",
]
CLAUDE_BIN = "/Users/cleonard1998/.local/bin/claude"


def build_prompt() -> str:
    return (
        "You are tagging a sales conversation between Wellio (a school wellbeing curriculum platform) "
        "and an Australian school. The input on stdin is JSON with deal_id and segments; each segment has "
        "speaker 'school' (the school's own words) or 'wellio' (a Wellio rep's email or notes).\n\n"
        "Apply tags ONLY from this list: " + ", ".join(TAGS) + ".\n"
        "A tag with speaker 'school' means the school raised or explicitly agreed to the idea in their own words. "
        "A tag with speaker 'wellio' means only Wellio raised it. Rep meeting notes that report what the school "
        "said count as speaker 'school' when the note clearly attributes it to the school.\n"
        "For each tag give a verbatim quote of at most 200 characters from the segment and its event_id.\n"
        "Put anything notable that fits no tag into 'other' as short phrases.\n\n"
        "Return JSON only, no prose, exactly this shape:\n"
        '{"tags":[{"tag":"...","speaker":"school|wellio","quote":"...","event_id":"..."}],"other":["..."]}'
    )


def record_to_input(rec: dict) -> str:
    return json.dumps({"deal_id": rec["deal_id"],
                       "segments": [{"event_id": s["event_id"], "speaker": s["speaker"], "kind": s["kind"], "text": s["text"]}
                                    for s in rec.get("segments", [])]}, ensure_ascii=False)


def parse_json(raw: str) -> dict:
    m = re.search(r"\{.*\}", raw, flags=re.S)
    if not m:
        raise ValueError("no JSON object in model output")
    try:
        return json.loads(m.group(0))
    except json.JSONDecodeError as e:
        raise ValueError(f"bad JSON: {e}")


def run_claude(prompt: str, stdin_text: str, model: str) -> str:
    p = subprocess.run([CLAUDE_BIN, "-p", prompt, "--output-format", "json", "--model", model],
                       input=stdin_text, capture_output=True, text=True, timeout=180)
    if p.returncode != 0:
        raise RuntimeError(f"claude exit {p.returncode}: {p.stderr[:300]}")
    env = json.loads(p.stdout)
    return env.get("result", "")


def tag_record(rec: dict, runner: Callable[[str, str, str], str] = run_claude, model: str = "sonnet") -> dict:
    base = {"deal_id": rec["deal_id"], "codebook_version": CODEBOOK_VERSION, "model": model,
            "tagged_at": dt.datetime.utcnow().isoformat(timespec="seconds"), "tags": [], "other": [], "error": None}
    if not rec.get("segments"):
        base["error"] = "no_text"
        return base
    prompt, stdin_text = build_prompt(), record_to_input(rec)
    last_err = None
    for _ in range(2):
        try:
            data = parse_json(runner(prompt, stdin_text, model))
            tags = []
            for x in data.get("tags", []):
                if x.get("tag") not in TAGS:
                    continue
                tags.append({"tag": x["tag"],
                             "speaker": "school" if x.get("speaker") == "school" else "wellio",
                             "quote": str(x.get("quote") or "")[:200],
                             "event_id": str(x.get("event_id") or "")})
            base["tags"] = tags
            base["other"] = [str(o)[:200] for o in data.get("other", []) if o]
            return base
        except (ValueError, RuntimeError, json.JSONDecodeError, subprocess.TimeoutExpired) as e:
            last_err = str(e)[:300]
    base["error"] = last_err or "unknown"
    return base


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="Tag early text with headless Claude. Resumable.")
    ap.add_argument("--inp", default=str(c.OUT / "deals_early_text.jsonl"))
    ap.add_argument("--out", default=str(c.OUT / "deals_tagged.jsonl"))
    ap.add_argument("--model", default="sonnet")
    ap.add_argument("--limit", type=int, default=0, help="tag at most N untagged records (0 = all)")
    ap.add_argument("--force", action="store_true", help="retag even if already tagged")
    args = ap.parse_args(argv)
    recs = c.read_jsonl(args.inp)
    done = {r["deal_id"] for r in c.read_jsonl(args.out) if not r.get("error") or r.get("error") == "no_text"}
    todo = [r for r in recs if args.force or r["deal_id"] not in done]
    if args.limit:
        todo = todo[: args.limit]
    print(f"records: {len(recs)}  already tagged: {len(done)}  to tag now: {len(todo)}  model: {args.model}", flush=True)
    for i, r in enumerate(todo, 1):
        row = tag_record(r, model=args.model)
        c.append_jsonl(args.out, [row])
        flag = f"ERROR {row['error']}" if row["error"] and row["error"] != "no_text" else f"{len(row['tags'])} tags"
        print(f"  {i}/{len(todo)} {r['deal_id']} {r['outcome']} {flag}", flush=True)
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_tag.py -q 2>&1 | tail -3
```

Expected: `6 passed`.

- [ ] **Step 5: Dry-run on five real deals and inspect**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.tag_deals --limit 5 --model sonnet && python3 -c "
from scripts.growth_os import common as c
for r in c.read_jsonl(c.OUT/'deals_tagged.jsonl'):
    print(r['deal_id'], r['error'], [(t['tag'], t['speaker'], t['quote'][:60]) for t in r['tags']], r['other'][:2])"
```

Expected: five rows, no `error` other than possibly `no_text`, quotes that visibly come from the text. If quotes are paraphrased rather than verbatim, tighten the prompt line about verbatim quotes and re-run with `--force --limit 5`.

- [ ] **Step 6: Commit**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: stage 2 tags early text with headless Claude, resumable, validated against codebook v1

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 6: `join_outcomes.py` (stage 3)

**Files:**
- Create: `~/Documents/GTM Project/scripts/growth_os/join_outcomes.py`
- Create: `~/Documents/GTM Project/scripts/growth_os/tests/test_join.py`

**Interfaces:**
- Consumes: early records (Task 4) and tagged rows (Task 5).
- Produces: `outputs/customer_truth/summary.json` and `summary.csv`. Summary shape:
  ```
  {"generated_at": iso, "codebook_version": str, "min_n": 20,
   "population": {"deals": int, "won": int, "lost": int, "with_text": int, "tagged": int, "errors": int},
   "base_rate": float,
   "tags": {tag: {"n_present": int, "won_present": int, "rate_present": float|None,
                  "n_absent": int, "won_absent": int, "rate_absent": float|None,
                  "lift_pts": float|None, "median_amount_won": float|None, "median_cycle_days": float|None,
                  "thin": bool, "by_segment": {seg: {"n": int, "rate": float|None}}, "by_source": {...}}},
   "other_sample": [str]}
  ```
  Functions: `school_tags(tagged_row) -> set`, `summarise(early, tagged, min_n=20) -> dict`, `to_csv_rows(summary) -> List[List]`, `main(argv)`.

- [ ] **Step 1: Write the failing tests**

`scripts/growth_os/tests/test_join.py`:
```python
from __future__ import annotations
from scripts.growth_os import join_outcomes as j


def early(deal_id, outcome, amount=1000.0, cycle=30, seg="Catholic", source="OFFLINE", chars=500):
    return {"deal_id": deal_id, "outcome": outcome, "amount": amount, "cycle_days": cycle,
            "segment": seg, "source": source, "chars": chars}


def tagged(deal_id, school=(), wellio=(), error=None, other=()):
    tags = [{"tag": t, "speaker": "school", "quote": "q", "event_id": "e"} for t in school]
    tags += [{"tag": t, "speaker": "wellio", "quote": "q", "event_id": "e"} for t in wellio]
    return {"deal_id": deal_id, "codebook_version": "1.0", "tags": tags, "other": list(other), "error": error}


def test_school_tags_excludes_wellio_voice():
    assert j.school_tags(tagged("d", school=["no_prep"], wellio=["price_budget"])) == {"no_prep"}


def test_summarise_rates_lift_and_thin():
    E = [early("w1", "won"), early("w2", "won"), early("l1", "lost"), early("l2", "lost")]
    T = [tagged("w1", school=["no_prep"]), tagged("w2", school=["no_prep"]),
         tagged("l1", school=["price_budget"]), tagged("l2", school=[])]
    s = j.summarise(E, T, min_n=2)
    assert s["population"] == {"deals": 4, "won": 2, "lost": 2, "with_text": 4, "tagged": 4, "errors": 0}
    assert s["base_rate"] == 0.5
    np = s["tags"]["no_prep"]
    assert np["n_present"] == 2 and np["won_present"] == 2 and np["rate_present"] == 1.0
    assert np["n_absent"] == 2 and np["rate_absent"] == 0.0 and np["lift_pts"] == 100.0
    assert np["thin"] is False
    assert np["median_amount_won"] == 1000.0 and np["median_cycle_days"] == 30
    pb = s["tags"]["price_budget"]
    assert pb["n_present"] == 1 and pb["rate_present"] == 0.0 and pb["thin"] is True
    assert s["tags"]["board_reporting"]["n_present"] == 0 and s["tags"]["board_reporting"]["rate_present"] is None


def test_summarise_breakdowns_only_when_not_thin():
    E = [early(f"w{i}", "won", seg="Catholic" if i % 2 else "Government") for i in range(3)] + [early("l", "lost")]
    T = [tagged(f"w{i}", school=["no_prep"]) for i in range(3)] + [tagged("l", school=["no_prep"])]
    s = j.summarise(E, T, min_n=3)
    assert set(s["tags"]["no_prep"]["by_segment"]) == {"Catholic", "Government"}
    s2 = j.summarise(E, T, min_n=10)
    assert s2["tags"]["no_prep"]["by_segment"] == {}


def test_summarise_counts_errors_and_other_sample():
    E = [early("a", "won"), early("b", "lost", chars=0)]
    T = [tagged("a", school=[], other=["wants Indigenous content"]), tagged("b", error="no_text")]
    s = j.summarise(E, T)
    assert s["population"]["with_text"] == 1 and s["population"]["errors"] == 0 and s["population"]["tagged"] == 2
    assert s["other_sample"] == ["wants Indigenous content"]


def test_to_csv_rows_header_and_order():
    E = [early("a", "won")]
    T = [tagged("a", school=["no_prep"])]
    rows = j.to_csv_rows(j.summarise(E, T))
    assert rows[0] == ["tag", "n_present", "won_present", "rate_present", "n_absent", "rate_absent",
                       "lift_pts", "median_amount_won", "median_cycle_days", "thin"]
    assert rows[1][0] == "no_prep"
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_join.py -q 2>&1 | tail -3
```

Expected: import error on `join_outcomes`.

- [ ] **Step 3: Write `join_outcomes.py`**

```python
from __future__ import annotations
"""Stage 3 of the customer-truth pass: join school-voiced tags to deal outcomes."""
import argparse
import csv
import datetime as dt
import statistics
import sys
from typing import Dict, List, Optional

from . import common as c
from .tag_deals import CODEBOOK_VERSION, TAGS

MIN_N = 20


def school_tags(row: dict) -> set:
    return {t["tag"] for t in row.get("tags", []) if t.get("speaker") == "school"}


def _rate(won: int, n: int) -> Optional[float]:
    return round(won / n, 4) if n else None


def _median(xs: List[float]) -> Optional[float]:
    xs = [x for x in xs if x is not None]
    return round(statistics.median(xs), 2) if xs else None


def summarise(early: List[dict], tagged: List[dict], min_n: int = MIN_N) -> dict:
    by_id = {r["deal_id"]: r for r in early if r.get("outcome") in ("won", "lost")}
    tag_by_id: Dict[str, dict] = {t["deal_id"]: t for t in tagged if t["deal_id"] in by_id}
    won_total = sum(1 for r in by_id.values() if r["outcome"] == "won")
    pop = {"deals": len(by_id), "won": won_total, "lost": len(by_id) - won_total,
           "with_text": sum(1 for r in by_id.values() if (r.get("chars") or 0) > 200),
           "tagged": len(tag_by_id),
           "errors": sum(1 for t in tag_by_id.values() if t.get("error") and t["error"] != "no_text")}
    out: Dict[str, dict] = {}
    for tag in TAGS:
        present = [by_id[d] for d, t in tag_by_id.items() if tag in school_tags(t)]
        present_ids = {r["deal_id"] for r in present}
        absent = [r for d, r in by_id.items() if d in tag_by_id and d not in present_ids]
        wp = sum(1 for r in present if r["outcome"] == "won")
        wa = sum(1 for r in absent if r["outcome"] == "won")
        rp, ra = _rate(wp, len(present)), _rate(wa, len(absent))
        thin = len(present) < min_n
        entry = {"n_present": len(present), "won_present": wp, "rate_present": rp,
                 "n_absent": len(absent), "won_absent": wa, "rate_absent": ra,
                 "lift_pts": round((rp - ra) * 100, 1) if rp is not None and ra is not None else None,
                 "median_amount_won": _median([r.get("amount") for r in present if r["outcome"] == "won"]),
                 "median_cycle_days": _median([r.get("cycle_days") for r in present]),
                 "thin": thin, "by_segment": {}, "by_source": {}}
        if not thin:
            for key, field in (("by_segment", "segment"), ("by_source", "source")):
                groups: Dict[str, List[dict]] = {}
                for r in present:
                    groups.setdefault(r.get(field) or "Unknown", []).append(r)
                entry[key] = {g: {"n": len(rs), "rate": _rate(sum(1 for r in rs if r["outcome"] == "won"), len(rs))}
                              for g, rs in sorted(groups.items())}
        out[tag] = entry
    others = []
    for t in tag_by_id.values():
        others.extend(t.get("other") or [])
    return {"generated_at": dt.datetime.utcnow().isoformat(timespec="seconds"),
            "codebook_version": CODEBOOK_VERSION, "min_n": min_n, "population": pop,
            "base_rate": _rate(won_total, len(by_id)), "tags": out, "other_sample": others[:50]}


def to_csv_rows(summary: dict) -> List[List]:
    rows = [["tag", "n_present", "won_present", "rate_present", "n_absent", "rate_absent",
             "lift_pts", "median_amount_won", "median_cycle_days", "thin"]]
    for tag, e in sorted(summary["tags"].items(), key=lambda kv: -(kv[1]["lift_pts"] or -999)):
        rows.append([tag, e["n_present"], e["won_present"], e["rate_present"], e["n_absent"], e["rate_absent"],
                     e["lift_pts"], e["median_amount_won"], e["median_cycle_days"], e["thin"]])
    return rows


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="Join tags to outcomes.")
    ap.add_argument("--early", default=str(c.OUT / "deals_early_text.jsonl"))
    ap.add_argument("--tagged", default=str(c.OUT / "deals_tagged.jsonl"))
    ap.add_argument("--out-json", default=str(c.OUT / "summary.json"))
    ap.add_argument("--out-csv", default=str(c.OUT / "summary.csv"))
    ap.add_argument("--min-n", type=int, default=MIN_N)
    args = ap.parse_args(argv)
    s = summarise(c.read_jsonl(args.early), c.read_jsonl(args.tagged), args.min_n)
    import json, pathlib
    pathlib.Path(args.out_json).parent.mkdir(parents=True, exist_ok=True)
    pathlib.Path(args.out_json).write_text(json.dumps(s, indent=1))
    with open(args.out_csv, "w", newline="") as f:
        csv.writer(f).writerows(to_csv_rows(s))
    print(f"population {s['population']} base rate {s['base_rate']}")
    for row in to_csv_rows(s)[1:8]:
        print("  ", row)
    print(f"WROTE {args.out_json} and {args.out_csv}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_join.py -q 2>&1 | tail -3
```

Expected: `5 passed`.

- [ ] **Step 5: Commit**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: stage 3 joins school-voiced tags to win rate, amount and cycle with thin flags

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 7: `write_truth.py` (stage 4) and `checks.py`

**Files:**
- Create: `~/Documents/GTM Project/scripts/growth_os/write_truth.py`
- Create: `~/Documents/GTM Project/scripts/growth_os/checks.py`
- Create: `~/Documents/GTM Project/scripts/growth_os/tests/test_truth_and_checks.py`
- Create: `~/Documents/growth-os/customer-truth/method.md`

**Interfaces:**
- Consumes: `summary.json`, `deals_tagged.jsonl`, `deals_early_text.jsonl`, `lead_company_context.json`.
- Produces: `growth-os/customer-truth/what-the-market-is-telling-us.md`; appends a run entry to `growth-os/customer-truth/method.md`.
  Functions in `write_truth.py`: `deidentify(text, names) -> str`, `school_label(rec) -> str` (e.g. "Catholic secondary, NSW"), `pick_quotes(tag, tagged, early_by_id, k=3) -> List[dict]`, `render(summary, quotes_by_tag, meta) -> str`, `main(argv)`.
  Functions in `checks.py`: `check_citations(paths) -> List[str]`, `check_deidentified(paths, names) -> List[str]`, `check_readonly(script_paths) -> List[str]`, `main(argv)` exiting 1 on any finding.

- [ ] **Step 1: Write the failing tests**

`scripts/growth_os/tests/test_truth_and_checks.py`:
```python
from __future__ import annotations
import pathlib
from scripts.growth_os import write_truth as w
from scripts.growth_os import checks as k


def test_deidentify_replaces_names_case_insensitively_and_shortforms():
    txt = "Mater Dei College asked, and mater dei said yes. St John Fisher too."
    out = w.deidentify(txt, ["Mater Dei College", "St John Fisher"])
    assert "Mater Dei" not in out and "mater dei" not in out and "St John Fisher" not in out
    assert out.count("[school]") == 3


def test_school_label():
    assert w.school_label({"segment": "Catholic", "state": "NSW", "dealtype": "newbusiness"}) == "Catholic secondary, NSW"
    assert w.school_label({"segment": "Government", "state": "", "dealtype": "New Business - Primary"}) == "Government primary"


def test_pick_quotes_prefers_won_and_distinct_deals():
    early = {"a": {"deal_id": "a", "outcome": "won", "segment": "Catholic", "state": "VIC", "dealtype": "newbusiness", "company_names": ["Alpha College"]},
             "b": {"deal_id": "b", "outcome": "lost", "segment": "Government", "state": "NSW", "dealtype": "newbusiness", "company_names": []}}
    tagged = [{"deal_id": "a", "tags": [{"tag": "no_prep", "speaker": "school", "quote": "Alpha College has no time to prep", "event_id": "e"},
                                        {"tag": "no_prep", "speaker": "school", "quote": "second quote same deal", "event_id": "e"}]},
              {"deal_id": "b", "tags": [{"tag": "no_prep", "speaker": "school", "quote": "we cannot prep", "event_id": "e"}]}]
    qs = w.pick_quotes("no_prep", tagged, early, k=3)
    assert [q["outcome"] for q in qs] == ["won", "lost"]
    assert qs[0]["label"] == "Catholic secondary, VIC" and "Alpha College" not in qs[0]["quote"]


def test_render_has_source_lines_and_limits():
    summary = {"generated_at": "2026-09-23T00:00:00", "codebook_version": "1.0", "min_n": 20,
               "population": {"deals": 4, "won": 2, "lost": 2, "with_text": 4, "tagged": 4, "errors": 0},
               "base_rate": 0.5, "other_sample": ["x"],
               "tags": {"no_prep": {"n_present": 25, "won_present": 20, "rate_present": 0.8, "n_absent": 100, "won_absent": 40,
                                    "rate_absent": 0.4, "lift_pts": 40.0, "median_amount_won": 9000.0, "median_cycle_days": 30,
                                    "thin": False, "by_segment": {}, "by_source": {}},
                        "price_budget": {"n_present": 3, "won_present": 0, "rate_present": 0.0, "n_absent": 122, "won_absent": 60,
                                         "rate_absent": 0.49, "lift_pts": -49.0, "median_amount_won": None, "median_cycle_days": 70,
                                         "thin": True, "by_segment": {}, "by_source": {}}}}
    md = w.render(summary, {"no_prep": [{"quote": "no time to prep", "label": "Catholic secondary, VIC", "outcome": "won"}]},
                  {"mode": "cycle zero (retrospective)", "source_line": "Source: test"})
    assert "## What separates winners from losers" in md and "Source:" in md
    assert "thin" in md and "no time to prep" in md and "cycle zero" in md
    assert "—" not in md


def test_check_citations_flags_sections_without_source(tmp_path):
    good = tmp_path / "good.md"; good.write_text("# T\n\n## A\ntext\nSource: x\n\n## B\nmore\nSource: y\n")
    bad = tmp_path / "bad.md"; bad.write_text("# T\n\n## A\ntext\n\n## B\nSource: y\n")
    assert k.check_citations([good]) == []
    assert k.check_citations([bad]) == [f"{bad}: section 'A' has no Source: line"]


def test_check_deidentified_finds_names(tmp_path):
    f = tmp_path / "f.md"; f.write_text("Quote from Mater Dei College here.")
    assert k.check_deidentified([f], ["Mater Dei College", "Nowhere High"]) == [f"{f}: contains 'Mater Dei College'"]


def test_check_readonly_allows_search_and_batch_read_only(tmp_path):
    ok = tmp_path / "ok.py"; ok.write_text('call("POST", f"/crm/v3/objects/{obj}/search", p)\ncall("GET", "/x")\ncall("POST", "/crm/v3/objects/emails/batch/read", p)\n')
    bad = tmp_path / "bad.py"; bad.write_text('call("PATCH", "/crm/v3/objects/deals/1", p)\ncall("POST", "/crm/v3/objects/notes", p)\n')
    assert k.check_readonly([ok]) == []
    out = k.check_readonly([bad])
    assert len(out) == 2 and "PATCH" in out[0] and "/crm/v3/objects/notes" in out[1]
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_truth_and_checks.py -q 2>&1 | tail -3
```

Expected: import errors on `write_truth` and `checks`.

- [ ] **Step 3: Write `write_truth.py`**

```python
from __future__ import annotations
"""Stage 4 of the customer-truth pass: render the de-identified finding into Growth OS and log the run."""
import argparse
import datetime as dt
import json
import pathlib
import re
import sys
from typing import Dict, List, Optional

from . import common as c

STOP = {"college", "school", "high", "secondary", "primary", "catholic", "grammar", "state", "public", "the", "of",
        "st", "saint", "christian", "anglican", "academy", "campus", "senior", "junior"}


def _name_variants(name: str) -> List[str]:
    words = [w for w in re.findall(r"[A-Za-z']+", name)]
    core = [w for w in words if w.lower() not in STOP]
    variants = [name]
    if len(core) >= 2:
        variants.append(" ".join(core))
    return variants


def deidentify(text: str, names: List[str]) -> str:
    out = text
    for name in sorted({n for n in names if n and len(n) > 3}, key=len, reverse=True):
        for v in _name_variants(name):
            if len(v) <= 3:
                continue
            out = re.sub(re.escape(v), "[school]", out, flags=re.I)
    return out


def school_label(rec: dict) -> str:
    level = "primary" if rec.get("dealtype") == "New Business - Primary" else "secondary"
    seg = rec.get("segment") or "Unknown"
    state = rec.get("state") or ""
    return f"{seg} {level}, {state}" if state else f"{seg} {level}"


def pick_quotes(tag: str, tagged: List[dict], early_by_id: Dict[str, dict], k: int = 3) -> List[dict]:
    cands = []
    for row in tagged:
        rec = early_by_id.get(row["deal_id"])
        if not rec:
            continue
        for t in row.get("tags", []):
            if t.get("tag") == tag and t.get("speaker") == "school" and t.get("quote"):
                cands.append((0 if rec["outcome"] == "won" else 1, row["deal_id"], t["quote"], rec))
                break  # one quote per deal
    cands.sort(key=lambda x: (x[0], -len(x[2])))
    out = []
    for _, deal_id, quote, rec in cands[:k]:
        out.append({"quote": deidentify(quote, rec.get("company_names") or []), "label": school_label(rec),
                    "outcome": rec["outcome"], "deal_id": deal_id})
    return out


def _pct(x: Optional[float]) -> str:
    return "n/a" if x is None else f"{x * 100:.0f}%"


def render(summary: dict, quotes_by_tag: Dict[str, List[dict]], meta: dict) -> str:
    pop, tags = summary["population"], summary["tags"]
    ranked = sorted(tags.items(), key=lambda kv: -(kv[1]["lift_pts"] if kv[1]["lift_pts"] is not None else -999))
    strong = [(t, e) for t, e in ranked if not e["thin"] and e["lift_pts"] is not None and abs(e["lift_pts"]) >= 10]
    lines = [
        "# What the market is telling us",
        "",
        f"Generated {summary['generated_at']} UTC. Mode: {meta['mode']}. Codebook {summary['codebook_version']}. Minimum sample {summary['min_n']}.",
        "",
        "## The answer in one paragraph",
        "",
    ]
    if strong:
        top = strong[0]
        lines.append(
            f"Across {pop['deals']} closed AU new-business deals ({pop['won']} won, {pop['lost']} lost, base win rate {_pct(summary['base_rate'])}), "
            f"the hook that most separates winners from losers is **{top[0]}**: {_pct(top[1]['rate_present'])} win rate when the school raised it "
            f"(n={top[1]['n_present']}) against {_pct(top[1]['rate_absent'])} when it did not. "
            f"{len(strong)} tags clear the minimum sample and a 10-point difference; the rest are shown but marked thin.")
    else:
        lines.append(f"Across {pop['deals']} closed deals no tag yet clears the minimum sample with a 10-point difference. Everything below is thin.")
    lines += ["", meta["source_line"], "", "## What separates winners from losers", "",
              "| Tag | Present n | Win rate present | Win rate absent | Lift (pts) | Median won $ | Median cycle days | Note |",
              "|---|---|---|---|---|---|---|---|"]
    for t, e in ranked:
        note = "thin" if e["thin"] else ""
        lines.append(f"| {t} | {e['n_present']} | {_pct(e['rate_present'])} | {_pct(e['rate_absent'])} | "
                     f"{'n/a' if e['lift_pts'] is None else e['lift_pts']} | {'n/a' if e['median_amount_won'] is None else int(e['median_amount_won'])} | "
                     f"{'n/a' if e['median_cycle_days'] is None else e['median_cycle_days']} | {note} |")
    lines += ["", meta["source_line"], "", "## In the schools' words", "", meta["source_line"], ""]
    for t, e in strong[:6]:
        qs = quotes_by_tag.get(t, [])
        if not qs:
            continue
        lines.append(f"### {t}")
        lines.append("")
        for q in qs:
            lines.append(f"- \"{q['quote']}\" ({q['label']}, {q['outcome']})")
        lines += ["", meta["source_line"], ""]
    lines += ["## What did not separate them", ""]
    flat = [t for t, e in ranked if not e["thin"] and e["lift_pts"] is not None and abs(e["lift_pts"]) < 10]
    lines.append(", ".join(flat) if flat else "Nothing yet clears the minimum sample without a difference.")
    lines += ["", meta["source_line"], "", "## Limits", "",
              f"- Population: closed deals only; {pop['with_text']} of {pop['deals']} had more than 200 characters of early text; {pop['tagged']} tagged; {pop['errors']} tagging errors.",
              "- Meeting outcomes are not recorded in HubSpot, so this reads what was written, not whether a meeting happened.",
              "- Loss-reason labels were not used (128 of 214 audited were wrong); the notes behind them were.",
              "- Retrospective tagging by a language model: quotes are verbatim, tags are judgement. Rerun quarterly from raw.",
              "", meta["source_line"], ""]
    if summary.get("other_sample"):
        lines += ["## Other, unsorted (read before the next codebook)", ""] + [f"- {o}" for o in summary["other_sample"][:25]] + ["", meta["source_line"], ""]
    return "\n".join(lines).replace("—", "-")


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="Render the truth file into Growth OS.")
    ap.add_argument("--mode", default="cycle zero (retrospective)")
    ap.add_argument("--out", default=str(c.GROWTH_OS / "customer-truth" / "what-the-market-is-telling-us.md"))
    args = ap.parse_args(argv)
    summary = json.load((c.OUT / "summary.json").open())
    tagged = c.read_jsonl(c.OUT / "deals_tagged.jsonl")
    early_by_id = {r["deal_id"]: r for r in c.read_jsonl(c.OUT / "deals_early_text.jsonl")}
    quotes = {t: pick_quotes(t, tagged, early_by_id) for t in summary["tags"]}
    source_line = ("Source: ~/Documents/GTM Project/outputs/customer_truth/summary.json (stage 3), "
                   "deals_tagged.jsonl (stage 2), deals_early_text.jsonl (stage 1); method in customer-truth/method.md")
    md = render(summary, quotes, {"mode": args.mode, "source_line": source_line})
    out = pathlib.Path(args.out)
    out.parent.mkdir(parents=True, exist_ok=True)
    out.write_text(md)
    method = c.GROWTH_OS / "customer-truth" / "method.md"
    entry = (f"\n- {dt.date.today().isoformat()}: {args.mode}; population {summary['population']}; "
             f"base rate {summary['base_rate']}; codebook {summary['codebook_version']}; portal {c.PROD_PORTAL_ID}.\n")
    with method.open("a") as f:
        f.write(entry)
    print(f"WROTE {out}\nLOGGED run in {method}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Write `checks.py`**

```python
from __future__ import annotations
"""Verification checks for Growth OS. Exit 1 on any finding so the weekly job stops before committing."""
import argparse
import pathlib
import re
import sys
from typing import List

from . import common as c

ALLOWED_POST = re.compile(r"/search\"?$|/batch/read\"?$")
GENERIC_NAMES = {"Primary", "Secondary", "Wellio", "Essentials", "Bundle", "Standard", "Insights", "Renewal", "Upsell"}


def check_citations(paths: List[pathlib.Path]) -> List[str]:
    out = []
    for p in paths:
        text = p.read_text()
        parts = re.split(r"^## ", text, flags=re.M)
        for part in parts[1:]:
            title = part.splitlines()[0].strip()
            if "Source:" not in part:
                out.append(f"{p}: section '{title}' has no Source: line")
    return out


def check_deidentified(paths: List[pathlib.Path], names: List[str]) -> List[str]:
    out = []
    pats = [(n, re.compile(re.escape(n), re.I)) for n in names if n and len(n) > 5 and n.strip() not in GENERIC_NAMES]
    for p in paths:
        text = p.read_text()
        for n, pat in pats:
            if pat.search(text):
                out.append(f"{p}: contains '{n}'")
                break
    return out


def check_readonly(script_paths: List[pathlib.Path]) -> List[str]:
    out = []
    for p in script_paths:
        for i, line in enumerate(p.read_text().splitlines(), 1):
            m = re.search(r"call\(\s*\"(GET|POST|PATCH|PUT|DELETE)\"\s*,\s*f?\"([^\"]+)\"", line)
            if not m:
                continue
            method, path = m.group(1), m.group(2)
            if method in ("PATCH", "PUT", "DELETE"):
                out.append(f"{p}:{i}: {method} not allowed")
            elif method == "POST" and not ALLOWED_POST.search(path):
                out.append(f"{p}:{i}: POST to {path} not allowed (only /search and /batch/read)")
    return out


def founder_facing_files() -> List[pathlib.Path]:
    g = c.GROWTH_OS
    files = [p for p in (g / "customer-truth").rglob("*.md") if p.name != "method.md"]
    files += list((g / "results").rglob("*.md"))
    files.append(g / "README.md")
    return [p for p in files if p.exists()]


def main(argv=None) -> int:
    ap = argparse.ArgumentParser()
    ap.add_argument("which", nargs="*", default=["citations", "deidentified", "readonly"])
    args = ap.parse_args(argv)
    findings: List[str] = []
    if "citations" in args.which:
        targets = [c.GROWTH_OS / "customer-truth" / "what-the-market-is-telling-us.md"]
        targets += list((c.GROWTH_OS / "markets").glob("*.md"))
        targets += [c.GROWTH_OS / "results" / "baseline-2026-09.md"]
        findings += check_citations([p for p in targets if p.exists() and p.name != "README.md"])
    if "deidentified" in args.which:
        names = [v.get("name") for v in c.load_companies().values() if v.get("name")]
        names += [r["dealname"].split(" - ")[0] for r in c.read_jsonl(c.FUNNEL / "deals.jsonl") if r.get("dealname")]
        findings += check_deidentified(founder_facing_files(), sorted(set(names)))
    if "readonly" in args.which:
        findings += check_readonly(sorted((c.GTM / "scripts" / "growth_os").glob("*.py")))
    for f in findings:
        print("FAIL", f)
    print("OK" if not findings else f"{len(findings)} findings")
    return 1 if findings else 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 5: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests -q 2>&1 | tail -3
```

Expected: `31 passed` (8 + 5 + 6 + 5 + 7).

- [ ] **Step 6: Write `customer-truth/method.md`**

```markdown
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

## Known limits
Meeting outcomes unrecorded (70 percent of past discovery meetings still SCHEDULED); a third of meetings untyped; call dispositions only from Feb 2026; loss labels unreliable (128 of 214 audited), notes used instead. Source: `~/Documents/GTM Project/docs/revops/2026-09-03-sdr-funnel-leak-analysis.md`.

## Run log
```

The run log is appended by `write_truth.py`.

- [ ] **Step 7: Commit both repos**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: stage 4 renders the de-identified truth file; checks for citations, de-identification and read-only

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
cd ~/Documents/growth-os && git add -A && git commit -m "customer-truth: method file for the pass

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 8: Migrate existing findings into markets and results

**Files:**
- Create: `~/Documents/growth-os/results/baseline-2026-09.md`
- Create: `~/Documents/growth-os/results/metric-definitions.md`
- Create: `~/Documents/growth-os/markets/au.md`, `uk.md`, `europe.md`, `canada.md`, `international.md`, `need-data-sources.md`

**Interfaces:**
- Consumes: the named GTM Project docs. Every `## ` section ends with a `Source:` line so `checks.py citations` passes.

- [ ] **Step 1: Write `results/baseline-2026-09.md`**

Pull each figure from the named source and keep the wording of the source. Skeleton with the figures already verified:

```markdown
# Baseline, September 2026

Every number below is a recorded deal amount in HubSpot production 20058914, not recognised ARR, unless stated.

## New business, AU

| Period | Deals won | Recorded amount |
|---|---|---|
| 1 Jan to 3 Sep 2025 | 82 | $693k |
| 1 Jan to 3 Sep 2026 | 85 | $629k |
| Full year 2025 | 284 | $2.35m |
| 2025 won on or after 4 Sep | 204 (70%) | |

Source: ~/Documents/GTM Project/docs/gtm-strategy-2026-09/01-hubspot-findings.md; 10-third-critique-response.md (corrected 2025 figures, re-verified live 7 Sep 2026).

## Funnel shape, AU meeting-booked leads (n = 1,424)

Booked 100%, qualified 60%, cancelled 15%, unsuccessful 12%. Of qualified deals: won 36%, lost 44%, open 19%. A booked meeting becomes a won deal 22% of the time.

Source: ~/Documents/GTM Project/docs/revops/2026-09-03-sdr-funnel-leak-analysis.md, section 1.1.

## Penetration

Catholic combined 30%, Catholic secondary 21%, government secondary 15%, primary 0.4% (24 of 6,156).

Source: ~/Documents/GTM Project/docs/gtm-strategy-2026-09/01-hubspot-findings.md.

## Sources of new business

Free Preview and Contact Sales: $196k, 18 deals, $10.9k average (2026). Referrals average $16k, 13 records ever.

Source: ~/Documents/GTM Project/docs/gtm-strategy-2026-09/01-hubspot-findings.md.

## What has been built and dry-run

- Sales Signals pilot: 20 schools, 10 drafts, zero HubSpot writes, awaiting Sales Manager sign-off.
- Primary attach spine: 202 K-12 schools, 201 scraped, junior POC named at 99.

Source: ~/Documents/GTM Project/docs/revops/2026-09-05-sdr-outreach-pilot-assessment.md; outputs/primary_attach_2026-09-05/.

## Recording gaps that limit measurement

70% of past discovery meetings still read SCHEDULED; 34% of meetings have no type; call booking dispositions exist only from Feb 2026.

Source: ~/Documents/GTM Project/docs/revops/2026-09-03-sdr-funnel-leak-analysis.md, section 0.
```

- [ ] **Step 2: Write `results/metric-definitions.md`**

```markdown
# Metric definitions

| Metric | Definition | HubSpot carrier | Measurable today? |
|---|---|---|---|
| Qualified reply | A human reply from a school contact to an outbound email or sequence that is not an auto-reply or unsubscribe | Email engagement, direction INCOMING_EMAIL, `hs_sequence_id` on the parent thread | Partly: replies yes, link to sequence only where enrolled via HubSpot |
| Meeting booked | A lead reaching a meeting-booked stage | Lead `date_first_entered_meeting_booked` | Yes |
| Meeting held | A booked meeting that took place | Meeting `hs_meeting_outcome` | No: outcomes not recorded (70% still SCHEDULED) |
| Pipeline created | A Sales Pipeline AU deal created from a lead | Deal `createdate`, `lead_ids` | Yes |
| Pipeline won | Deal `hs_is_closed_won = true` | Deal | Yes |
| Hook attribution | Which hook a school responded to | Not present. Requires `hook_id`, `persona`, `variant` properties on enrolment or deal | No: prospective stamp waits for the decision |

Source: ~/Documents/GTM Project/docs/revops/2026-09-03-sdr-funnel-leak-analysis.md, section 0; outputs/sdr_funnel field lists.
```

- [ ] **Step 3: Write the market files**

`markets/au.md`, `markets/uk.md` from known facts; `europe.md`, `canada.md`, `international.md` as honest stubs; `need-data-sources.md` as the dataset table. Every file uses these headings: Footprint, Buyer and signer, Average deal and what drives it, Compliance or discretionary, Statutory hook, Public need data, Entry route, What we do not know. Each section ends with a `Source:` line. For AU the sources are brainstorm Q13, 01-hubspot-findings.md and 2026-08-20-sam-tof-synthesis.md (SAM 720, clean bulk 208, 683 swing group). For UK: brainstorm Q13 (about $2,000 average, PSHE requirement, Outreach for outbound, 300+ schools). For Europe and Canada: "Footprint: zero schools. Source: brainstorm Q8." and every other section "Not yet known. Source: brainstorm 2026-09-15, open flag."

`need-data-sources.md`:
```markdown
# Need data sources

| Market | Dataset | Measures | Granularity | Access | Verified |
|---|---|---|---|---|---|
| SA | Wellbeing and Engagement Collection | student wellbeing and engagement | school | public summary, request for detail | unverified |
| NSW | Tell Them From Me | student engagement and wellbeing | school, via annual reports | annual report PDFs | unverified |
| VIC | Attitudes to School Survey | student attitudes | school, via annual reports | annual report PDFs | unverified |
| AU | My School | ICSEA, funding, staffing, enrolments | school | public site | unverified |
| England | Ofsted personal development judgement | inspection grade | school | public | unverified |
| England | Free school meal percentage | disadvantage | school | DfE public data | unverified |
| England | Mental Health Support Team coverage | service coverage | area | NHS public | unverified |
| Ontario | School climate surveys | climate and wellbeing | board or school | varies | unverified |
| BC | Student Learning Survey | wellbeing and belonging | school | public | unverified |
| All | Wellio benchmark, 210,000 students | check-in scores | school | internal | verified |

Source: brainstorm 2026-09-15 Q4; ~/Documents/GTM Project/docs/gtm-strategy-2026-09/03-market-research-brief.md. Each row is marked verified only after someone opens the dataset and records the URL.
```

- [ ] **Step 4: Run the citation check and commit**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.checks citations
cd ~/Documents/growth-os && git add -A && git commit -m "results: baseline and metric definitions; markets: AU, UK, stubs and need-data sources, all sourced

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

Expected: `OK` from the check.

---

### Task 9: Private GitHub remote

**Files:** none in repo.

- [ ] **Step 1: Install gh or use the web UI**

Either:
```bash
brew install gh && gh auth login
```
or create an empty private repository named `growth-os` at github.com and copy its SSH or HTTPS URL.

- [ ] **Step 2: Add the remote and push**

With gh:
```bash
cd ~/Documents/growth-os && gh repo create growth-os --private --source=. --remote=origin --push
```
Without gh:
```bash
cd ~/Documents/growth-os && git remote add origin <URL> && git push -u origin main
```

Expected: `git remote -v` shows origin; the repository page shows Private.

- [ ] **Step 3: Record it**

Append to `decisions/2026-09-16-standalone-repo.md`: `Remote: private GitHub repository added <date>. Access: Caelum only until the Head of Marketing review.` Commit and push.

---

### Task 10: Friday sponsor review (week 1 gate)

Not a code task. Checklist:

- [ ] Time a cold read of `README.md` and `results/README.md`. Record the minutes in `decisions/README.md` under a "Week 1 gate" heading.
- [ ] Run `python3 -m scripts.growth_os.checks` from the GTM Project root. Expected `OK`.
- [ ] Show the Head of Marketing the repo for 15 minutes: the loop in the README, the codebook, the baseline. Record his reactions as a decisions entry `2026-09-19-sponsor-review-1.md`.
- [ ] Commit and push.

---

## Week 2

### Task 11: Cycle zero end to end

**Files:**
- Modify: `~/Documents/growth-os/customer-truth/method.md` (run log appended by the script)
- Create (generated): `~/Documents/growth-os/customer-truth/what-the-market-is-telling-us.md`

- [ ] **Step 1: Tag everything**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.tag_deals --model sonnet 2>&1 | tee outputs/customer_truth/tag_run_$(date +%F).log | tail -5
```

Expected: about 800 records processed over one to two hours; the last line shows the count. Re-run the same command if interrupted; it resumes. Cost expectation: a few tens of dollars on Sonnet.

- [ ] **Step 2: Join and inspect**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.join_outcomes
```

Expected: the population line shows about 800 deals with about 700 with text, and the top seven rows of the ranked table. Read the `other_sample` in `summary.json`. If a theme appears five or more times, add it to the codebook as version 1.1 (edit `hook-codebook.md` change log and `TAGS` in `tag_deals.py`, bump `CODEBOOK_VERSION`), then retag with `--force` and rejoin. Record that decision.

- [ ] **Step 3: Render, check, read**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.write_truth --mode "cycle zero (retrospective)" && python3 -m scripts.growth_os.checks
```

Expected: `OK`. Open the truth file and read every quote. Any quote that names a school, a person, or contains student information is fixed in `deidentify` (add the pattern) and re-rendered, never edited by hand.

- [ ] **Step 4: Rerun check in a fresh session**

Open a new Claude Code session in `~/Documents/GTM Project`, paste only: "Follow ~/Documents/growth-os/customer-truth/method.md stages 3 and 4 and report the top five tags by lift." Expected: the same five tags in the same order as your run. Record the result in `method.md` under the run log.

- [ ] **Step 5: Commit and push**

```bash
cd ~/Documents/growth-os && git add -A && git commit -m "customer-truth: cycle zero truth file and run log

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>" && git push
```

---

### Task 12: `pull_delta.py` with tests

**Files:**
- Create: `~/Documents/GTM Project/scripts/growth_os/pull_delta.py`
- Create: `~/Documents/GTM Project/scripts/growth_os/tests/test_pull_delta.py`

**Interfaces:**
- Consumes: `call`, `search_all`, `assoc` from `scripts/hubspot/funnel/pull_funnel.py` (imported at runtime; the module exits at import if the token is missing, so tests inject fakes).
- Produces: `outputs/growth_os/delta/<week>/{deals,meetings,calls,notes,emails}.jsonl` and `manifest.json`. Functions: `week_dir(week) -> Path`, `since_date(days=8) -> date`, `one_window(since, end=None)`, `pull(week, since, api) -> dict` where `api` has `.call`, `.search_all`, `.assoc`; `main(argv)`.
  Engagement rows carry `company_ids`, `deal_ids` and, for emails, `hs_email_text`.

- [ ] **Step 1: Write the failing tests**

`scripts/growth_os/tests/test_pull_delta.py`:
```python
from __future__ import annotations
import datetime as dt
import json
from scripts.growth_os import pull_delta as pdl


class FakeApi:
    def __init__(self):
        self.calls = []

    def call(self, method, path, payload=None):
        self.calls.append((method, path))
        if path == "/account-info/v3/details":
            return 200, {"portalId": 20058914}
        if path.endswith("/emails/batch/read"):
            return 200, {"results": [{"id": i["id"], "properties": {"hs_email_text": f"body {i['id']}"}} for i in payload["inputs"]]}
        raise AssertionError(path)

    def search_all(self, obj, props, base_filters, date_prop, since, sort_prop, windows=None):
        assert date_prop == "hs_lastmodifieddate"
        if obj == "deals":
            yield {"id": "d1", "properties": {"dealname": "X", "pipeline": "41942802"}}
        elif obj == "emails":
            yield {"id": "e1", "properties": {"hs_email_subject": "s", "hs_email_direction": "INCOMING_EMAIL", "hs_timestamp": "2026-09-22T00:00:00Z"}}
        else:
            return

    def assoc(self, from_obj, to_obj, ids):
        return {i: ["c1"] if to_obj == "companies" else ["d1"] for i in ids}


def test_one_window_is_single_span():
    w = list(pdl.one_window(dt.date(2026, 9, 20), dt.date(2026, 9, 28)))
    assert w == [(dt.date(2026, 9, 20), dt.date(2026, 9, 28))]


def test_pull_writes_files_and_manifest(tmp_path, monkeypatch):
    monkeypatch.setattr(pdl.c, "GOS_OUT", tmp_path)
    api = FakeApi()
    m = pdl.pull("2026-W39", dt.date(2026, 9, 20), api)
    d = tmp_path / "delta" / "2026-W39"
    deals = [json.loads(l) for l in (d / "deals.jsonl").open()]
    emails = [json.loads(l) for l in (d / "emails.jsonl").open()]
    assert deals[0]["id"] == "d1" and deals[0]["company_ids"] == ["c1"]
    assert emails[0]["hs_email_text"] == "body e1" and emails[0]["deal_ids"] == ["d1"]
    assert m["portal"] == 20058914 and m["counts"]["deals"] == 1 and m["counts"]["emails"] == 1
    assert m["since"] == "2026-09-20"
    assert api.calls[0] == ("GET", "/account-info/v3/details")


def test_pull_refuses_wrong_portal(tmp_path, monkeypatch):
    monkeypatch.setattr(pdl.c, "GOS_OUT", tmp_path)

    class Wrong(FakeApi):
        def call(self, method, path, payload=None):
            return 200, {"portalId": 51706233}

    try:
        pdl.pull("2026-W39", dt.date(2026, 9, 20), Wrong())
    except SystemExit:
        return
    raise AssertionError("expected SystemExit on sandbox portal")
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_pull_delta.py -q 2>&1 | tail -3
```

Expected: import error on `pull_delta`.

- [ ] **Step 3: Write `pull_delta.py`**

```python
from __future__ import annotations
"""Weekly read-only delta pull: deals and engagements modified in the last N days.
Reuses call/search_all/assoc from scripts/hubspot/funnel/pull_funnel.py. GET and POST-to-search/batch-read only."""
import argparse
import datetime as dt
import json
import pathlib
import sys
from typing import Iterable, List, Optional, Tuple

from . import common as c

DEAL_PROPS = ["dealname", "dealstage", "pipeline", "createdate", "closedate", "amount", "hubspot_owner_id",
              "closed_lost_reason", "closed_lost___further_detail", "dealtype", "campaign_source", "rep_source",
              "hs_analytics_source", "hs_is_closed_won", "hs_is_closed", "hs_lastmodifieddate"] + \
             [f"hs_v2_date_entered_{s}" for s in c.STAGES]
ENGAGEMENTS = {
    "meetings": ["hs_timestamp", "hs_meeting_title", "hs_internal_meeting_notes", "hs_meeting_outcome", "hs_activity_type", "hs_lastmodifieddate"],
    "calls": ["hs_timestamp", "hs_call_title", "hs_call_body", "hs_call_disposition", "hs_call_direction", "hs_lastmodifieddate"],
    "notes": ["hs_timestamp", "hs_note_body", "hs_lastmodifieddate"],
    "emails": ["hs_timestamp", "hs_email_subject", "hs_email_direction", "hs_email_status", "hs_sequence_id", "hs_lastmodifieddate"],
}


def week_dir(week: str) -> pathlib.Path:
    return c.GOS_OUT / "delta" / week


def since_date(days: int = 8) -> dt.date:
    return dt.date.today() - dt.timedelta(days=days)


def one_window(since: dt.date, end: Optional[dt.date] = None) -> Iterable[Tuple[dt.date, dt.date]]:
    yield (since, end or (dt.date.today() + dt.timedelta(days=1)))


def _rows(records: List[dict], amap: dict) -> List[dict]:
    out = []
    for r in records:
        row = {"id": r["id"], **r.get("properties", {})}
        for k, m in amap.items():
            row[k] = m.get(r["id"], [])
        out.append(row)
    return out


def pull(week: str, since: dt.date, api) -> dict:
    s, b = api.call("GET", "/account-info/v3/details")
    if b.get("portalId") != c.PROD_PORTAL_ID:
        sys.exit(f"ERROR: wrong portal {b.get('portalId')}; refusing to run")
    d = week_dir(week)
    d.mkdir(parents=True, exist_ok=True)
    counts = {}
    win = lambda st, en=None: one_window(since, en)
    deals = list(api.search_all("deals", DEAL_PROPS,
                                [{"propertyName": "pipeline", "operator": "EQ", "value": c.SALES_PIPELINE_AU}],
                                "hs_lastmodifieddate", since, "hs_lastmodifieddate", windows=win))
    ids = [r["id"] for r in deals]
    c.write_jsonl(d / "deals.jsonl", _rows(deals, {"company_ids": api.assoc("deals", "companies", ids) if ids else {}}))
    counts["deals"] = len(deals)
    for obj, props in ENGAGEMENTS.items():
        recs = list(api.search_all(obj, props, [], "hs_lastmodifieddate", since, "hs_lastmodifieddate", windows=win))
        ids = [r["id"] for r in recs]
        amap = {"company_ids": api.assoc(obj, "companies", ids) if ids else {},
                "deal_ids": api.assoc(obj, "deals", ids) if ids else {}}
        rows = _rows(recs, amap)
        if obj == "emails" and ids:
            bodies = {}
            for i in range(0, len(ids), 100):
                s, b = api.call("POST", "/crm/v3/objects/emails/batch/read",
                                {"properties": ["hs_email_text"], "inputs": [{"id": x} for x in ids[i:i + 100]]})
                for r in b.get("results", []):
                    bodies[r["id"]] = (r.get("properties") or {}).get("hs_email_text") or ""
            for row in rows:
                row["hs_email_text"] = bodies.get(row["id"], "")
        c.write_jsonl(d / f"{obj}.jsonl", rows)
        counts[obj] = len(recs)
    manifest = {"week": week, "since": since.isoformat(), "pulled_at": dt.datetime.utcnow().isoformat(timespec="seconds"),
                "portal": c.PROD_PORTAL_ID, "counts": counts}
    (d / "manifest.json").write_text(json.dumps(manifest, indent=1))
    print(f"delta {week} since {since}: {counts}")
    return manifest


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="Weekly read-only delta pull from HubSpot production.")
    ap.add_argument("--days", type=int, default=8)
    ap.add_argument("--week", default=c.iso_week(dt.date.today()))
    args = ap.parse_args(argv)
    sys.path.insert(0, str(c.GTM / "scripts" / "hubspot" / "funnel"))
    import pull_funnel as pf  # exits with a clear message if HUBSPOT_PROD_TOKEN is not set

    class Api:
        call = staticmethod(pf.call)
        search_all = staticmethod(pf.search_all)
        assoc = staticmethod(pf.assoc)

    pull(args.week, since_date(args.days), Api())
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_pull_delta.py -q 2>&1 | tail -3 && python3 -m scripts.growth_os.checks readonly
```

Expected: `3 passed` and `OK`.

- [ ] **Step 5: Live dry run**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && source ~/.hubspot_prod_env && python3 -m scripts.growth_os.pull_delta --days 8
```

Expected: `delta 2026-W3x since ...: {'deals': n, 'meetings': n, 'calls': n, 'notes': n, 'emails': n}` and a manifest file. If the token lacks a scope, HubSpot returns 403 and `search_all` exits with the object name; record which object failed in the data access register and continue with the objects that work.

- [ ] **Step 6: Commit**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: weekly read-only delta pull with portal assertion and tests

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 13: `weekly_digest.py` with tests

**Files:**
- Create: `~/Documents/GTM Project/scripts/growth_os/weekly_digest.py`
- Create: `~/Documents/GTM Project/scripts/growth_os/tests/test_weekly_digest.py`

**Interfaces:**
- Consumes: the delta folder (Task 12), `build_record` and `select_segments` (Task 4), `tag_record` (Task 5), `summarise` (Task 6), `deidentify`, `school_label` (Task 7).
- Produces: `growth-os/customer-truth/weekly/<week>.md` and a copy in `outputs/growth_os/digests/<week>.md`; appends new deals to `deals_early_text.jsonl` and `deals_tagged.jsonl`; rewrites `summary.json`.
  Functions: `delta_events_by_company(delta_dir) -> Dict[str, List[dict]]` (converts delta engagement rows to timeline-shaped events), `closed_this_week(delta_deals, since) -> List[dict]`, `movements(prev_summary, new_summary, min_pts=5.0) -> List[dict]`, `render_digest(week, closed_recs, tagged_rows, movements, new_summary, errors) -> str`, `main(argv)`.

- [ ] **Step 1: Write the failing tests**

`scripts/growth_os/tests/test_weekly_digest.py`:
```python
from __future__ import annotations
import json
from scripts.growth_os import weekly_digest as wd


def test_delta_events_by_company_converts_rows(tmp_path):
    (tmp_path / "emails.jsonl").write_text(json.dumps({"id": "e1", "hs_timestamp": "2026-09-22T00:00:00Z", "hs_email_direction": "INCOMING_EMAIL",
                                                        "hs_email_subject": "s", "hs_email_text": "school words", "company_ids": ["c1"], "deal_ids": []}) + "\n")
    (tmp_path / "notes.jsonl").write_text(json.dumps({"id": "n1", "hs_timestamp": "2026-09-22T01:00:00Z", "hs_note_body": "rep note", "company_ids": ["c1"], "deal_ids": ["d1"]}) + "\n")
    (tmp_path / "meetings.jsonl").write_text("")
    (tmp_path / "calls.jsonl").write_text("")
    ev = wd.delta_events_by_company(tmp_path)
    kinds = sorted((e["type"], e["direction"], e["body"]) for e in ev["c1"])
    assert kinds == [("email", "INCOMING_EMAIL", "school words"), ("note", "None", "rep note")]
    assert ev["c1"][0]["ts"] == "2026-09-22T00:00:00"


def test_closed_this_week_filters():
    deals = [{"id": "a", "hs_is_closed": "true", "hs_is_closed_won": "true", "closedate": "2026-09-24T00:00:00Z", "dealtype": "newbusiness", "pipeline": "41942802"},
             {"id": "b", "hs_is_closed": "true", "hs_is_closed_won": "false", "closedate": "2026-08-01T00:00:00Z", "dealtype": "newbusiness", "pipeline": "41942802"},
             {"id": "c", "hs_is_closed": "false", "hs_is_closed_won": "false", "closedate": "2026-09-24T00:00:00Z", "dealtype": "newbusiness", "pipeline": "41942802"},
             {"id": "d", "hs_is_closed": "true", "hs_is_closed_won": "true", "closedate": "2026-09-24T00:00:00Z", "dealtype": "existingbusiness", "pipeline": "41942802"}]
    import datetime as dt
    assert [d["id"] for d in wd.closed_this_week(deals, dt.date(2026, 9, 20))] == ["a"]


def test_movements_reports_rate_changes_over_threshold():
    prev = {"tags": {"no_prep": {"rate_present": 0.50, "n_present": 30, "thin": False}, "price_budget": {"rate_present": 0.2, "n_present": 25, "thin": False}}}
    new = {"tags": {"no_prep": {"rate_present": 0.58, "n_present": 33, "thin": False}, "price_budget": {"rate_present": 0.22, "n_present": 26, "thin": False}}}
    mv = wd.movements(prev, new, min_pts=5.0)
    assert mv == [{"tag": "no_prep", "from": 0.5, "to": 0.58, "delta_pts": 8.0, "n": 33}]


def test_render_digest_sections():
    md = wd.render_digest("2026-W39",
                          [{"deal_id": "a", "outcome": "won", "amount": 9000.0, "segment": "Catholic", "state": "NSW", "dealtype": "newbusiness", "company_names": ["Alpha College"]}],
                          [{"deal_id": "a", "tags": [{"tag": "oversight_visibility", "speaker": "school", "quote": "Alpha College needs to see delivery", "event_id": "e"}], "other": [], "error": None}],
                          [{"tag": "no_prep", "from": 0.5, "to": 0.58, "delta_pts": 8.0, "n": 33}],
                          {"population": {"deals": 100, "won": 40, "lost": 60}, "base_rate": 0.4},
                          [{"deal_id": "z", "reason": "no_text"}])
    assert "# Weekly digest 2026-W39" in md
    assert "## Closed this week" in md and "oversight_visibility" in md and "Alpha College" not in md
    assert "## Movements" in md and "no_prep" in md
    assert "## Proposed truth changes" in md and "## Needs Caelum" in md and "z" in md
    assert "Source:" in md
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_weekly_digest.py -q 2>&1 | tail -3
```

Expected: import error on `weekly_digest`.

- [ ] **Step 3: Write `weekly_digest.py`**

```python
from __future__ import annotations
"""Weekly digest: tag the week's closed AU new-business deals, recompute the summary, list movements
and proposed truth changes, write the digest into Growth OS. Reads the delta folder and local caches only."""
import argparse
import datetime as dt
import json
import pathlib
import shutil
import sys
from typing import Dict, List, Optional

from . import common as c
from .assemble_early_text import build_record, default_load_history
from .join_outcomes import summarise, to_csv_rows
from .tag_deals import tag_record
from .write_truth import deidentify, school_label


def _ts(s: str) -> str:
    d = c.parse_ts(s)
    return d.isoformat(timespec="seconds") if d else ""


def delta_events_by_company(delta_dir: pathlib.Path) -> Dict[str, List[dict]]:
    out: Dict[str, List[dict]] = {}
    spec = {"emails": ("email", "hs_email_text", "hs_email_direction", "hs_email_subject"),
            "notes": ("note", "hs_note_body", None, None),
            "meetings": ("meeting", "hs_internal_meeting_notes", None, "hs_meeting_title"),
            "calls": ("call", "hs_call_body", "hs_call_direction", "hs_call_title")}
    for fname, (typ, body_f, dir_f, subj_f) in spec.items():
        for row in c.read_jsonl(delta_dir / f"{fname}.jsonl"):
            ev = {"type": typ, "id": str(row.get("id")), "ts": _ts(row.get("hs_timestamp")),
                  "direction": (row.get(dir_f) if dir_f else None) or "None",
                  "subject": (row.get(subj_f) if subj_f else "") or "", "body": row.get(body_f) or "",
                  "outcome": "None", "activity_type": "None", "disposition": "None", "auto_reply": "False"}
            for cid in c.ids(row.get("company_ids")):
                out.setdefault(cid, []).append(ev)
    return out


def closed_this_week(delta_deals: List[dict], since: dt.date) -> List[dict]:
    out = []
    for d in delta_deals:
        if str(d.get("pipeline")) != c.SALES_PIPELINE_AU or not c.is_new_business(d) or c.outcome(d) == "open":
            continue
        cd = c.parse_ts(d.get("closedate"))
        if cd and cd.date() >= since:
            out.append(d)
    return out


def movements(prev: dict, new: dict, min_pts: float = 5.0) -> List[dict]:
    out = []
    for tag, e in new.get("tags", {}).items():
        p = (prev.get("tags") or {}).get(tag)
        if not p or e.get("thin") or p.get("rate_present") is None or e.get("rate_present") is None:
            continue
        delta = round((e["rate_present"] - p["rate_present"]) * 100, 1)
        if abs(delta) >= min_pts:
            out.append({"tag": tag, "from": p["rate_present"], "to": e["rate_present"], "delta_pts": delta, "n": e["n_present"]})
    return sorted(out, key=lambda m: -abs(m["delta_pts"]))


def render_digest(week: str, closed_recs: List[dict], tagged_rows: List[dict], mv: List[dict], new_summary: dict, errors: List[dict]) -> str:
    tag_by_id = {t["deal_id"]: t for t in tagged_rows}
    L = [f"# Weekly digest {week}", "", f"Automated. Branch auto/weekly. Population now {new_summary['population']['deals']} closed deals, base win rate {new_summary.get('base_rate')}.", "",
         "## Closed this week", "", "| Deal | Outcome | Amount | School | School-voiced tags | Quote |", "|---|---|---|---|---|---|"]
    for r in closed_recs:
        t = tag_by_id.get(r["deal_id"], {})
        stags = [x for x in t.get("tags", []) if x.get("speaker") == "school"]
        quote = deidentify(stags[0]["quote"], r.get("company_names") or []) if stags else ""
        amt = "" if r.get("amount") is None else f"{int(r['amount'])}"
        L.append(f"| {r['deal_id']} | {r['outcome']} | {amt} | {school_label(r)} | {', '.join(sorted({x['tag'] for x in stags}))} | {quote} |")
    if not closed_recs:
        L.append("| none | | | | | |")
    L += ["", "Source: outputs/growth_os/delta/" + week + "/ and outputs/customer_truth/deals_tagged.jsonl", "", "## Movements", ""]
    L += [f"- {m['tag']}: {m['from']:.2f} to {m['to']:.2f} ({m['delta_pts']:+.1f} pts, n={m['n']})" for m in mv] or ["- none above 5 points"]
    L += ["", "## Proposed truth changes", ""]
    props = [m for m in mv if abs(m["delta_pts"]) >= 10]
    L += [f"- Re-rank {m['tag']} ({m['delta_pts']:+.1f} pts at n={m['n']}). Merge to accept; reject to hold." for m in props] or ["- none. Minimum sample and 10-point rule not met by any movement."]
    L += ["", "## Needs Caelum", ""]
    L += [f"- {e['deal_id']}: {e['reason']}" for e in errors] or ["- nothing"]
    L += ["", "Source: outputs/customer_truth/summary.json (this run) against summary.prev.json", ""]
    return "\n".join(L).replace("—", "-")


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="Write the weekly digest from the delta pull.")
    ap.add_argument("--week", default=c.iso_week(dt.date.today()))
    ap.add_argument("--days", type=int, default=8)
    ap.add_argument("--model", default="sonnet")
    args = ap.parse_args(argv)
    delta_dir = c.GOS_OUT / "delta" / args.week
    since = dt.date.today() - dt.timedelta(days=args.days)
    companies = c.load_companies()
    extra = delta_events_by_company(delta_dir)

    def load_history(cid: str):
        h = default_load_history(cid) or {"timeline": []}
        seen = {str(e.get("id")) for e in h["timeline"]}
        h["timeline"] = h["timeline"] + [e for e in extra.get(cid, []) if e["id"] not in seen]
        return h

    early_path, tagged_path = c.OUT / "deals_early_text.jsonl", c.OUT / "deals_tagged.jsonl"
    known = {r["deal_id"] for r in c.read_jsonl(early_path)}
    closed = [d for d in closed_this_week(c.read_jsonl(delta_dir / "deals.jsonl"), since) if str(d["id"]) not in known]
    recs, rows, errors = [], [], []
    for d in closed:
        rec = build_record(d, companies, load_history)
        row = tag_record(rec, model=args.model)
        recs.append(rec); rows.append(row)
        if row["error"]:
            errors.append({"deal_id": rec["deal_id"], "reason": row["error"]})
    c.append_jsonl(early_path, recs)
    c.append_jsonl(tagged_path, rows)
    prev_path, cur_path = c.OUT / "summary.prev.json", c.OUT / "summary.json"
    prev = json.load(cur_path.open()) if cur_path.exists() else {"tags": {}}
    if cur_path.exists():
        shutil.copy(cur_path, prev_path)
    new = summarise(c.read_jsonl(early_path), c.read_jsonl(tagged_path))
    cur_path.write_text(json.dumps(new, indent=1))
    md = render_digest(args.week, recs, rows, movements(prev, new), new, errors)
    out = c.GROWTH_OS / "customer-truth" / "weekly" / f"{args.week}.md"
    out.parent.mkdir(parents=True, exist_ok=True)
    out.write_text(md)
    copy = c.GOS_OUT / "digests" / f"{args.week}.md"
    copy.parent.mkdir(parents=True, exist_ok=True)
    copy.write_text(md)
    print(f"closed this week: {len(closed)}  errors: {len(errors)}  WROTE {out}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests -q 2>&1 | tail -3
```

Expected: `38 passed`.

- [ ] **Step 5: Manual first run against the live delta from Task 12**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.weekly_digest && python3 -m scripts.growth_os.checks
```

Expected: a digest at `growth-os/customer-truth/weekly/2026-W3x.md` and `OK`. Read it. Note: this manual run writes to the working tree on `main`; do not commit it there. Task 14 moves the flow onto the branch. Discard with `cd ~/Documents/growth-os && git checkout -- . && git clean -fd customer-truth/weekly` after reading, or keep it for the branch commit in Task 14.

- [ ] **Step 6: Commit**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: weekly digest tags the week's closed deals, reports movements and proposed truth changes

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 14: `run_weekly.sh`, launchd, branch flow

**Files:**
- Create: `~/Documents/GTM Project/scripts/growth_os/run_weekly.sh`
- Create: `~/Library/LaunchAgents/com.wellio.growthos.weekly.plist`

- [ ] **Step 1: Write `run_weekly.sh`**

```bash
#!/bin/zsh
# Sunday job: read-only delta pull, digest, commit to auto/weekly. Never touches main.
set -euo pipefail
export PATH="/Users/cleonard1998/.local/bin:/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin"
GTM="/Users/cleonard1998/Documents/GTM Project"
GOS="/Users/cleonard1998/Documents/growth-os"
LOG="$GTM/outputs/growth_os/logs/$(date +%F).log"
mkdir -p "$(dirname "$LOG")"
exec >>"$LOG" 2>&1
echo "=== run_weekly $(date)"
source ~/.hubspot_prod_env
cd "$GTM"
python3 -m scripts.growth_os.checks readonly
python3 -m scripts.growth_os.pull_delta --days 8
python3 -m scripts.growth_os.weekly_digest
python3 -m scripts.growth_os.checks
cd "$GOS"
WEEK=$(python3 -c "import datetime as d; y,w,_=d.date.today().isocalendar(); print(f'{y}-W{w:02d}')")
git stash --include-untracked -q || true
git checkout -q -B auto/weekly main
git stash pop -q || true
git add customer-truth/weekly
git commit -q -m "auto: weekly digest $WEEK

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>" || echo "nothing to commit"
git push -q -u origin auto/weekly --force-with-lease || echo "push skipped (no remote or offline)"
git checkout -q main
echo "=== done $(date)"
```

```bash
chmod +x "/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/run_weekly.sh"
```

- [ ] **Step 2: Manual end-to-end run**

```bash
bash "/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/run_weekly.sh"; tail -20 "/Users/cleonard1998/Documents/GTM Project/outputs/growth_os/logs/$(date +%F).log"; cd ~/Documents/growth-os && git log --oneline -1 auto/weekly && git status --short
```

Expected: log ends with `=== done`; `auto/weekly` has one `auto: weekly digest` commit; `main` working tree is clean.

- [ ] **Step 3: Write the launchd plist**

`~/Library/LaunchAgents/com.wellio.growthos.weekly.plist`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>com.wellio.growthos.weekly</string>
  <key>ProgramArguments</key>
  <array>
    <string>/bin/zsh</string>
    <string>/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/run_weekly.sh</string>
  </array>
  <key>StartCalendarInterval</key>
  <dict><key>Weekday</key><integer>0</integer><key>Hour</key><integer>21</integer><key>Minute</key><integer>0</integer></dict>
  <key>StandardOutPath</key><string>/Users/cleonard1998/Documents/GTM Project/outputs/growth_os/logs/launchd.out</string>
  <key>StandardErrorPath</key><string>/Users/cleonard1998/Documents/GTM Project/outputs/growth_os/logs/launchd.err</string>
</dict>
</plist>
```

```bash
launchctl unload ~/Library/LaunchAgents/com.wellio.growthos.weekly.plist 2>/dev/null; launchctl load ~/Library/LaunchAgents/com.wellio.growthos.weekly.plist && launchctl list | grep growthos
```

Expected: one line with the label. Weekday 0 is Sunday. The Mac must be awake at 21:00 Sunday; if it is not, run the script by hand Monday morning.

- [ ] **Step 4: Update the agent spec status and commit**

In `agents/weekly-pull-and-tag.md` change `Status: specced` to `Status: running. Since: <today>.` and in `agents/README.md` the status table row to `running`. Commit on `main` and push.

---

### Task 15: Week 2 gate, README rewrite, verification

- [ ] **Step 1: Full verification**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests -q | tail -1 && python3 -m scripts.growth_os.checks && git -C ~ status --short "Documents/GTM Project" | grep -v "scripts/growth_os\|outputs/" ; python3 -c "import json;d=json.load(open('$HOME/.claude/settings.json'));assert 'gtm-skills@gtm-plugins' not in d.get('enabledPlugins',{}); print('plugin scope OK')"
```

Expected: `38 passed`, `OK`, no unexpected GTM Project modifications, `plugin scope OK`.

- [ ] **Step 2: Rewrite the root README for the five-minute read**

Replace the status version from Task 1 with: one paragraph on what this is; "What has run" listing cycle zero with its date, population and base rate, and the weekly job with its first week; "The one finding" copied from the truth file's first paragraph; three links (results, customer truth, decisions); the loop in one line; a "What is not yet measured" line pointing at `results/metric-definitions.md`. No school names. End with `Source:` lines for the figures.

- [ ] **Step 3: Update `results/README.md`**

Three sections: What has been built (three lines with links), Baseline (link to `baseline-2026-09.md` with the four headline numbers), What the role would be measured on (link to `metric-definitions.md`, and the sentence "Hook attribution is not measurable today; the stamp waits for the decision"). Each with a `Source:` line.

- [ ] **Step 4: Time the cold read, record, commit, push**

Time a cold read of `README.md` and `results/README.md`. Record under "Week 2 gate" in `decisions/README.md` with the pytest count, the checks result and the read time. Then:

```bash
cd ~/Documents/growth-os && git add -A && git commit -m "README and results rewritten for the five-minute read; week 2 gate recorded

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>" && git push
```

- [ ] **Step 5: Friday sponsor review 2**

Show the Head of Marketing the truth file and the first digest. Record reactions in `decisions/2026-09-26-sponsor-review-2.md`. His corrections to tags or quotes go into the codebook change log and the agent spec's corrections log, not into the truth file by hand.

---

## Self-review against the spec

- Section 1 (loop): stages 1, 2 and 5 are built (Tasks 12, 13, 5 to 7); stage 3 is the branch merge (Task 14); stage 4 stamp is documented as not yet existing (Task 8 metric definitions). Retrospective mode labelled in `write_truth --mode` and the README. Promotion rule written in Task 1 creative-testing README and enforced by `thin` in Task 6 and the 10-point rule in Task 13.
- Section 4 (tooling): plugin scope verified in Task 15; CLAUDE.md in Task 1; private GitHub in Task 9; launchd in Task 14; secrets never written (checked by `.gitignore` and `run_weekly.sh` sourcing the env file).
- Section 7 (working machine): Tasks 4 to 7 and 11.
- Section 9 (data access): first row's fallback (cycle zero on cache) is Task 11; the live delta in Task 12 step 5 records scope failures.
- Section 10 (risk): read-only check (Task 7), de-identification check (Task 7), aggregate-not-by-rep (no rep field is ever rendered; `hubspot_owner_id` is pulled but never written to Growth OS), main always presentable (Task 14 branch flow), sponsor cadence (Tasks 10 and 15).
- Section 13 (verification): all eight checks appear in Tasks 7, 14 and 15 except the "second person rerun", which is Task 11 step 4.
- Deferred to the weeks 3 and 4 plan: agent specs for signals-researcher, sequence-analytics, meeting-writeback, need-scanner; outbound-engine inventory; markets verification; results scorecard and attribution; case study.

Type consistency checked: `build_record`, `select_segments`, `tag_record`, `summarise`, `deidentify`, `school_label` are imported in Task 13 with the signatures defined in Tasks 4 to 7. `c.GOS_OUT` is defined in Task 3 and monkeypatched in Task 12 tests.
