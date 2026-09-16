# Growth OS: design

Date: 16 September 2026. Status: draft for Caelum's review.
Brainstorm: `~/Desktop/GTM:Testing/brainstorms/2026-09-15-growth-marketing-role-takeover.md`.
Source idea: Greg Isenberg, "Marketing Engineer: The $1M Job with AI Agents" (YouTube 8ZC1G1ezN5o).

## 1. Purpose

Growth OS is Wellio's marketing memory: one repo that holds what the market has told us, the machines that act on it, the job spec of every agent, and the results, so the work stops disappearing into chats.

It has two readers and must serve both:

1. Caelum and Claude, working in it daily. Every file must be re-runnable or re-derivable from a cited source.
2. A founder or the Head of Marketing, opening it cold. They must understand what has been built, and what it found, in five minutes from the root README.

The second reader is the reason this exists now. The founders' stated objection is that they do not know what Caelum has built or how it translates. This repo is the thing that gets opened in front of them. It is a demonstration, not an application.

### Success criteria

- A founder can read `README.md` and `results/README.md` in five minutes and name the three things Caelum has built and the one finding that matters.
- The customer-truth pass has run end to end on real HubSpot data, produced `customer-truth/what-the-market-is-telling-us.md`, and can be rerun from `method.md` without Caelum present.
- Every agent has a written job spec in the same format. At least one has run.
- Nothing in the build writes to HubSpot, sends an email, or moves a file that another skill depends on.
- Done by 13 October 2026, before any founder probe and well before the December planning cycle.

## 2. Principles

Carried from the video and from the RevOps work already done:

- **Memory over chats.** Findings live in files with a source citation. Chat output that is not filed does not exist.
- **Distil, do not duplicate.** GTM Project stays the workshop: scripts, raw pulls, caches. Growth OS holds the distilled finding and a path to its source. No raw data is copied in.
- **Job specs for agents.** Every agent gets the same seven-field spec before it runs, the way a hire gets a job description.
- **Measure pipeline, not activity.** Results are qualified replies, meetings and pipeline created. Sends and opens are diagnostics.
- **One working machine.** One system runs end to end before a second starts. Content and creative folders hold templates until there is budget.
- **Read-only until the decision.** No HubSpot writes, no live sends. The production portal is 20058914; the sandbox 51706233 is never the source of a finding.
- **Precision, human on every first send.** Inherited from Five Engines section 5. Applies to every agent spec.

## 3. Location

A standalone repository at `~/Documents/growth-os` with its own git history. Decided 16 September over two alternatives (a folder inside GTM Project, or restructuring GTM Project), because the repo must be clean enough to open in front of a founder, and because the home directory is currently the git root for everything, which makes GTM Project's history unshowable.

Relationship to GTM Project (`~/Documents/GTM Project`):

- Growth OS files cite GTM Project sources by absolute path, in a `Source:` line under each finding.
- Scripts written for Growth OS machines live in GTM Project under `scripts/growth_os/`, next to the caches they read. Growth OS holds the method file that tells you how to run them.
- Large outputs (tagged JSONL, per-deal evidence) stay in GTM Project under `outputs/customer_truth/`. Growth OS holds the summary tables and the quotes that made the cut.

## 4. Layout

```
growth-os/
  README.md                 what this is, how to read it in five minutes, what has run
  customer-truth/
    README.md               the hook in one paragraph, then the evidence
    what-the-market-is-telling-us.md    the machine's output; rewritten each run
    hook-codebook.md        the tags, their definitions, example phrases
    method.md               how the pass runs, inputs, freshness, rerun steps
    buyer-and-champion.md   who buys, who signs, by region; from Q13
    objections.md           loss reasons and the notes behind them
    quotes/                 curated, de-identified quotes by hook and by role
  markets/
    README.md               the compliance-versus-discretionary axis, and why it decides the channel
    au.md  uk.md  europe.md  canada.md  international.md
    need-data-sources.md    public school-level datasets per market, with URLs and what each measures
  outbound-engine/
    README.md               what exists (Signals pilot, SDR outreach skill, primary attach spine), what does not (EDM engine)
    signals-layer.md        the unified layer above regional tools; open design decision
    edm-engine-spec.md      the greenfield JD deliverable, specified not built
    sequences/              designs only, no sends
  content-engine/
    README.md               hooks turned into content briefs; smallest folder
    brief-template.md
  creative-testing/
    README.md               empty until budget; the test log template
    test-log-template.md
  agents/
    README.md               the spec format and the run status table
    _spec-template.md
    customer-truth-pass.md
    signals-researcher.md
    meeting-writeback.md
    need-scanner-eu-ca.md
  results/
    README.md               one page a founder reads
    metric-definitions.md
    baseline-2026-09.md
  decisions/
    README.md
    2026-09-16-standalone-repo.md
    2026-09-16-first-machine.md
  docs/superpowers/         this spec and the plan that follows it
```

