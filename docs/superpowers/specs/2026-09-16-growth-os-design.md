# Growth OS: design (v2, loop-first)

Date: 16 September 2026. Status: approved direction, expanded scope. Supersedes v1 of the same day.
Brainstorm: `~/Desktop/GTM:Testing/brainstorms/2026-09-15-growth-marketing-role-takeover.md`.
Source idea: Greg Isenberg, "Marketing Engineer: The $1M Job with AI Agents" (YouTube 8ZC1G1ezN5o).
Owner: Caelum. Sponsor when shown: Head of Marketing. Audience when finished: founders.

## 1. The loop

Growth OS exists to close one loop: what goes out to the market comes back as a measured result and changes what goes out next. Everything else in this document is a place where one stage of that loop reads or writes.

### The five stages

| Stage | Who or what | Reads | Writes | Cadence |
|---|---|---|---|---|
| 1. Observe | Pull script, read-only | HubSpot (deals, meetings, emails, calls, sequence events), Outreach (UK), public need datasets, content assets | Raw cache in GTM Project | Weekly, Sunday night |
| 2. Learn | Tagging and ranking agent, on a git branch | The cache, the codebook, last week's truth | Weekly digest, proposed changes to truth files, re-ranked hooks with outcomes | Weekly, after the pull |
| 3. Decide | Caelum | The branch diff | Truth files on `main`, a decision entry, corrections into agent specs | Monday, 20 minutes |
| 4. Act | SDRs, marketing, the machines | Ranked hooks, persona playbooks, lists, briefs | Messages out, each stamped with hook ID, persona, variant | Per campaign |
| 5. Measure | Automatic | Outcomes carrying the stamp | Scorecard, attribution table, experiments results | Weekly, with the pull |

Stage 5 is next week's stage 1.

### Four requirements for the loop to learn rather than accumulate

1. **The stamp.** Every message out carries `hook_id`, `persona`, `variant`, `sequence_id`, written to the HubSpot enrolment, email or deal as properties. Status today: does not exist. SDRs self-author sequences and nothing records which idea a school responded to. Creating the properties is a HubSpot write and waits for the decision. Until then the loop runs in **retrospective mode**: the pass tags what schools said after the fact. Cycle zero is retrospective; cycle one onwards is prospective. The results page states which mode each number came from.
2. **The join.** Outcomes must be recorded to be joined. Status today: meeting outcomes are not recorded (70 percent of past discovery meetings still read scheduled, 3 Sep funnel analysis). Replies and won deals can be joined now; the middle of the funnel is blind until the meeting-writeback agent runs. That agent is specced in this build and runs after the decision.
3. **The promotion rule.** A hook moves up or down the ranking only past a minimum sample, and the threshold is written in `creative-testing/README.md`, not held in anyone's head. Reason: the quote-harvest work showed a 53 percent lift on 30 schools become one point at scale.
4. **The rebuild.** Weekly runs patch the truth files. Once a quarter the full pass reruns from raw and the truth file is rewritten, not patched. Patching accumulates drift; rebuilding checks whether last quarter's ranking survives.

### Three loops layered

- **Outcome loop**: the market teaches the repo which hooks work. Compounds revenue.
- **Correction loop**: Caelum teaches the agents. Every rejection at stage 3 is written into the agent's spec under "what breaks it". Compounds judgement. This is Isenberg's "train agents like a new hire" made literal.
- **Rebuild loop**: the repo checks itself quarterly. Keeps the other two honest.

### Milestone

The switch from retrospective to prospective is a dated event, not a gradual drift. Target: first stamped sequence enrolled in the week after the decision, whenever that is. Until then every output is labelled cycle zero.

## 2. Purpose and readers

Growth OS is Wellio's marketing memory: one repo holding what the market has said, the machines that act on it, the job spec of every agent, and the results.

Two readers, both served:

1. Caelum and Claude working in it daily. Every file re-runnable or re-derivable from a cited source.
2. A founder or the Head of Marketing opening it cold. Five minutes from the root README to knowing what was built and what it found.

The second reader is why this exists now. The founders' objection is that they do not know what Caelum has built or how it translates. The repo is the translation, and its honesty about what is not yet measured is the seniority signal.

### Success criteria, binary

