# Events intelligence: design

Date: 18 September 2026. Status: approved, ready for an implementation plan.
Owner: Caelum. Sponsor: Head of Marketing. Final audience: the founders.
Predecessor specs: `2026-09-16-growth-os-design.md`, `2026-09-18-term4-outbound-case-design.md`, `2026-09-18-message-map-design.md`.

## 1. Why this exists

Two founder cold reads named the same gap. The role covers outbound, events and new market entry; the work so far covers outbound. Events is the part Caelum has never done, and the company relies on it: 240 event rows sit in the 2026 planning workbook, and a single conference, NSWSPC, is associated with 30 won deals worth $182,468.

A prototype run showed the gap is not knowledge, it is measurement. Nobody has joined the planning workbook to HubSpot, so nobody knows what any event returned against what it cost. Doing that turns Caelum's weakest area into the one where he brings something the company does not have.

## 2. Decisions already taken

| Question | Decision |
|---|---|
| Framing | A method, illustrated by its first findings. The claim is the measurement, not a judgement of anyone's event choices. Events are run by the Head of Marketing and Sales, and the sponsor routing already agreed applies. |
| Where it lives | A new `events/` folder in the Growth OS repo. This is its own subsystem, not a page in `outbound-engine/`. |
| Who attended | Left out. It is recorded nowhere, and inferring it from lead ownership would produce a document that rates colleagues. It becomes a field to capture going forward. |
| Prediction | No model. Nine to twelve measurable events is far too few. The output is a rubric that combines measured return where it exists with structural factors where it does not. |
| Research tool | WebFetch, which needs no credentials. Firecrawl is not installed and its setup needs the user's own key; it is an optional upgrade, not a dependency. |

## 3. Absolute constraints

- **HubSpot is read-only. No writes, no deletes, no changes of any kind, ever.** Everything here reads local caches. No network call touches HubSpot.
- The published pages carry no school name, no staff name, no email address, and **no named Wellio individual**. The workbook's "Kane Notes" and "HA Notes" columns are internal and are never rendered.
- Python 3.9, standard library only, in `~/Documents/GTM Project/scripts/growth_os/`. Outputs under `outputs/growth_os/`.
- Every `## ` section of every rendered file ends with a `Source:` line. The four existing checks must pass.
- No em-dashes, sentence case headings, every figure carries a source.
- Thin rule: an event with fewer than 3 associated deals is marked thin and carries no efficiency conclusion.

## 4. What the data actually supports

Measured on 18 September 2026, so the design does not over-promise:

- The workbook holds 240 event rows across five sheets, 152 with a URL. By market: 122 AU rows with 80 URLs and 25 booked or in progress; 55 UK rows with 35 URLs and 17 booked; 63 international rows with 37 URLs and 4 booked. Rows include noise such as criteria lines and stray notes, so normalisation must filter rather than trust.
- The cached deal file is the Australian pipeline only, 1,018 deals, 1,011 of them carrying a `campaign_source` across 102 campaigns. **UK and international events cannot be measured from this cache at all** and must be listed as unmeasurable rather than quietly dropped.
- Twelve AU events reconcile to campaigns today. Nine of those have a cost in the workbook. That is the entire evidence base for return.
- Campaign naming is inconsistent, which is itself a finding: CASPA appears as "2024 CASPA Campaign", "2025 CaSPAC" and "AU 2026 - CASPA"; VCSSDPA claims a deal and has no campaign at all; PESA claims 8 deals and has 14, all lost.

## 5. What gets built

### 5.1 The normalised event table, `events/data/events.csv`

One row per event, committed and hand-editable, so correcting a cost or a delegate count is a one-line edit rather than a code change. Columns:

`event_key, name, market, acronym, audience, date_text, date_start, location, state, expected_delegates, cost_aud, tier, status, speaking_slot, delegate_list, url, source_sheet`

`event_key` is a slug derived from the name, stable across runs, and is the join key everywhere else. `expected_delegates` parses "420 Delegates" and "Curated group; number TBC" to an integer or blank. `speaking_slot` and `delegate_list` normalise to `yes`, `no` or `unknown`, since the workbook uses `Yes`, `No`, `?` and blanks. `state` is derived from `location` for AU rows. Rows whose name is clearly not an event, such as criteria lines, bare URLs and fragments under four characters, are excluded, and the count excluded is reported.

### 5.2 The alias file, `events/data/campaign-aliases.csv`

`event_key, campaign_source` with one row per known campaign spelling. Seeded from the matching already prototyped and extended by hand. This is the layer that makes attribution reproducible instead of archaeological, and it is data rather than code precisely so Sales can correct it.

Any campaign naming a conference-like token with no alias row appears in the exceptions list rather than being silently unattributed.

### 5.3 Generated: `events/returns.md`

The reconciliation and the returns table.

Per event with a matched campaign: deals created, deals won, revenue, leads, qualified meetings, cost, cost per win, cost per qualified meeting, and return as a multiple of cost. Plus the reconciliation against the workbook's own `2025 Performance` column, showing where the sheet and HubSpot disagree and by how much.

The exceptions list is part of the output, not an appendix: events claiming performance with no campaign, campaigns naming an event with no alias, and events with spend and no wins.