Two departures from the video's six folders, both Wellio-specific:

- `markets/` exists because Europe and Canada entry-market selection is open and unowned, and the compliance-versus-discretionary axis (UK ~$2k need-based, AU ~$10.9k discretionary) decides channel choice before anything else.
- `decisions/` exists because the dated decision log in `docs/revops/DECISIONS.md` is the habit that made the RevOps work auditable. Same format.

## 5. The working machine: the customer-truth pass

### The question

What did the schools that bought say early in the conversation that the schools that did not buy never said? Which named pain points convert at a higher rate, at a larger deal size, in a shorter cycle?

### The hypotheses under test

From the brainstorm, Q14:

- Leader hook: oversight of a standardised program plus trackable wellbeing outcomes, and a report that can go to the board.
- Teacher hook: no-prep, psychology-backed, curriculum-mapped lessons; time saved.
- Null: "wellbeing matters" on its own does not separate winners from losers.

The pass may find a hook nobody hypothesised. The codebook has an `other` tag with free text, and the method requires reading a sample of `other` before the file is written.

### Population

AU new business only, matching the funnel analysis scope. Deals in the Sales Pipeline - AU with a close date from 1 January 2025 to the run date, closed won or closed lost. Open deals are held out and reported separately. UK and international are excluded because their deals live in other pipelines and their motion is different; the method file records how to extend.

Expected size, from the 3 September funnel pull: 986 AU deals since January 2025, of which roughly 308 won and 378 lost among the qualified cohort. Enough for hook-level win rates with honest n.

### Inputs, all already on disk

| Input | Where | Notes |
|---|---|---|
| Deals, meetings, calls, emails, leads, tasks | `GTM Project/outputs/sdr_funnel/*.jsonl` | Pulled 3 Sep 2026 via REST from production. Emails and email bodies since July 2025. |
| Per-company timelines, 5,513 schools | `GTM Project/outputs/sdr_history/<company_id>.json` | Company, contacts, deals, 56-event timeline, last human inbound. Pulled late August. |
| Distilled records, 2,881 schools | `GTM Project/outputs/sdr_distilled/` | Prior distillation for the SDR relevance project; different question, reusable context. |
| Exec report quotes | `~/Desktop/Exec Report Skill/reports/_generator/` and Shae batch | 116 schools, customer voice, de-identified. Renewal-side evidence, not first-meeting. |

Freshness rule: the run date is the pull date of `sdr_funnel`. Deals closed after 3 September are missing from the first run. A refresh is a rerun of `scripts/hubspot/funnel/pull_funnel.py`, read-only, and is recorded in `method.md`. The HubSpot connector is used only to fill gaps and only with read queries.

### Method, four stages

1. **Assemble the early-conversation text per deal.** For each deal: notes of the first two meetings, the first three email threads, call notes, and the lead record's qualification notes, all dated before the deal reached Evaluating. Everything after that stage is excluded so the pass measures what was said early, not what was said to close. Output: one JSONL record per deal with the text, outcome, amount, source, cycle days, segment, close date. Lives in `outputs/customer_truth/deals_early_text.jsonl`.
2. **Tag against the codebook.** Claude reads each record with a fixed prompt and the codebook, returns the tags present, the verbatim phrase that triggered each tag, and who said it (school or Wellio). School-voiced tags and Wellio-voiced tags are kept separate; a hook only counts if the school raised it or agreed to it in their own words. Output: `outputs/customer_truth/deals_tagged.jsonl`. The prompt and codebook version are stored with the output.
3. **Join to outcome.** Per tag: win rate when present versus absent, median deal amount, median cycle length, with n for each cell. Per outcome: tag frequency among won versus lost. Broken down by source (Free Preview / Contact Sales, outbound, referral, event) and by segment (Catholic, government, independent). A cell under n=20 is shown but marked thin.
4. **Write the file.** `what-the-market-is-telling-us.md` states the answer in one paragraph, then the table, then three to five quotes per surviving hook in the schools' words, then what did not separate winners from losers, then the known limits. Quotes carry school type and state, never the school name, in the file that a founder reads. The internal evidence file keeps the identifiers.

