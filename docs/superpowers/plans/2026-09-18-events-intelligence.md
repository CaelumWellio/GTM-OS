# Events Intelligence Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** An `events/` folder that says what each conference returned against what it cost, what kind of event each one is, and a rubric for deciding which to back, built from the 2026 planning workbook joined to HubSpot.

**Architecture:** One module, `events.py`, normalises the workbook into a committed CSV, joins it to cached HubSpot deals and leads through an editable alias file, and renders three pages. Research enrichment is a separate resumable script writing a second CSV that the module reads if present. The rubric and the recording guidance are hand-written, because the judgement is the point.

**Tech Stack:** Python 3.9 standard library plus `openpyxl` 3.1.5, already installed and already used in this project. Firecrawl v2 over `urllib` for research. pytest 8 via `python3 -m pytest`.

## Global Constraints

- **HubSpot is read-only. No writes, no deletes, no changes of any kind, ever.** Nothing here calls HubSpot at all; it reads local caches.
- Firecrawl is called only by the research script in Task 4, and only with `POST /v2/scrape` and `POST /v2/search`. The key is read from `~/.firecrawl_env` via the environment, never printed, never written to any file, never committed.
- Python 3.9: module docstring first, then `from __future__ import annotations`; `Optional[...]`; no `match`; no `X | Y` at runtime.
- Paths come from `common.py` constants. New outputs go under `outputs/growth_os/`.
- **No named Wellio individual in any rendered file.** The workbook's `Kane Notes` and `HA Notes` columns are internal and are never read into any output.
- No school name, no staff name, no email address in any rendered file.
- Every `## ` section of every rendered file ends with a `Source:` line. `checks.py citations` must be extended to cover `events/*.md`.
- No em-dashes, sentence case headings, every figure carries a source.
- Thin rule: an event with fewer than 3 associated deals is thin and gets no efficiency verdict.
- Git: code commits to the home repo staging only `Documents/GTM Project/scripts/growth_os`; content commits to `~/Documents/growth-os` and are pushed. Every commit message ends with `Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>`.
- Never delete or clean anything under `outputs/`. Fixtures use `tmp_path`.

## Domain facts (verified 18 September 2026)

- Workbook `/Users/cleonard1998/Downloads/2026 Conference Planning (1).xlsx`, five sheets: `AU Events`, `UK Events`, `International Events`, `AU Revised - KM`, `2026 Events`.
- Columns on `AU Events` and `AU Revised - KM`: Tiers, Status, Conference, Audiences, Date, Location, Expected Attendance, 2025 Performance, Costs, Speaking Slot, Delegate List, Contact Status, Links, Notes, Kane Notes. `UK Events` is the same without `2025 Performance`, with `Costs (inc VAT)` and `HA Notes`. `International Events` adds `Score` and `Network`. `2026 Events` has only Market, Conference name, Date, Location, Status, Event URL, Costs AUD$.
- 240 event rows, 152 with a URL. AU 122 rows, 80 with URL, 25 booked or in progress. UK 55 rows, 35 with URL, 17 booked. International 63 rows, 37 with URL, 4 booked. Rows include noise such as `Criteria`, `6 foot table`, `249 just to exibhit` and bare URLs.
- `Expected Attendance` values look like `420 Delegates`, `30 Participants`, `100 Attendees`, `400+`, `Curated group; number TBC`, `?`.
- `Costs` values look like `1000.0`, `$220`, `8800.0`, `6000?`, `AED 30,000`, `?`.
- `Speaking Slot` and `Delegate List` values are `Yes`, `No`, `?`, `-`, blank, and occasionally a sentence such as `Yes (no emails)` or `5-minute introduction at w`.
- `2025 Performance` values look like `1 Deal`, `6 Deals`, `29 Deals`, `-`, `?`, and once `Underperformed: 2 conversations and broke even`.
- `outputs/sdr_funnel/deals.jsonl`: 1,018 rows, all pipeline `41942802`, 1,011 with a `campaign_source`, across 102 distinct campaigns.
- `outputs/sdr_funnel/leads.jsonl`: 3,548 rows; qualified is `hs_v2_date_entered_qualified_stage_id_233247981` being non-empty; 2,566 carry a `campaign_source`.
- Known campaign spellings that must be aliased: `2025 NSWSPC` and `AU 2026 - NSWSPC`; `2025 WASSEA`; `2025 ASA Conference`; `2025 PESA Conference`; `AU 2026 - QASSP Conference`; `2024 CASPA Campaign`, `2025 CaSPAC` and `AU 2026 - CASPA`; `QSPA 2025`; `2025 NSWSDPA` and `AU 2026 - NSWSDPA`; `AU 2026 - VASSP`; `2025 T1 SEA EARCOS`.
- Prototype result to reproduce: NSWSPC 75 deals 30 won $182,468; WASSEA 20 deals 13 won $119,640; ASA 18 deals 7 won $66,745; PESA 14 deals 0 won; QSPA 7 deals 5 won $46,889; NSWSDPA 6 deals 2 won $19,271; HICES 5 deals 1 won; SAWLIS 3 deals 1 won $1,000.
- Firecrawl v2 verified working: `POST https://api.firecrawl.dev/v2/scrape` with `{"url":..., "formats":["markdown"]}` returns 200 and renders JavaScript pages. `POST /v2/search` with `{"query":..., "limit":4}` returns `data.web` as a list of `{title, url}`. Search results are frequently irrelevant, so a domain relevance gate is required.
- Existing suite: 192 tests, `python3 -m pytest scripts/growth_os/tests -q` from `/Users/cleonard1998/Documents/GTM Project`.

## File structure