### 5.4 Generated: `events/catalogue.md`

One profile per event: market, audience, type, expected delegates, cost, location, tier, status, speaking slot, delegate list, and the enriched fields from section 5.5 where they exist. Grouped by market, with UK and international marked clearly as listed but unmeasurable from the current cache.

### 5.5 Generated: `events/efficiency.md`

Return normalised so a 70-delegate event is not judged against a 500-delegate one: cost per delegate, wins per hundred delegates, qualified meetings per hundred delegates, and cost per qualified meeting. Only events clearing the thin rule get an efficiency verdict.

This file also carries the one comparison the evidence can actually support: whether events where the delegate list was obtained returned more than events where it was not, and the same for a speaking slot. With nine to twelve events these are hypotheses with numbers attached, not findings, and the file says so in those words.

### 5.6 Research enrichment, `events/data/research.csv`

Scoped to AU events plus everything booked or in progress in any market, roughly forty rows, all of which have a URL. For each, fetched from the event's own site: what kind of event it is, who actually attends by role, whether exhibiting includes the delegate list, whether speaking is sold separately, the organiser, and confirmed attendance where published.

Every enriched field is stored with the URL it came from, in a `<field>_source` column, so a wrong value can be traced and corrected. A field that cannot be found is left blank rather than guessed. Fetching is one call per event with a fixed prompt, results cached to the CSV so a rerun does not refetch.

### 5.7 Hand-written: `events/scoring.md`

The rubric for deciding which events to back. Six dimensions, each scored and each with its evidence or its absence stated:

| Dimension | Source |
|---|---|
| Audience fit | Whether the audience is secondary, primary or K12, against where the company actually wins. Primary deals win at 30 percent against 46 for secondary. |
| Access | Whether the delegate list is obtained. Without it there is no follow-up, only booth conversations. |
| Voice | Whether a speaking slot is included. |
| Efficiency | Cost per delegate, and cost per qualified meeting where measured. |
| Market need | State penetration, since an event in a state where the company is under-penetrated is worth more than the same event where the schools are already customers. Drawn from `markets/au.md`. |
| Measured return | Where it exists. Absent for most events, and the rubric says so rather than scoring a blank as zero. |

The rubric produces a band, not a number to two decimal places, because the inputs do not support that precision.

### 5.8 Hand-written: `events/what-to-record.md`

The short list of fields to capture from the next event onward so that next year's answer is better than this one: who attended, whether the delegate list arrived and when, how many conversations were had, the campaign name used, and a single consistent campaign naming convention. This is the piece that compounds, and it is the cheapest thing in the whole design.

### 5.9 `events/README.md`

What the folder is, how to read it, what it can and cannot see, in five lines.

## 6. Architecture

One module, `scripts/growth_os/events.py`, following the established shape: pure functions, a `summarise`, render functions per output, a `main`. Tests in `tests/test_events.py` on hand-built fixtures.

It reuses `common.py` for paths, parsing and outcome rules. It reads the workbook through a dated snapshot rather than from Downloads: `main` copies the source file to `outputs/growth_os/events_snapshot_<date>.xlsx` and reads that, so a number in a published page can always be traced to the version of the sheet it came from. The workbook path is a parameter with the Downloads location as its default.

`openpyxl` is already installed and already used elsewhere in the project, so reading the workbook adds no new dependency. Nothing else outside the standard library is introduced.

Research uses WebFetch, driven by the implementing agent rather than by the module, with results written to `events/data/research.csv`. The module reads that CSV if present and renders the enriched fields; if it is absent the catalogue renders from workbook fields only. That keeps the module deterministic and offline.

## 7. Verification

- The full suite passes, currently 192 tests.
- All four checks print OK before any commit.
- The normalisation is reconciled: rows in, rows kept, rows excluded, and the excluded reasons, printed on every run and rendered in the catalogue.
- Every figure in the hand-written files traces to `events.csv`, `research.csv` or a generated page. A reviewer spot-checks at least four.
- The attribution is reproducible: rerunning after an alias edit changes the numbers in the expected direction, tested on a fixture.
- No named Wellio individual appears in any file under `events/`. Checked by grep against the notes column headers and the owner names in the cache.

## 8. Sequence

| Step | Deliverable | Gate |
|---|---|---|
| 1 | `events.py` normalising the workbook to `events.csv`, with the exclusion report | Rows reconcile, noise excluded, states derived |
| 2 | Alias file and the returns page, including the reconciliation and exceptions | Twelve known events match, the three known mismatches appear as exceptions |
| 3 | Efficiency page and catalogue | Thin rule applied, UK and international marked unmeasurable |
| 4 | Research enrichment for about forty events | Every enriched field carries its source URL, blanks where not found |
| 5 | Scoring rubric and what to record, hand-written | Rubric bands, not false precision; every dimension names its evidence or its absence |

Steps 1 to 3 are independent of step 4 and ship without it.

## 9. Out of scope

Who attended, which is deferred to `what-to-record.md`. UK and international return, which the cache cannot support. Any change to how events are run or booked. Any HubSpot write, including creating the campaign naming convention the analysis will recommend. And a predictive model, which the sample size does not permit and which this design deliberately refuses to fake.