### Known limits, stated in the file

From the funnel analysis, 3 September: meeting outcomes are not recorded (70 percent of past discovery meetings still read scheduled), a third of meetings have no type, and call dispositions exist only from February 2026. The pass therefore reads what was written, not whether the meeting happened. Loss-reason labels are unreliable (128 of 214 audited); the notes behind them are used, the labels are not.

### Rerun cadence

Quarterly, or after any pull refresh. Each run appends a dated entry to `method.md` and overwrites the output file. The previous output is kept in git history, not as a copy.

## 6. Agents

### Spec format

Every file in `agents/` has the same eight sections, the video's seven plus a status line:

1. Job in one sentence
2. Data source, with path or object and the read scope
3. Run schedule
4. Filters: what it must skip
5. Expected output: file, format, where it lands
6. Approval step: who sees it before anything acts on it
7. Metric: the one number that says it worked
8. What breaks it: the known failure modes and the check that catches them

Plus a status table in `agents/README.md`: specced, dry-run, running, retired, with dates.

### The first four specs

| Agent | Status target by 13 Oct | Why this one |
|---|---|---|
| customer-truth-pass | running | The working machine. Section 5 is its spec. |
| signals-researcher | dry-run | Already built as the sdr-outreach skill. Writing its spec in this format shows the machinery exists and waits on Sales Manager sign-off. |
| meeting-writeback | specced | First agent in the Five Engines build order. Closes the recording gap that blocks every meeting metric. Cannot run before the decision because it writes. |
| need-scanner-eu-ca | specced, stretch dry-run | Reads the public datasets in `markets/need-data-sources.md`, scores schools on wellbeing need, outputs a ranked entry list per market. The original-thinking piece. |

The remaining seven Five Engines agents (inbound responder, reschedule defence, attempt governor, Baseline analyst, referral queue, renewal risk reader, proposal assembler) are listed in `agents/README.md` with one line each and no spec. Specs are written when they are next.

## 7. Markets

One file per market, same headings: footprint today, buyer and signer, average deal and what drives it, compliance or discretionary, statutory hook if any, public need data available, entry route, what we do not know.

AU and UK are written from what is known (brainstorm Q13, Five Engines penetration figures, funnel analysis). Europe, Canada and international start as honest stubs with the classification question open and the datasets listed. The stretch goal is to run the need scanner on one Canadian province (Ontario school climate survey or BC Student Learning Survey) and one European candidate, and write the entry recommendation.

`need-data-sources.md` lists, per market, the dataset, what it measures, granularity, access method and URL: SA Wellbeing and Engagement Collection, NSW Tell Them From Me via annual reports, Vic AtoSS via annual reports, My School ICSEA and staffing, England Ofsted personal development judgements, free school meal percentage, Mental Health Support Team coverage, Ontario school climate surveys, BC Student Learning Survey. Wellio's own 210,000-student benchmark is the layer on top. Sources not yet verified are marked unverified until someone opens them.

## 8. Results

`results/README.md` is one page. Three parts:

1. **What has been built**, three lines with links: Five Engines strategy, the Sales Signals pilot and SDR outreach skill, the primary attach spine and scraping pipeline. Then the customer-truth pass once it has run.
2. **Baseline**, the numbers already held, each with a source and a date: 2025 versus 2026 new business to 3 September, 2025 full-year and the 70 percent after 4 September, the AU funnel shape (booked meeting to won deal 22 percent), penetration by segment, the Signals dry-run counts, the primary attach spine counts.
3. **What the role would be measured on**, defined in `metric-definitions.md`: qualified reply, meeting booked, meeting held, pipeline created, pipeline won, by source and market. Each has a definition, the HubSpot property or object that would carry it, and whether it is measurable today. Cells that cannot be measured today say "not yet measured, because" and name the recording gap. Nothing is hidden.

## 9. Decisions

Same format as `docs/revops/DECISIONS.md`: date, decision, alternatives considered, why, who decided, what would reverse it. First two entries are the standalone repo and the first-machine choice. Every later change to layout, method or scope gets an entry.

## 10. Content engine and creative testing

Templates and a README only. The content README states the rule: a content brief starts from a tagged hook in `customer-truth/`, never from a topic. The creative README states that nothing goes here until there is budget authority, and holds the test log template (hypothesis, audience, variant, metric, result, decision).

## 11. Out of scope