| Path | Responsibility |
|---|---|
| `scripts/growth_os/events.py` | Normalise the workbook, join HubSpot through aliases, render three pages |
| `scripts/growth_os/tests/test_events.py` | Unit tests on `tmp_path` fixtures |
| `scripts/growth_os/event_research.py` | Firecrawl enrichment, resumable, writes `research.csv` |
| `scripts/growth_os/checks.py` | Extend citations to `events/*.md` |
| `growth-os/events/data/events.csv` | Normalised event table, committed, hand-editable |
| `growth-os/events/data/campaign-aliases.csv` | Event key to campaign name, committed, hand-editable |
| `growth-os/events/data/research.csv` | Enriched fields with per-field source URLs |
| `growth-os/events/returns.md`, `catalogue.md`, `efficiency.md` | Generated |
| `growth-os/events/scoring.md`, `what-to-record.md`, `README.md` | Hand-written |

---

### Task 1: Normalise the workbook

**Files:**
- Create: `/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/events.py`
- Create: `/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/tests/test_events.py`
- Create (generated): `/Users/cleonard1998/Documents/growth-os/events/data/events.csv`

**Interfaces:**
- Consumes: `common.py` (`GTM`, `GROWTH_OS`, `GOS_OUT`, `FUNNEL`, `read_jsonl`, `parse_ts`, `ids`, `outcome`, `is_new_business`, `SALES_PIPELINE_AU`)
- Produces, used by Tasks 2 to 5:
  - `EVENT_COLUMNS: List[str]`
  - `slug(name: str) -> str`
  - `parse_int(text) -> Optional[int]`
  - `parse_money(text) -> Optional[float]`
  - `tri(text) -> str` returning `"yes"`, `"no"` or `"unknown"`
  - `parse_claimed_deals(text) -> Optional[int]`
  - `state_of(location) -> str`
  - `is_event_row(name) -> bool`
  - `normalise(workbook_rows) -> Tuple[List[dict], Dict[str, int]]` returning rows and an exclusion tally
  - `load_workbook_rows(path) -> List[dict]`
  - `main(argv)`

- [ ] **Step 1: Write the failing tests**

Create `scripts/growth_os/tests/test_events.py`:

```python
from __future__ import annotations
from scripts.growth_os import events as ev


def test_slug_is_stable_and_safe():
    assert ev.slug("NSWSPC Conference") == "nswspc-conference"
    assert ev.slug("CaSPA National Conference") == "caspa-national-conference"
    assert ev.slug("BSME Annual School Leaders’ Conference") == "bsme-annual-school-leaders-conference"
    assert ev.slug("  Spaced   Out  ") == "spaced-out"


def test_parse_int_handles_the_real_formats():
    assert ev.parse_int("420 Delegates") == 420
    assert ev.parse_int("30 Participants") == 30
    assert ev.parse_int("100 Attendees ") == 100
    assert ev.parse_int("400+") == 400
    assert ev.parse_int("1100 Delegates") == 1100
    assert ev.parse_int("Curated group; number TBC") is None
    assert ev.parse_int("?") is None
    assert ev.parse_int(None) is None


def test_parse_money_handles_the_real_formats():
    assert ev.parse_money(1000.0) == 1000.0
    assert ev.parse_money("$220") == 220.0
    assert ev.parse_money("8800.0") == 8800.0
    assert ev.parse_money("6000?") == 6000.0
    assert ev.parse_money("AED 30,000") == 30000.0
    assert ev.parse_money("?") is None
    assert ev.parse_money("") is None


def test_tri_normalises_the_messy_values():
    assert ev.tri("Yes") == "yes"
    assert ev.tri("yes (no emails)") == "yes"
    assert ev.tri("No") == "no"
    assert ev.tri("?") == "unknown"
    assert ev.tri("-") == "unknown"
    assert ev.tri("") == "unknown"
    assert ev.tri(None) == "unknown"


def test_parse_claimed_deals():
    assert ev.parse_claimed_deals("29 Deals") == 29
    assert ev.parse_claimed_deals("1 Deal") == 1
    assert ev.parse_claimed_deals("Underperformed: 2 conversations and broke even") is None
    assert ev.parse_claimed_deals("-") is None
    assert ev.parse_claimed_deals(None) is None


def test_state_of():
    assert ev.state_of("Brisbane, QLD") == "QLD"
    assert ev.state_of("Sydney, NSW") == "NSW"
    assert ev.state_of("Perth, WA") == "WA"
    assert ev.state_of("SA") == "SA"
    assert ev.state_of("Online") == ""
    assert ev.state_of(None) == ""


def test_is_event_row_rejects_noise():
    assert ev.is_event_row("NSWSPC Conference")
    assert not ev.is_event_row("Criteria")
    assert not ev.is_event_row("6 foot table")
    assert not ev.is_event_row("249 just to exibhit")
    assert not ev.is_event_row("https://example.com/thing")
    assert not ev.is_event_row("TBC")
    assert not ev.is_event_row("")
    assert not ev.is_event_row(None)


def test_normalise_dedupes_and_counts_exclusions():
    rows = [
        {"sheet": "AU Events", "market": "AU", "Conference": "WASSEA", "Audiences": "Secondary",
         "Expected Attendance": "200 Delegates", "Costs": 9350.0, "Speaking Slot": "Yes",
         "Delegate List": "Yes", "Location": "Perth, WA", "Tiers": 1.0, "Status": "Booked",
         "2025 Performance": "15 Deals", "Links": "https://wassea.asn.au/x", "Date": "13th August 2026"},
        {"sheet": "AU Revised - KM", "market": "AU", "Conference": "WASSEA", "Audiences": "Secondary",
         "Expected Attendance": "200 Delegates", "Costs": 9350.0, "Speaking Slot": "Yes",
         "Delegate List": "No", "Location": "Perth, WA", "Tiers": 1.0, "Status": "Booked",
         "2025 Performance": "15 Deals", "Links": "", "Date": "13th August 2026"},
        {"sheet": "AU Events", "market": "AU", "Conference": "Criteria"},
    ]
    out, excl = ev.normalise(rows)
    assert len(out) == 1
    assert out[0]["event_key"] == "wassea"
    assert out[0]["expected_delegates"] == 200
    assert out[0]["cost_aud"] == 9350.0
    assert out[0]["state"] == "WA"
    assert out[0]["claimed_deals"] == 15
    assert out[0]["url"] == "https://wassea.asn.au/x"
    assert excl["not_an_event"] == 1


def test_normalise_prefers_a_known_value_over_unknown():
    rows = [
        {"sheet": "AU Events", "market": "AU", "Conference": "QSPA", "Delegate List": "?", "Costs": "?"},
        {"sheet": "AU Revised - KM", "market": "AU", "Conference": "QSPA", "Delegate List": "Yes", "Costs": 8800.0},
    ]
    out, _ = ev.normalise(rows)
    assert len(out) == 1
    assert out[0]["delegate_list"] == "yes"
    assert out[0]["cost_aud"] == 8800.0


def test_normalise_never_carries_internal_note_columns():
    rows = [{"sheet": "AU Events", "market": "AU", "Conference": "ASA",
             "Kane Notes": "Gold sponsor", "HA Notes": "x", "Notes": "Incredible returns"}]
    out, _ = ev.normalise(rows)
    assert "Kane Notes" not in out[0] and "HA Notes" not in out[0]
    assert not any("kane" in str(v).lower() for v in out[0].values())
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_events.py -q 2>&1 | tail -3
```