- Root `README.md` and `results/README.md` read cold in under five minutes, timed.
- The customer-truth pass has run end to end on real data, produced `customer-truth/what-the-market-is-telling-us.md`, and a fresh Claude session reproduces the summary table from `method.md` within rounding.
- The weekly pull has run at least twice on schedule and produced two digests.
- Every agent has a spec in the standard format; at least one is running, one is dry-run.
- Zero HubSpot writes, zero sends, zero moved files in GTM Project. Verified by the checks in section 13.
- Done by 13 October 2026.

## 3. Principles

- **Memory over chats.** Findings live in files with a source line. Unfiled chat output does not exist.
- **Distil, do not duplicate.** GTM Project is the workshop. Growth OS holds the finding and the path to its source. No raw data copied in.
- **Job specs for agents.** Same eight-section format before anything runs.
- **Measure pipeline, not activity.** Replies, meetings, pipeline, wins. Sends and opens are diagnostics.
- **One working machine.** The weekly pull and tagging runs end to end before anything else runs.
- **Read-only until the decision.** Production portal 20058914 only. The sandbox is never a source.
- **Precision, human on every first send.** From Five Engines section 5.
- **Plugin is commodity, repo is taste.** The gtm-skills plugin supplies frameworks. Customer truth and markets are the differentiator and are never outsourced to a framework.
- **Aggregate, never name.** Sequence and rep performance is reported by sequence, persona and market, never by individual. The repo is not a performance review of the SDR team.

## 4. Location and tooling

- Repo: `~/Documents/growth-os`, own git history (created 16 Sep, first commit d4ba052). To be pushed to a **private** GitHub repo before the Head of Marketing is given access, and before any cloud-scheduled job reads it.
- Plugin: `gtm-skills@gtm-plugins` v2.6 installed 16 Sep at **project scope** only (`.claude/settings.json` in the repo). User-level settings untouched, so its 54 skills cannot trigger in renewals or exec-report sessions.
- `CLAUDE.md` at the repo root maps the plugin's expected workspace files onto Growth OS folders: brand and voice to `customer-truth/language.md`, progress log to `decisions/`, content to `content-engine/`. The plugin's bootstrap skill is **not** run; it would create a second workspace.
- Scripts for Growth OS machines live in GTM Project under `scripts/growth_os/`, beside the caches. Growth OS holds `method.md` files saying how to run them.
- Large outputs stay in GTM Project under `outputs/customer_truth/` and `outputs/growth_os/`. Growth OS holds summaries and curated quotes.
- Secrets: never in the repo. The HubSpot token stays in `~/.hubspot_prod_env`. A `.gitignore` blocks `*.env`, `*.jsonl`, `outputs/`.
- Schedule, before the decision: a launchd job on Caelum's Mac runs the pull script and a headless Claude Code session on Sunday night. After the decision: repo on GitHub, pull moved to n8n or a cloud routine with the token as a secret.

## 5. Layout