- Any HubSpot write, any email send, any sequence enrolment.
- Moving or renaming any file in GTM Project. Skills depend on those paths.
- Events and conferences. The honest gap; a decisions entry records that it is not addressed here.
- UK and international customer-truth. Method file says how to extend.
- Renewal-side customer truth from exec report quotes is cited as supporting evidence, not merged into the new-business pass.
- The Five Engines artifact itself. It is linked, not rewritten.

## 12. Sequence

| Week | Dates | Deliverable | Isenberg's week |
|---|---|---|---|
| 1 | 16 to 22 Sep | Repo scaffold, all READMEs, decisions entries, migration of existing findings into customer-truth, markets/au and uk, outbound-engine and results baseline. Codebook drafted. | Audit and market map |
| 2 | 23 to 29 Sep | Customer-truth pass stages 1 to 4 run. `what-the-market-is-telling-us.md` written. Method file complete. | Repo and the market file |
| 3 | 30 Sep to 6 Oct | Four agent specs. Markets stubs for Europe, Canada, international. `need-data-sources.md` verified. Need scanner dry-run if time allows. | One working machine |
| 4 | 7 to 13 Oct | Results page final. Case study written from the repo, on Wellio's own data, ready to hand over unprompted. Root README rewritten for the five-minute read. | Results and case study |

Day-job constraint: this runs alongside renewals close-out and CS projects. Week 2 is the only week with a hard dependency (the pass). If it slips, week 3 slips with it and the need scanner is dropped first.

## 13. Verification

This is a documents-and-analysis repo, so the tests are checks, run before each week is called done:

- **Citation check.** Every finding in `customer-truth/`, `markets/` and `results/` has a `Source:` line with a path or URL. A script in `scripts/growth_os/check_citations.py` lists any heading-level finding without one.
- **Read-only audit.** Every script in `scripts/growth_os/` is grepped for HubSpot write endpoints and connector write tools before it runs. The method file records the portal ID used.
- **Rerun check.** A second person, or a fresh Claude session, follows `method.md` and reproduces the summary table within rounding.
- **De-identification check.** The founder-facing output file contains no school name from the population. Grep against the company names in the deals file.
- **Five-minute check.** Caelum times a cold read of the root README and results page.
- **Path check.** No file in GTM Project has moved. `git -C ~ status` on that folder shows only modifications Caelum expects.

## 14. Migration list

What moves in as a distilled finding, and where:

| Source in GTM Project | Lands in |
|---|---|
| `docs/gtm-strategy-2026-09/01-hubspot-findings.md` | results/baseline, customer-truth/objections |
| `docs/gtm-strategy-2026-09/03-market-research-brief.md` | markets/au, markets/need-data-sources |
| `docs/gtm-strategy-2026-09/04-strategy-working-draft.md` and the artifact | results/README link, outbound-engine/README |
| `docs/revops/2026-09-03-sdr-funnel-leak-analysis.md` | results/baseline, results/metric-definitions (the recording gaps) |
| `docs/revops/2026-08-20-sam-tof-synthesis.md` | markets/au (SAM 720, 208 clean-bulk), outbound-engine/signals-layer |
| `docs/revops/2026-08-25-two-motions-analysis.md` | customer-truth/buyer-and-champion (net new versus returning rollout) |
| `docs/revops/2026-08-26-distillation-findings-log.md` | customer-truth/objections (systemic issues: domain restriction, YouTube blocking, support AI) |
| `docs/revops/2026-09-05-sdr-outreach-pilot-assessment.md` and brief | outbound-engine/README, agents/signals-researcher |
| `outputs/primary_attach_2026-09-05/` | outbound-engine/README (the scraping pipeline as evidence), results/baseline counts |
| Exec report curated quotes | customer-truth/quotes, marked renewal-side |
| Brainstorm Q13, Q14 | customer-truth/buyer-and-champion, hook-codebook |

## 15. Open questions carried from the brainstorm

Affecting this build:

- **Whether primary attach travels with the role as ABM.** Caelum's call. Affects whether `outbound-engine/README` presents it as marketing evidence or CS work. Default: present it as the scraping and list-building capability, owner unstated.
- **Outreach data access.** Needed to extend customer-truth to UK. Not needed for the AU pass. Flagged in `markets/uk.md`.
- **AU sequence performance baseline.** Pullable read-only by Caelum. Goes into `results/baseline` if pulled in week 1; otherwise "not yet measured".

Not affecting this build, carried into `decisions/README.md` as pending: international pricing, tool inventory per region, external challenge brief, events budget, UK meeting rate, decision deadline, other candidate, wellbeing-to-attendance evidence base, webinar recording location, numeric 2027 target.