Expected: `ModuleNotFoundError: No module named 'scripts.growth_os.events'`.

- [ ] **Step 3: Write the normalisation half of `events.py`**

```python
"""What each conference returned against what it cost. Reads the planning workbook and the
cached HubSpot objects; makes no network call and never touches HubSpot.

The workbook is copied to a dated snapshot before it is read, so any number in a published page
can be traced to the version of the sheet it came from.
"""
from __future__ import annotations
import argparse
import csv
import datetime as dt
import pathlib
import re
import shutil
import sys
from typing import Dict, List, Optional, Tuple

from . import common as c

WORKBOOK_DEFAULT = pathlib.Path.home() / "Downloads" / "2026 Conference Planning (1).xlsx"
EVENTS_DIR = c.GROWTH_OS / "events"
DATA_DIR = EVENTS_DIR / "data"
INTERNAL_COLUMNS = ("kane notes", "ha notes")
STATES = ("NSW", "VIC", "QLD", "WA", "SA", "TAS", "ACT", "NT")
NOISE = {"tbc", "criteria", "others", "virtual", "not a fit", "delegate lists"}
MIN_DEALS_FOR_VERDICT = 3

EVENT_COLUMNS = ["event_key", "name", "market", "audience", "date_text", "location", "state",
                 "expected_delegates", "cost_aud", "tier", "status", "speaking_slot",
                 "delegate_list", "claimed_deals", "url", "source_sheets"]


def slug(name: str) -> str:
    s = re.sub(r"[‘’“”']", "", str(name or "").strip().lower())
    s = re.sub(r"[^a-z0-9]+", "-", s)
    return s.strip("-")


def parse_int(text) -> Optional[int]:
    if text is None:
        return None
    if isinstance(text, (int, float)) and not isinstance(text, bool):
        return int(text)
    m = re.search(r"\d[\d,]*", str(text))
    if not m:
        return None
    head = str(text).strip()
    if not re.match(r"^\s*\d", head):
        return None
    return int(m.group(0).replace(",", ""))


def parse_money(text) -> Optional[float]:
    if text is None or text == "":
        return None
    if isinstance(text, (int, float)) and not isinstance(text, bool):
        return float(text) or None
    m = re.search(r"\d[\d,]*(?:\.\d+)?", str(text))
    if not m:
        return None
    try:
        v = float(m.group(0).replace(",", ""))
    except ValueError:
        return None
    return v or None


def tri(text) -> str:
    t = str(text or "").strip().lower()
    if not t or t in ("?", "-", "n/a", "na"):
        return "unknown"
    if t.startswith("y"):
        return "yes"
    if t.startswith("n"):
        return "no"
    return "unknown"


def parse_claimed_deals(text) -> Optional[int]:
    m = re.match(r"^\s*(\d+)\s+deals?\b", str(text or ""), re.I)
    return int(m.group(1)) if m else None


def state_of(location) -> str:
    t = str(location or "").upper()
    for s in STATES:
        if re.search(r"\b" + s + r"\b", t):
            return s
    return ""


def is_event_row(name) -> bool:
    t = str(name or "").strip()
    if len(t) < 4 or t.lower().startswith("http"):
        return False
    if t.lower() in NOISE:
        return False
    if re.match(r"^\d", t):
        return False
    return bool(re.search(r"[A-Za-z]{3}", t))


def _pick(old, new):
    """Prefer a known value over an unknown one, and the first known value otherwise."""
    if old in (None, "", "unknown"):
        return new
    return old


def normalise(rows: List[dict]) -> Tuple[List[dict], Dict[str, int]]:
    out: Dict[str, dict] = {}
    excl: Dict[str, int] = {"not_an_event": 0, "no_name": 0}
    for r in rows:
        name = r.get("Conference") or r.get("Conference name")
        if not name:
            excl["no_name"] += 1
            continue
        name = " ".join(str(name).split())
        if not is_event_row(name):
            excl["not_an_event"] += 1
            continue
        key = slug(name)
        rec = out.get(key) or {"event_key": key, "name": name, "market": r.get("market", ""),
                               "audience": "", "date_text": "", "location": "", "state": "",
                               "expected_delegates": None, "cost_aud": None, "tier": "",
                               "status": "", "speaking_slot": "unknown", "delegate_list": "unknown",
                               "claimed_deals": None, "url": "", "source_sheets": ""}
        rec["audience"] = _pick(rec["audience"], str(r.get("Audiences") or "").strip())
        rec["date_text"] = _pick(rec["date_text"], str(r.get("Date") or "").strip())
        rec["location"] = _pick(rec["location"], str(r.get("Location") or "").strip())
        rec["state"] = _pick(rec["state"], state_of(r.get("Location")))
        if rec["expected_delegates"] is None:
            rec["expected_delegates"] = parse_int(r.get("Expected Attendance"))
        if rec["cost_aud"] is None:
            rec["cost_aud"] = parse_money(r.get("Costs") if r.get("Costs") not in (None, "")
                                          else r.get("Costs (inc VAT)") or r.get("Costs AUD$"))
        rec["tier"] = _pick(rec["tier"], str(r.get("Tiers") or "").strip())
        rec["status"] = _pick(rec["status"], str(r.get("Status") or "").strip())
        rec["speaking_slot"] = _pick(rec["speaking_slot"], tri(r.get("Speaking Slot")))
        rec["delegate_list"] = _pick(rec["delegate_list"], tri(r.get("Delegate List")))
        if rec["claimed_deals"] is None:
            rec["claimed_deals"] = parse_claimed_deals(r.get("2025 Performance"))
        link = str(r.get("Links") or r.get("Event URL") or "").strip()
        if link.startswith("http") and not rec["url"]:
            rec["url"] = link
        sheets = [s for s in rec["source_sheets"].split("|") if s]
        if r.get("sheet") and r["sheet"] not in sheets:
            sheets.append(r["sheet"])
        rec["source_sheets"] = "|".join(sheets)
        out[key] = rec
    return sorted(out.values(), key=lambda x: (x["market"], x["name"])), excl


def load_workbook_rows(path) -> List[dict]:
    import openpyxl
    wb = openpyxl.load_workbook(path, data_only=True)
    rows: List[dict] = []
    for ws in wb.worksheets:
        header = [str(x.value or "").strip() for x in next(ws.iter_rows(max_row=1))]
        market = ("INT" if "International" in ws.title
                  else "UK" if ws.title.startswith("UK") else "AU")
        for raw in ws.iter_rows(min_row=2, values_only=True):
            rec = {"sheet": ws.title, "market": market}
            for i, h in enumerate(header):
                if not h or h.lower() in INTERNAL_COLUMNS or i >= len(raw):
                    continue
                rec[h] = raw[i]
            if rec.get("Market"):
                rec["market"] = str(rec["Market"]).strip().upper()
            rows.append(rec)
    return rows


def write_csv(path, columns: List[str], rows: List[dict]) -> None:
    p = pathlib.Path(path)
    p.parent.mkdir(parents=True, exist_ok=True)
    with p.open("w", newline="") as f:
        w = csv.DictWriter(f, fieldnames=columns, extrasaction="ignore")
        w.writeheader()
        for r in rows:
            w.writerow({k: ("" if r.get(k) is None else r.get(k)) for k in columns})


def snapshot(src, out_dir) -> pathlib.Path:
    src = pathlib.Path(src)
    out_dir = pathlib.Path(out_dir)
    out_dir.mkdir(parents=True, exist_ok=True)
    dest = out_dir / f"events_snapshot_{dt.date.today().isoformat()}{src.suffix}"
    shutil.copy2(src, dest)
    return dest


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="Normalise the conference workbook. Read-only.")
    ap.add_argument("--workbook", default=str(WORKBOOK_DEFAULT))
    args = ap.parse_args(argv)
    snap = snapshot(args.workbook, c.GOS_OUT)
    rows = load_workbook_rows(snap)
    events, excl = normalise(rows)
    write_csv(DATA_DIR / "events.csv", EVENT_COLUMNS, events)
    by_market: Dict[str, int] = {}
    for e in events:
        by_market[e["market"]] = by_market.get(e["market"], 0) + 1
    print(f"snapshot {snap.name} | rows in {len(rows)} | events kept {len(events)} | excluded {excl}")
    print(f"by market {by_market}")
    print(f"WROTE {DATA_DIR / 'events.csv'}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_events.py -q 2>&1 | tail -3
```