```
growth-os/
  README.md                    five-minute read: what this is, what has run, the one finding
  CLAUDE.md                    how Claude works here; plugin file mapping; read order
  .claude/settings.json        gtm-skills enabled, project scope
  .gitignore
  customer-truth/
    README.md                  the hook in one paragraph, then the evidence
    what-the-market-is-telling-us.md   approved truth; rewritten quarterly, patched weekly
    weekly/2026-W39.md ...     auto-written digests; what changed, proposed truth changes
    hook-codebook.md           tags, definitions, trigger phrases, version history
    language.md                words schools use vs words we use; phrase bank by persona
    objections.md              objection taxonomy from loss notes; the counter that worked
    buyer-and-champion.md      who raises it, who signs; by segment and state
    competitors.md             every named alternative and what the school said
    segments.md                Catholic, government, independent, by state
    method.md                  how the pass runs; inputs; freshness; rerun log
    quotes/                    curated, de-identified, by hook and role
  markets/
    README.md                  compliance vs discretionary axis; why it decides channel
    au.md  uk.md  europe.md  canada.md  international.md
    need-data-sources.md       datasets per market: what, granularity, access, URL, verified?
  outbound-engine/
    README.md                  what exists, what does not (EDM engine)
    sequence-inventory.md      every AU and UK sequence: copy, persona, enrolments, reply, meeting, attribution
    hooks-and-language.md      ranked hooks per persona; subject lines that earned replies; avoid list
    personas/                  deputy principal, head of wellbeing, principal, head of junior school, business manager
    signals-layer.md           trigger events, sources, detection method
    lists/                     SAM 720, clean bulk 208, primary attach 202, need-scored; CSV with source and date
    suppression-rules.md       prior-no gate, compliance flags, opt-outs
    edm-engine-spec.md         the marketing-owned engine: segments, cadence, governance, deliverability
    sequences/                 designs only; E1 to E3 from sdr-outreach
  content-engine/
    README.md                  a brief starts from a tagged hook, never a topic
    inventory.csv              every existing piece with performance
    proof-library.md           quotes and figures cleared for external use
    voice.md                   derived from language that converted
    briefs/                    one file per brief, hook ID in the header
    repurposing-pipeline.md    one webinar to post, email, one-pager, three LinkedIn pieces
    aeo-map.md                 AI-search visibility: questions, current answers, target pages
  creative-testing/
    README.md                  what counts as an experiment; the minimum sample rule
    ledger.csv                 hypothesis, variant, metric, n needed, n reached, result, decision
    experiments/               one file per experiment
  agents/
    README.md                  spec format; status table (specced, dry-run, running, retired)
    _spec-template.md
    weekly-pull-and-tag.md     the working machine
    signals-researcher.md      the sdr-outreach skill, specced
    meeting-writeback.md       first post-decision agent
    need-scanner-eu-ca.md      stretch
    sequence-analytics.md      inventory and attribution
  results/
    README.md                  one page a founder reads
    metric-definitions.md      each metric: definition, HubSpot carrier, measurable today?
    baseline-2026-09.md        numbers already held, sourced and dated
    scorecard/2026-W39.md ...  weekly, auto-generated
    attribution.md             which hook, sequence or content touched each won deal
    learned/2026-10.md ...     monthly, three lines
  decisions/
    README.md                  format; pending questions register
    2026-09-16-standalone-repo.md
    2026-09-16-first-machine.md
    2026-09-16-plugin-scope.md
  docs/superpowers/            this spec and the plan
```

## 6. Folder contents in detail

### customer-truth

The weekly pull reads what changed in seven days: meetings and notes, email threads, call notes, deal stage changes, new wins and losses. Claude tags the new text against the codebook and writes `weekly/<week>.md`: new wins and losses with their tags, notable quotes, anomalies, proposed changes to the truth files. The truth files change only when Caelum merges.

Every finding in a truth file has the same shape: claim, n, up to three quotes in the school's words, source IDs (deal or company IDs, internal file only), date, confidence (high, medium, thin). Quotes shown to anyone outside Caelum carry school type and state, never the name.

### markets

One file per market, same headings: footprint, buyer and signer, average deal and what drives it, compliance or discretionary, statutory hook if any, public need data available, entry route, what we do not know. AU and UK written from known facts. Europe, Canada, international start as honest stubs. `need-data-sources.md` lists SA WEC, NSW TTFM, Vic AtoSS, My School, Ofsted personal development, FSM percentage, MHST coverage, Ontario school climate, BC Student Learning Survey, each marked verified or unverified until opened.

### outbound-engine

Six parts: sequence inventory with attribution; hooks and language; persona playbooks; signals layer; target lists with suppression rules; the EDM engine spec. The inventory is the baseline nobody has measured. AU comes from HubSpot sequences via the token; UK needs an Outreach export or API key. Performance is aggregated by sequence, persona and market, never by rep.

### content-engine

Inventory with performance (HubSpot blog, pages and marketing email via the connector; LinkedIn by export), proof library, voice file derived from converting language, briefs keyed to hook IDs, the repurposing pipeline, and the AEO map using HubSpot's AEO tools. Plugin skills behind it: webinar-content-and-events, content-strategy-and-planning, linkedin, seo-and-aeo-strategy.

### creative-testing

The experiments ledger. An experiment is any controlled comparison: subject lines, hooks to one persona, follow-up timing, Free Preview page variants, conference follow-ups. Zero spend needed. The README states the minimum sample rule and the ab-test-setup plugin skill supplies the arithmetic. Paid creative joins when budget exists.

### agents

Eight-section spec plus an implementation line naming the skill, script or scheduled task that does the work. Status table with dates.

### results

README (one page), metric definitions with measurable-today flags, baseline, weekly scorecard, attribution table, monthly learned entries. Cells that cannot be measured say "not yet measured, because" and name the gap. Plugin skills behind it: data-and-funnel-analytics, executive-dashboard-generator with a no-emoji style override.