Expected: `10 passed`. Then the whole suite: expect `202 passed`.

- [ ] **Step 5: Run it on the real workbook and sanity-check**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.events
```

Expected: roughly 240 rows in, somewhere near 170 to 200 events kept after noise and duplicates, an exclusion tally, and a market split near AU 60 to 70, UK 50, INT 55. Open `events/data/events.csv` and check ten rows by eye against the sheet, including WASSEA, NSWSPC, PESA and VASSP. If a real event was excluded as noise, widen `is_event_row` and say which in your report.

- [ ] **Step 6: Commit**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: normalise the conference workbook into a committed event table

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
cd ~/Documents/growth-os && git add -A && git commit -m "events: the normalised event table

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

---

### Task 2: Aliases, the join, and the returns page

**Files:**
- Modify: `scripts/growth_os/events.py`
- Modify: `scripts/growth_os/tests/test_events.py`
- Modify: `scripts/growth_os/checks.py` (citations target list)
- Create: `/Users/cleonard1998/Documents/growth-os/events/data/campaign-aliases.csv`
- Create (generated): `/Users/cleonard1998/Documents/growth-os/events/returns.md`

**Interfaces:**
- Consumes: `events.csv` from Task 1.
- Produces, used by Task 3:
  - `load_aliases(path) -> Dict[str, List[str]]` mapping `event_key` to campaign names
  - `measure(events, aliases, deals, leads) -> dict` with `per_event`, `exceptions`, `totals`
  - `render_returns(summary) -> str`

- [ ] **Step 1: Seed the alias file**

Create `events/data/campaign-aliases.csv` with header `event_key,campaign_source` and these rows, which are the spellings verified present in the cache:

```csv
event_key,campaign_source
nswspc-conference,2025 NSWSPC
nswspc-conference,AU 2026 - NSWSPC
wassea,2025 WASSEA
asa,2025 ASA Conference
pesa,2025 PESA Conference
qassp-school-leadership-conference,AU 2026 - QASSP Conference
caspa-national-conference,2024 CASPA Campaign
caspa-national-conference,2025 CaSPAC
caspa-national-conference,AU 2026 - CASPA
qspa,QSPA 2025
nswsdpa,2025 NSWSDPA
nswsdpa,AU 2026 - NSWSDPA
vassp-2026,AU 2026 - VASSP
earcos-annual-leadership-conference,2025 T1 SEA EARCOS
```

If a listed `event_key` does not exist in `events.csv`, correct it to the key the normaliser actually produced and note the correction in your report. Do not invent campaign names; only spellings present in `deals.jsonl` or `leads.jsonl` belong here.

- [ ] **Step 2: Write the failing tests**

Append to `scripts/growth_os/tests/test_events.py`:

```python
def deal(camp, won=True, amount="1000", pipeline="41942802", dealtype="newbusiness"):
    return {"id": "d" + camp + str(won) + str(amount), "campaign_source": camp,
            "pipeline": pipeline, "dealtype": dealtype, "amount": amount,
            "hs_is_closed": "true", "hs_is_closed_won": "true" if won else "false",
            "company_ids": ["c1"]}


def lead(camp, qualified=True):
    r = {"id": "l" + camp + str(qualified), "campaign_source": camp, "company_ids": ["c1"]}
    if qualified:
        r["hs_v2_date_entered_qualified_stage_id_233247981"] = "2025-10-01T00:00:00Z"
    return r


def test_load_aliases_groups_by_event(tmp_path):
    p = tmp_path / "a.csv"
    p.write_text("event_key,campaign_source\nwassea,2025 WASSEA\nwassea,AU 2026 - WASSEA\nasa,2025 ASA\n")
    a = ev.load_aliases(p)
    assert a["wassea"] == ["2025 WASSEA", "AU 2026 - WASSEA"]
    assert a["asa"] == ["2025 ASA"]


def test_measure_counts_deals_wins_revenue_and_leads():
    events = [{"event_key": "wassea", "name": "WASSEA", "market": "AU", "cost_aud": 9350.0,
               "expected_delegates": 200, "claimed_deals": 15, "delegate_list": "yes",
               "speaking_slot": "yes", "state": "WA", "audience": "Secondary"}]
    aliases = {"wassea": ["2025 WASSEA"]}
    deals = [deal("2025 WASSEA", True, "5000"), deal("2025 WASSEA", False, "3000"),
             deal("Other Campaign", True, "9000")]
    leads = [lead("2025 WASSEA", True), lead("2025 WASSEA", False)]
    s = ev.measure(events, aliases, deals, leads)
    row = s["per_event"][0]
    assert row["deals"] == 2 and row["won"] == 1 and row["revenue"] == 5000.0
    assert row["leads"] == 2 and row["qualified"] == 1
    assert row["cost_per_win"] == 9350.0
    assert row["claimed_deals"] == 15 and row["claimed_vs_won"] == 14


def test_measure_flags_an_event_with_spend_and_no_wins():
    events = [{"event_key": "pesa", "name": "PESA", "market": "AU", "cost_aud": 4000.0,
               "expected_delegates": 100, "claimed_deals": 8, "delegate_list": "unknown",
               "speaking_slot": "unknown", "state": "", "audience": ""}]
    s = ev.measure(events, {"pesa": ["2025 PESA Conference"]},
                   [deal("2025 PESA Conference", False), deal("2025 PESA Conference", False)], [])
    assert s["per_event"][0]["won"] == 0
    assert any("no wins" in x["issue"] for x in s["exceptions"])


def test_measure_flags_a_claim_with_no_campaign_and_a_campaign_with_no_alias():
    events = [{"event_key": "vcssdpa", "name": "VCSSDPA", "market": "AU", "cost_aud": None,
               "expected_delegates": None, "claimed_deals": 1, "delegate_list": "unknown",
               "speaking_slot": "unknown", "state": "", "audience": ""}]
    deals = [deal("2025 HICES Conference", True)]
    s = ev.measure(events, {}, deals, [])
    issues = " ".join(x["issue"] for x in s["exceptions"])
    assert "no campaign" in issues
    assert "no alias" in issues and "HICES" in " ".join(x["detail"] for x in s["exceptions"])


def test_measure_ignores_non_au_pipeline_and_non_new_business():
    events = [{"event_key": "wassea", "name": "WASSEA", "market": "AU", "cost_aud": None,
               "expected_delegates": None, "claimed_deals": None, "delegate_list": "unknown",
               "speaking_slot": "unknown", "state": "", "audience": ""}]
    deals = [deal("2025 WASSEA", True, "1000", pipeline="999"),
             deal("2025 WASSEA", True, "1000", dealtype="existingbusiness")]
    s = ev.measure(events, {"wassea": ["2025 WASSEA"]}, deals, [])
    assert s["per_event"][0]["deals"] == 0


def test_render_returns_has_source_in_every_section_and_no_internal_names():
    import re
    events = [{"event_key": "wassea", "name": "WASSEA", "market": "AU", "cost_aud": 9350.0,
               "expected_delegates": 200, "claimed_deals": 15, "delegate_list": "yes",
               "speaking_slot": "yes", "state": "WA", "audience": "Secondary"}]
    md = ev.render_returns(ev.measure(events, {"wassea": ["2025 WASSEA"]}, [deal("2025 WASSEA")], []))
    for part in re.split(r"^## ", md, flags=re.M)[1:]:
        assert "Source:" in part
    assert "\u2014" not in md
    assert "Kane" not in md and "HA Notes" not in md
```

- [ ] **Step 3: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_events.py -q 2>&1 | tail -3
```

Expected: failures on `load_aliases`, `measure` and `render_returns` not existing.

- [ ] **Step 4: Add the join and the returns renderer to `events.py`**