### decisions

Date, decision, alternatives, why, who decided, what reverses it. Also the pending questions register carried from the brainstorm.

## 7. The working machine: weekly pull and tag

**Cycle zero (retrospective, weeks 1 to 2).** Population: Sales Pipeline - AU deals closed won or lost, 1 Jan 2025 to run date, roughly 986 deals, about 308 won and 378 lost among the qualified cohort. Inputs already on disk in GTM Project: `outputs/sdr_funnel/*.jsonl` (pulled 3 Sep), `outputs/sdr_history/` (5,513 company timelines), `outputs/sdr_distilled/` (2,881 records). Four stages: assemble early-conversation text per deal (before the deal reached Evaluating); tag against the codebook with the verbatim trigger phrase and who said it; join to outcome, amount, cycle, source, segment; write the file with quotes, thin cells marked, limits stated. Full detail unchanged from v1 section 5 and carried into `customer-truth/method.md`.

**Weekly mode (from week 2).** The same pipeline over a seven-day delta, committed to branch `auto/weekly`, digest written, proposed truth changes listed. Caelum merges Monday.

**Known limits stated in every output.** Meeting outcomes unrecorded; a third of meetings untyped; call dispositions only from Feb 2026; loss labels unreliable (128 of 214 audited), notes used instead.

## 8. Agents: first five

| Agent | Implementation | Target status 13 Oct |
|---|---|---|
| weekly-pull-and-tag | `scripts/growth_os/pull_delta.py` + tagging prompt + launchd | running |
| signals-researcher | sdr-outreach skill | dry-run, awaiting Sales Manager |
| sequence-analytics | `scripts/growth_os/sequences.py` over HubSpot sequence events | dry-run on AU |
| meeting-writeback | spec only; writes, so post-decision | specced |
| need-scanner-eu-ca | Firecrawl skills + public dataset scripts | specced; dry-run on one province if time |

Remaining Five Engines agents listed in `agents/README.md` with one line each.

## 9. Data access register

| Need | Unblocks | Owner | Status 16 Sep |
|---|---|---|---|
| HubSpot private app: contacts read scope restored; sequences and marketing email read added if available. Preferably a second, read-only private app for Growth OS | Weekly pull, sequence inventory | Caelum (portal admin if needed) | Lapsed ~4 Sep |
| Outreach API key or CSV export of UK sequences with performance | UK sequence inventory | Caelum, UK sales lead | Not requested |
| Webinar recordings location and transcripts | Content inventory, repurposing | Head of Marketing | Unknown |
| Drive folder for case studies, decks, conference kits, or a yes to searching Drive via connector | Content inventory, proof library | Head of Marketing | Not requested |
| Founders' LinkedIn content, export or public pull | Content engine | Caelum | Not requested |
| Search Console or GA access or export | AEO map, content performance | Head of Marketing | Not requested |
| List of AU sequences that matter | Sequence inventory | Sales Manager | Not requested |
| Blazer platform feedback quotes | Proof library | Caelum, method exists | Available |

Nothing in weeks 1 and 2 depends on anything except the first row, and even that has a fallback: cycle zero runs entirely on the 3 September cache.

## 10. Governance and risk

Caelum has no track record in this role to absorb a mistake. These controls exist so the build cannot embarrass him.

| Risk | Control |
|---|---|
| Accidental HubSpot write | Dedicated read-only token where possible; every script grepped for write endpoints before first run; connector used with read tools only; `method.md` logs the portal ID per run |
| School or student data leaks via the repo | Repo private; no student data ever enters it; founder-facing files de-identified; `.gitignore` blocks caches and JSONL; a grep check against company names before any commit to `main` |
| Over-claiming | Every number has a source line and an n; a claims register in `results/README.md` lists every headline figure with its source; thin cells marked |
| Founders see half-built work | `main` is always presentable; automated work lands on `auto/*` branches; README states status honestly, including what has not run |
| SDR team feels surveilled | Aggregate by sequence, persona, market; never by rep; Sales Manager sees outbound-engine material before any SDR does |
| Plugin skills intrude on day job | Project scope only; verified in user settings |
| Scope creep across six folders | One machine runs; other folders hold specs and templates until it does; any addition needs a decisions entry |
| Time | Timebox five hours a week outside the Monday review; week 2 is the only hard dependency; need scanner is dropped first if it slips |
| Sponsor surprised | Head of Marketing sees the repo at the end of week 1 and every Friday after, 15 minutes; nothing goes to a founder before he has seen it |
| Loop learns noise | Minimum sample rule written down; quarterly rebuild; codebook versioned with every output |