Add these imports at the top of the existing import block: `import collections`, `import json`.

```python
def load_aliases(path) -> Dict[str, List[str]]:
    p = pathlib.Path(path)
    out: Dict[str, List[str]] = {}
    if not p.exists():
        return out
    with p.open() as f:
        for row in csv.DictReader(f):
            k = (row.get("event_key") or "").strip()
            v = (row.get("campaign_source") or "").strip()
            if k and v:
                out.setdefault(k, []).append(v)
    return out


CONFERENCE_HINT = re.compile(r"conference|summit|caspa|nswspc|wassea|qspa|qassp|pesa|sdpa|hices|"
                             r"sawlis|earcos|cobis|appa|vassp|asreap|expo|congress", re.I)


def measure(events: List[dict], aliases: Dict[str, List[str]],
            deals: List[dict], leads: List[dict]) -> dict:
    scoped = [d for d in deals if str(d.get("pipeline")) == c.SALES_PIPELINE_AU and c.is_new_business(d)]
    by_campaign: Dict[str, List[dict]] = {}
    for d in scoped:
        by_campaign.setdefault((d.get("campaign_source") or "").strip(), []).append(d)
    leads_by_campaign: Dict[str, List[dict]] = {}
    for l in leads:
        leads_by_campaign.setdefault((l.get("campaign_source") or "").strip(), []).append(l)

    per_event, exceptions, claimed_campaigns = [], [], set()
    for e in events:
        camps = aliases.get(e["event_key"], [])
        claimed_campaigns.update(camps)
        ds = [d for cm in camps for d in by_campaign.get(cm, [])]
        ls = [l for cm in camps for l in leads_by_campaign.get(cm, [])]
        won = [d for d in ds if c.outcome(d) == "won"]
        revenue = 0.0
        for d in won:
            try:
                revenue += float(d.get("amount") or 0)
            except (TypeError, ValueError):
                pass
        qualified = [l for l in ls
                     if c.parse_ts(l.get("hs_v2_date_entered_qualified_stage_id_233247981"))]
        cost = e.get("cost_aud")
        row = dict(e)
        row.update({"campaigns": camps, "deals": len(ds), "won": len(won),
                    "revenue": round(revenue, 2), "leads": len(ls), "qualified": len(qualified),
                    "cost_per_win": round(cost / len(won), 2) if cost and won else None,
                    "cost_per_qualified": round(cost / len(qualified), 2) if cost and qualified else None,
                    "return_multiple": round(revenue / cost, 2) if cost else None,
                    "thin": len(ds) < MIN_DEALS_FOR_VERDICT,
                    "claimed_vs_won": (e["claimed_deals"] - len(won))
                    if e.get("claimed_deals") is not None else None})
        per_event.append(row)
        if e.get("claimed_deals") and not camps:
            exceptions.append({"event": e["name"], "issue": "claims deals but has no campaign alias",
                               "detail": f"{e['claimed_deals']} claimed"})
        if cost and ds and not won:
            exceptions.append({"event": e["name"], "issue": "has spend and no wins",
                               "detail": f"{len(ds)} deals, cost {cost:,.0f}"})
        if row["claimed_vs_won"] not in (None, 0) and camps:
            exceptions.append({"event": e["name"], "issue": "sheet and HubSpot disagree",
                               "detail": f"sheet {e['claimed_deals']}, HubSpot {len(won)} won"})
    for camp, ds in by_campaign.items():
        if camp and camp not in claimed_campaigns and CONFERENCE_HINT.search(camp):
            exceptions.append({"event": "(none)", "issue": "campaign looks like an event but has no alias",
                               "detail": f"{camp}: {len(ds)} deals"})
    per_event.sort(key=lambda r: (-(r["revenue"] or 0), r["name"]))
    return {"generated_at": dt.datetime.utcnow().isoformat(timespec="seconds"),
            "min_deals": MIN_DEALS_FOR_VERDICT,
            "totals": {"events": len(events), "measured": sum(1 for r in per_event if r["deals"]),
                       "deals": sum(r["deals"] for r in per_event),
                       "won": sum(r["won"] for r in per_event),
                       "revenue": round(sum(r["revenue"] for r in per_event), 2),
                       "au_deals_in_cache": len(scoped)},
            "per_event": per_event, "exceptions": exceptions}


def _money(x) -> str:
    return "" if x in (None, "") else f"{float(x):,.0f}"


SRC_RETURNS = ("Source: `events/data/events.csv` and `events/data/campaign-aliases.csv`; joined to "
               "~/Documents/GTM Project/outputs/sdr_funnel/deals.jsonl and leads.jsonl; workbook "
               "snapshot in outputs/growth_os/")


def render_returns(s: dict) -> str:
    t = s["totals"]
    L = ["# What each event returned", "",
         f"Generated {s['generated_at']} UTC. {t['measured']} of {t['events']} events have a matched "
         f"campaign, together {t['deals']} deals and {t['won']} wins worth ${t['revenue']:,.0f}.",
         "", "## What this can and cannot see", "",
         "Every figure comes from the Australian sales pipeline, which is the only one cached here, so "
         "UK and international events appear in the catalogue and cannot be measured at all. A campaign "
         "is an assignment made when the deal was created, not proof the event caused it. Costs are the "
         "planned 2026 figures from the workbook while returns span 2025 and 2026, and amounts are "
         "recorded deal values rather than recognised revenue.",
         "", SRC_RETURNS, "", "## The returns", "",
         "| Event | Cost | Deals | Won | Revenue | Cost per win | Return | Note |", "|---|---|---|---|---|---|---|---|"]
    for r in s["per_event"]:
        if not r["deals"]:
            continue
        note = "thin" if r["thin"] else ""
        L.append(f"| {r['name']} | {_money(r['cost_aud'])} | {r['deals']} | {r['won']} | "
                 f"{_money(r['revenue'])} | {_money(r['cost_per_win'])} | "
                 f"{'' if r['return_multiple'] is None else str(r['return_multiple']) + 'x'} | {note} |")
    L += ["", SRC_RETURNS, "", "## Where the sheet and HubSpot disagree", ""]
    rows = [r for r in s["per_event"] if r["claimed_vs_won"] not in (None, 0)]
    if rows:
        L += ["| Event | Sheet claims | HubSpot wins | Difference |", "|---|---|---|---|"]
        for r in rows:
            L.append(f"| {r['name']} | {r['claimed_deals']} | {r['won']} | {r['claimed_vs_won']:+d} |")
    else:
        L.append("Nothing disagrees.")
    L += ["", SRC_RETURNS, "", "## Exceptions to resolve", ""]
    if s["exceptions"]:
        L += ["| Event | Issue | Detail |", "|---|---|---|"]
        for x in s["exceptions"]:
            L.append(f"| {x['event']} | {x['issue']} | {x['detail']} |")
    else:
        L.append("None.")
    L += ["", "Each of these is a naming problem rather than a measurement problem, and each is fixed "
              "by a row in `events/data/campaign-aliases.csv` or by a consistent campaign name at the "
              "point the campaign is created.",
          "", SRC_RETURNS, ""]
    return "\n".join(L).replace("\u2014", "-")
```

Then extend `main` to load the events CSV back, load the aliases, read `deals.jsonl` and `leads.jsonl`, call `measure`, write `outputs/growth_os/events_returns.json`, and render `events/returns.md`.

- [ ] **Step 5: Extend the citations check**

In `checks.py`'s `citations_targets()`, add every `.md` directly under `events/` except `README.md`, following the pattern already used for `outbound-engine/`.

- [ ] **Step 6: Run the tests, then the real thing**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests -q 2>&1 | tail -2
python3 -m scripts.growth_os.events && python3 -m scripts.growth_os.checks
```

Expected: the suite green, then a returns table reproducing the prototype figures, NSWSPC 75 deals 30 won $182,468, WASSEA 20 and 13 and $119,640, ASA 18 and 7 and $66,745, PESA 14 and 0, QSPA 7 and 5, NSWSDPA 6 and 2. If a figure differs, stop and report it rather than adjusting the expectation. Checks must print OK.

- [ ] **Step 7: Commit**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: join events to campaigns through an editable alias file and render returns

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
cd ~/Documents/growth-os && git add -A && git commit -m "events: returns, the reconciliation and the exceptions to resolve

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

---

### Task 3: Catalogue and efficiency

**Files:**
- Modify: `scripts/growth_os/events.py`, `tests/test_events.py`
- Create (generated): `events/catalogue.md`, `events/efficiency.md`

**Interfaces:**
- Produces: `render_catalogue(summary, research) -> str`, `render_efficiency(summary) -> str`, `load_research(path) -> Dict[str, dict]`

- [ ] **Step 1: Write the failing tests**

```python
def test_load_research_returns_empty_when_absent(tmp_path):
    assert ev.load_research(tmp_path / "nope.csv") == {}


def test_efficiency_normalises_by_delegates_and_skips_thin():
    events = [
        {"event_key": "big", "name": "Big", "market": "AU", "cost_aud": 10000.0,
         "expected_delegates": 500, "claimed_deals": None, "delegate_list": "yes",
         "speaking_slot": "yes", "state": "NSW", "audience": "Secondary"},
        {"event_key": "tiny", "name": "Tiny", "market": "AU", "cost_aud": 1000.0,
         "expected_delegates": 50, "claimed_deals": None, "delegate_list": "no",
         "speaking_slot": "no", "state": "SA", "audience": "K12"},
    ]
    aliases = {"big": ["B"], "tiny": ["T"]}
    deals = [deal("B", True, "5000") for _ in range(4)] + [deal("T", True, "2000")]
    for i, d in enumerate(deals):
        d["id"] = f"d{i}"
    s = ev.measure(events, aliases, deals, [])
    md = ev.render_efficiency(s)
    assert "Big" in md
    assert "Tiny" not in md.split("## Events too small to judge")[0]
    assert "\u2014" not in md


def test_efficiency_compares_delegate_list_and_speaking_groups():
    events = [{"event_key": f"e{i}", "name": f"E{i}", "market": "AU", "cost_aud": 1000.0,
               "expected_delegates": 100, "claimed_deals": None,
               "delegate_list": "yes" if i < 2 else "no",
               "speaking_slot": "yes" if i % 2 == 0 else "no",
               "state": "NSW", "audience": "Secondary"} for i in range(4)]
    aliases = {f"e{i}": [f"C{i}"] for i in range(4)}
    deals = []
    for i in range(4):
        for j in range(3):
            d = deal(f"C{i}", i < 2, "1000")
            d["id"] = f"d{i}{j}"
            deals.append(d)
    md = ev.render_efficiency(ev.measure(events, aliases, deals, []))
    assert "delegate list" in md.lower()
    assert "hypothes" in md.lower()


def test_catalogue_marks_non_au_as_unmeasurable():
    events = [{"event_key": "uk1", "name": "UK One", "market": "UK", "cost_aud": 900.0,
               "expected_delegates": 100, "claimed_deals": None, "delegate_list": "yes",
               "speaking_slot": "no", "state": "", "audience": "K12"}]
    md = ev.render_catalogue(ev.measure(events, {}, [], []), {})
    assert "UK One" in md
    assert "cannot be measured" in md.lower()
```

- [ ] **Step 2: Run them, see them fail, then implement**

`load_research(path)` reads `research.csv` keyed by `event_key` into a dict of dicts, returning `{}` when the file is absent.

`render_efficiency(summary)` renders, for events clearing the thin rule, a table of cost per delegate, wins per hundred delegates, qualified meetings per hundred delegates and cost per qualified meeting; a section listing events too small to judge; and a final section comparing the group whose `delegate_list` is `yes` against `no`, and the same for `speaking_slot`, stating in plain words that with this few events these are hypotheses with numbers attached rather than findings. Every `## ` section ends with a `Source:` line.