## 11. Out of scope

Any HubSpot write, send or enrolment. Moving or renaming GTM Project files. Events and conferences (a decisions entry records the gap). UK and international customer truth (method says how to extend). Running the plugin's bootstrap. Paid creative. The Five Engines artifact itself, linked not rewritten.

## 12. Sequence

| Week | Dates | Deliverables | Gate to call it done |
|---|---|---|---|
| 1 | 16 to 22 Sep | Scaffold, CLAUDE.md, .gitignore, all READMEs, decisions entries, migration of existing findings, codebook v1, private GitHub repo, pull script written and dry-run on the cache, Head of Marketing shown the repo Friday | Citation check passes; five-minute read timed; sponsor has seen it |
| 2 | 23 to 29 Sep | Cycle zero run end to end; truth file written; method file complete; first weekly delta run on Sunday 27 Sep | Rerun check passes in a fresh session; de-identification check passes |
| 3 | 30 Sep to 6 Oct | Five agent specs; sequence inventory on AU; markets stubs; need-data-sources verified; second weekly run; need scanner dry-run if time | Second digest exists; every spec has all eight sections |
| 4 | 7 to 13 Oct | Results page final with claims register; attribution table cycle zero; case study written from the repo; README rewritten for the cold read | All section 13 checks pass; case study reviewed by the Head of Marketing |

Day job runs alongside. If week 2 slips, week 3 slips with it and the need scanner goes first.

## 13. Verification

- **Citation check.** `scripts/growth_os/check_citations.py` lists any heading-level finding in customer-truth, markets or results without a `Source:` line. Zero tolerated on `main`.
- **Read-only audit.** Grep every script for write endpoints and connector write tools; log the portal ID per run.
- **Rerun check.** A fresh Claude session reproduces the summary table from `method.md` within rounding.
- **De-identification check.** Grep founder-facing files against the company names in the deals file. Zero hits.
- **Five-minute check.** Timed cold read of README and results page.
- **Path check.** No file moved in GTM Project; `git -C ~ status` on that folder shows only expected modifications.
- **Scope check.** User-level `enabledPlugins` does not contain gtm-skills.
- **Branch check.** No automated commit lands on `main` without a merge by Caelum.

## 14. Migration list

| Source in GTM Project | Lands in |
|---|---|
| `docs/gtm-strategy-2026-09/01-hubspot-findings.md` | results/baseline, customer-truth/objections |
| `docs/gtm-strategy-2026-09/03-market-research-brief.md` | markets/au, markets/need-data-sources |
| `docs/gtm-strategy-2026-09/04-strategy-working-draft.md` and artifact | results/README link, outbound-engine/README |
| `docs/revops/2026-09-03-sdr-funnel-leak-analysis.md` | results/baseline, results/metric-definitions |
| `docs/revops/2026-08-20-sam-tof-synthesis.md` | markets/au, outbound-engine/lists, signals-layer |
| `docs/revops/2026-08-25-two-motions-analysis.md` | customer-truth/buyer-and-champion |
| `docs/revops/2026-08-26-distillation-findings-log.md` | customer-truth/objections |
| `docs/revops/2026-08-20-outbound-sequence-design.md` | outbound-engine/sequences |
| `docs/revops/2026-09-05-sdr-outreach-pilot-assessment.md` and brief | outbound-engine/README, agents/signals-researcher |
| `outputs/primary_attach_2026-09-05/` | outbound-engine/lists, results/baseline |
| Exec report curated quotes | customer-truth/quotes, content-engine/proof-library, marked renewal-side |
| Brainstorm Q13, Q14 | buyer-and-champion, hook-codebook |
| gtm-skills plugin skill list | agents/README implementation column |

## 15. Open questions

Affecting this build: whether primary attach travels with the role (default: presented as list-building capability, owner unstated); Outreach access (UK inventory only); AU sequence baseline (week 3 if token scopes allow, else "not yet measured").

Carried to `decisions/README.md` as pending: international pricing, tool inventory per region, external challenge brief, events budget, UK meeting rate, decision deadline, other candidate, wellbeing-to-attendance evidence, webinar location, numeric 2027 target.