`render_catalogue(summary, research)` renders one block per event grouped by market, carrying the workbook fields and any enriched fields present in `research`, with UK and international groups prefixed by a sentence saying they are listed here and cannot be measured from the cached Australian pipeline. Every `## ` section ends with a `Source:` line.

Extend `main` to write both files. Run the suite, then the real run, then `checks`.

- [ ] **Step 3: Commit**

```bash
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: event catalogue and efficiency, normalised by size and spend

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
cd ~/Documents/growth-os && git add -A && git commit -m "events: catalogue and efficiency

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

---

### Task 4: Research enrichment

**Files:**
- Create: `scripts/growth_os/event_research.py`
- Create: `events/data/research.csv`

**Interfaces:**
- Produces: `research.csv` with `event_key` plus, for each enriched field, the value and a `<field>_source` column.

Fields: `event_type`, `attendee_roles`, `confirmed_attendance`, `delegate_list_policy`, `speaking_available`, `organiser`.

- [ ] **Step 1: Write the script**

A resumable script that, for each event in `events.csv` that is Australian or has a status of booked, confirmed or WIP, and that has a URL:

1. Scrapes the event URL with `POST https://api.firecrawl.dev/v2/scrape`, body `{"url": url, "formats": ["markdown"]}`, `Authorization: Bearer $FIRECRAWL_API_KEY`.
2. Runs `POST /v2/search` with the query `"<event name> <year> sponsorship prospectus exhibitor delegate list"`, limit 4, and keeps only results whose host matches the event URL's host, since search returns many irrelevant overseas conferences. Scrapes any that survive.
3. Passes the combined markdown to headless Claude with a fixed prompt asking only for the six fields, instructing it to answer `not stated` for anything absent and to quote nothing.
4. Writes one row per event, with each field's source URL, appending as it goes so a rerun resumes rather than restarting.

The key is read from the environment after `source ~/.firecrawl_env`. It is never printed and never written to any file. Rate limit to one event at a time with a short pause. Any HTTP error is recorded as a blank field with the error in a `notes` column, never a guess.

- [ ] **Step 2: Run it on a sample of five first**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && source ~/.firecrawl_env && python3 -m scripts.growth_os.event_research --limit 5
```

Read the five rows. If `delegate_list_policy` is blank for all five, say so in your report rather than expanding the search, because the honest finding is that the field is not published.

- [ ] **Step 3: Run the rest, then report coverage**

Run without `--limit`. Then report, per field, how many of the attempted events got a value. That coverage table goes into the catalogue's source section so a reader knows how much of the profile is real.

- [ ] **Step 4: Commit**

Commit the script to the home repo and `research.csv` to the Growth OS repo. Never commit the key or the environment file.

---

### Task 5: The rubric and what to record

**Files:**
- Create: `events/scoring.md`, `events/what-to-record.md`, `events/README.md`

Hand-written. No code, no tests.

- [ ] **Step 1: Read everything generated first**

`events/returns.md`, `efficiency.md`, `catalogue.md`, the coverage table from Task 4, and `markets/au.md` for state penetration.

- [ ] **Step 2: Write `scoring.md`**

Six dimensions, each with its evidence or the explicit absence of it: audience fit, access through the delegate list, voice through a speaking slot, efficiency as cost per delegate and cost per qualified meeting, market need from state penetration, and measured return where it exists. Produce a band rather than a score to two decimals, and say why: nine to twelve measurable events cannot support more precision. Include a worked example on two real events, one that scores well and one that does not. End with what would change the rubric.

- [ ] **Step 3: Write `what-to-record.md`**

The fields to capture from the next event onward: who attended, whether the delegate list arrived and when, how many conversations were had, the campaign name used, and one consistent campaign naming convention. For each, one line on why it matters and what it would have told us this year if we had it.

- [ ] **Step 4: Write `README.md`**

Five lines: what the folder is, what to read first, what it can see, what it cannot, and that the two CSVs are meant to be edited by hand.

- [ ] **Step 5: Check, commit, link**

Run `checks`, which must print OK. Add one line to the repo `README.md` under the recommendation pointing at `events/`. Commit and push.

---

## Self-review

**Spec coverage.** Spec 5.1 is Task 1. 5.2 and 5.3 are Task 2, including the reconciliation and the exceptions list. 5.4 and 5.5 are Task 3. 5.6 is Task 4 including the coverage report the spec asks for. 5.7, 5.8 and 5.9 are Task 5. Spec 6's snapshot requirement is `snapshot()` in Task 1, and the offline determinism requirement is met because `events.py` never calls the network and `event_research.py` is the only script that does. Spec 7's verification points map to the test steps and the real-run checks in Tasks 1, 2 and 3.

**Placeholders.** None. Task 3's implementation is described rather than given verbatim because its three renderers follow the exact pattern of `render_returns`, which is given in full in Task 2; the tests that constrain them are given verbatim.

**Type consistency.** `normalise` returns rows whose keys are exactly `EVENT_COLUMNS`, which `write_csv` writes and `measure` consumes. `measure` adds `campaigns`, `deals`, `won`, `revenue`, `leads`, `qualified`, `cost_per_win`, `cost_per_qualified`, `return_multiple`, `thin`, `claimed_vs_won`, and all three renderers read only those plus the event columns. `load_aliases` returns the shape `measure` expects. `load_research` is keyed by `event_key`, which is the same key `normalise` produces.

**One thing the implementer must watch.** `parse_int` deliberately returns `None` for `Curated group; number TBC` by requiring the string to start with a digit. That rule also rejects a hypothetical `Expected 400 delegates`. If such a value appears in the real sheet, widen the rule and add a test rather than letting it through.
