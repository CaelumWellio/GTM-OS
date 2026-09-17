# Term 4 Outbound Case Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produce the evidence, the argument and the front door for a 90-day Term 4 outbound plan: a response-speed rule, an AU sequence inventory, the plan itself with a derived meetings number, and the two founder-facing pages rewritten to lead with the recommendation.

**Architecture:** Two new analysis modules join the existing four-stage customer-truth pipeline in `~/Documents/GTM Project/scripts/growth_os/`, reading the same local caches and reusing `common.py` for paths, parsing and outcome rules. Each computes a summary dict, renders a sourced markdown file into the Growth OS repo, and is covered by unit tests on hand-built fixtures. The two prose deliverables, the plan and the rewritten pages, are hand-written from those outputs, because judgement is the product.

**Tech Stack:** Python 3.9 standard library only (no `requests`, use `urllib` like the existing scripts), pytest 8 via `python3 -m pytest`, git.

## Global Constraints

- **HubSpot is read-only. No writes, no deletes, no changes of any kind, ever.** Allowed HTTP is GET, and POST only to paths ending `/search` or `/batch/read`. Production portal 20058914 only. Tasks 1 and 2 make no network calls at all except the one optional GET in Task 2; everything else reads local cache files.
- No sends, no sequence enrolments, no property creation, no list changes.
- Python 3.9 syntax: module docstring first, then `from __future__ import annotations`; `Optional[...]`; no `match`; no `X | Y` at runtime.
- Paths come from `common.py` constants, never hard-coded elsewhere.
- Founder-facing files carry no school name, no staff name, no email address. Founder-facing means every `.md` under `customer-truth/` except `method.md`, everything under `results/`, `outbound-engine/term-4-plan.md`, `outbound-engine/sequence-inventory.md`, and `README.md`.
- Every `## ` section of every generated or hand-written markdown file ends with a `Source:` line. `scripts/growth_os/checks.py citations` enforces this.
- Aggregate by sequence, persona and market. Never by individual rep. `hubspot_owner_id` is never written into any Growth OS file.
- Prose: no em-dashes, sentence case headings, every figure carries a source and an n.
- Thin-cell rule: fewer than 20 units in a cell means the cell is marked thin and no conclusion is drawn from it.
- Git: code commits go to the home-directory repo, staging only `Documents/GTM Project/scripts/growth_os`; content commits go to `~/Documents/growth-os` and are pushed. Every commit message ends with `Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>`.
- Never delete or clean anything under `outputs/`. Test fixtures use `tmp_path`.

## Domain facts the code depends on (verified 18 September 2026)

- `outputs/sdr_funnel/emails.jsonl`: 170,422 rows. Fields include `id`, `hs_timestamp`, `hs_email_direction` (`EMAIL` outbound 121,473, `INCOMING_EMAIL` inbound 48,942), `hs_sequence_id`, `hs_template_id`, `hs_email_thread_id`, `company_ids`. 62,167 rows carry a sequence id across 401 distinct sequences, 61,367 of those with a company association, spanning 2025-06-30 to 2026-09-03.
- `outputs/sdr_funnel/leads.jsonl`: 3,548 rows; 1,973 have `date_first_entered_meeting_booked` (date only, `YYYY-MM-DD`); 3,173 have `company_ids`.
- `outputs/sdr_funnel/deals.jsonl`: 1,018 rows, all in Sales Pipeline AU (`41942802`).
- `outputs/sdr_funnel/meetings.jsonl`: 7,451 rows, 6,840 with `company_ids`.
- `outputs/sdr_history/<company_id>.json`: `timeline` is a list of events with `type` (email, call, task, meeting, note), `id`, `ts` (ISO, naive UTC), `direction` (`EMAIL`, `INCOMING_EMAIL`, `OUTBOUND`, `INBOUND`), `subject`, `body`, `auto_reply`.
- `outputs/customer_truth/deals_early_text.jsonl`: one record per closed-or-open AU new-business deal with `deal_id`, `outcome`, `amount`, `cycle_days`, `createdate`, `closedate`, `window_start`, `window_end`, `company_ids`, `company_names`, `contact_names`, `source`, `campaign`, `dealtype`, `segment`, `state`, `chars`, `segments`.
- `outputs/customer_truth/deals_tagged.jsonl`: one row per deal with `error` (`None`, `no_text`, or a message).
- Measured baseline for Task 1, on 842 in-scope closed deals, base win rate 43.3%: reply within 0 to 3 days 46.9% (n=559), 4 to 14 days 31.3% (n=67), 15 or more days 16.3% (n=49), no reply in the early window 44.3% (n=167). Reply ratio above 0.7 wins 61.0% (n=59). Wellio sending 11 or more early emails wins 37.7% (n=276) against 48.4% at three to five (n=223).
- Existing test suite: 131 tests, `python3 -m pytest scripts/growth_os/tests -q` from `/Users/cleonard1998/Documents/GTM Project`.

## File structure

| Path | Responsibility |
|---|---|
| `scripts/growth_os/response_speed.py` | Compute the speed table, the anomaly split, and render `customer-truth/response-speed.md` |
| `scripts/growth_os/tests/test_response_speed.py` | Unit tests for the above on `tmp_path` fixtures |
| `scripts/growth_os/sequence_inventory.py` | Compute per-sequence metrics plus the unsequenced control, write the CSV, render `outbound-engine/sequence-inventory.md` |
| `scripts/growth_os/tests/test_sequence_inventory.py` | Unit tests for the above |
| `growth-os/customer-truth/response-speed.md` | Generated. The speed finding and its rule |
| `growth-os/outbound-engine/sequence-inventory.md` | Generated. Top 20 sequences, tail aggregate, control group, outliers |
| `growth-os/outbound-engine/term-4-plan.md` | Hand-written. The document sent ahead |
| `growth-os/README.md`, `growth-os/results/README.md` | Hand-written rewrites |
| `GTM Project/outputs/growth_os/response_speed.json`, `sequence_inventory.csv`, `sequence_inventory.json` | Machine outputs, never committed |

---

### Task 1: Response-speed analysis

**Files:**
- Create: `/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/response_speed.py`
- Create: `/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/tests/test_response_speed.py`
- Create (generated): `/Users/cleonard1998/Documents/growth-os/customer-truth/response-speed.md`
- Modify: `/Users/cleonard1998/Documents/growth-os/customer-truth/method.md` (append a Rules bullet and a run-log line)

**Interfaces:**
- Consumes: `common.py` (`OUT`, `HISTORY`, `GROWTH_OS`, `read_jsonl`, `write_jsonl`, `parse_ts`, `ids`, `outcome`)
- Produces, used by Task 3 (the plan) and Task 4 (the rewrite):
  - `deal_signals(rec, load_timeline) -> dict` with keys `deal_id`, `won`, `n_in`, `n_out`, `n_mtg`, `days_to_first_reply` (int or None), `reply_ratio` (float or None), `source`, `campaign`, `dealtype`, `segment`
  - `bucket_days(d: Optional[int]) -> str` returning one of `"0-3"`, `"4-14"`, `"15+"`, `"none"`
  - `summarise(records, load_timeline) -> dict` with `generated_at`, `population`, `base_rate`, `by_reply_speed`, `by_reply_ratio`, `by_outbound_volume`, `anomaly`
  - `render(summary) -> str`
  - `main(argv)` writing `outputs/growth_os/response_speed.json` and the markdown

- [ ] **Step 1: Write the failing tests**

Create `scripts/growth_os/tests/test_response_speed.py`:

```python
from __future__ import annotations
from scripts.growth_os import response_speed as rs


def ev(type_, direction, ts, auto="False", id_="e"):
    return {"type": type_, "id": id_, "ts": ts, "direction": direction, "subject": "s",
            "body": "b", "outcome": "None", "activity_type": "None", "disposition": "None",
            "auto_reply": auto}


def rec(deal_id="d1", outcome="won", start="2025-03-01T00:00:00", end="2025-05-01T00:00:00",
        created="2025-03-01T00:00:00Z", **kw):
    base = {"deal_id": deal_id, "outcome": outcome, "window_start": start, "window_end": end,
            "createdate": created, "company_ids": ["c1"], "source": "OFFLINE",
            "campaign": "camp", "dealtype": "newbusiness", "segment": "Catholic"}
    base.update(kw)
    return base


def test_bucket_days_boundaries():
    assert rs.bucket_days(0) == "0-3"
    assert rs.bucket_days(3) == "0-3"
    assert rs.bucket_days(4) == "4-14"
    assert rs.bucket_days(14) == "4-14"
    assert rs.bucket_days(15) == "15+"
    assert rs.bucket_days(None) == "none"


def test_deal_signals_counts_and_first_reply():
    tl = [ev("email", "EMAIL", "2025-03-02T00:00:00", id_="o1"),
          ev("email", "INCOMING_EMAIL", "2025-03-06T00:00:00", id_="i1"),
          ev("email", "INCOMING_EMAIL", "2025-03-09T00:00:00", id_="i2"),
          ev("meeting", "None", "2025-03-10T00:00:00", id_="m1")]
    out = rs.deal_signals(rec(), lambda cid: tl)
    assert out["n_out"] == 1 and out["n_in"] == 2 and out["n_mtg"] == 1
    assert out["days_to_first_reply"] == 5
    assert out["reply_ratio"] == 2.0
    assert out["won"] is True


def test_deal_signals_ignores_auto_replies_and_out_of_window():
    tl = [ev("email", "INCOMING_EMAIL", "2025-03-05T00:00:00", auto="True", id_="auto"),
          ev("email", "INCOMING_EMAIL", "2025-01-01T00:00:00", id_="early"),
          ev("email", "INCOMING_EMAIL", "2025-06-01T00:00:00", id_="late")]
    out = rs.deal_signals(rec(), lambda cid: tl)
    assert out["n_in"] == 0 and out["days_to_first_reply"] is None and out["reply_ratio"] is None


def test_summarise_buckets_and_base_rate():
    fast_won = [rec(f"w{i}", "won") for i in range(3)]
    slow_lost = [rec(f"l{i}", "lost") for i in range(2)]
    tl_fast = [ev("email", "INCOMING_EMAIL", "2025-03-02T00:00:00")]
    tl_slow = [ev("email", "INCOMING_EMAIL", "2025-03-25T00:00:00")]

    def loader(deal_id):
        return lambda cid: tl_fast if deal_id.startswith("w") else tl_slow

    s = rs.summarise(fast_won + slow_lost, lambda cid, _cache={}: tl_fast)
    assert s["population"]["deals"] == 5
    assert s["base_rate"] == 0.6
    assert s["by_reply_speed"]["0-3"]["n"] == 5


def test_summarise_anomaly_splits_no_reply_group():
    recs = [rec("a", "won", source="OFFLINE"), rec("b", "won", source="DIRECT_TRAFFIC")]
    s = rs.summarise(recs, lambda cid: [])
    an = s["anomaly"]
    assert an["n"] == 2
    assert an["by_source"]["OFFLINE"]["n"] == 1
    assert an["by_source"]["DIRECT_TRAFFIC"]["n"] == 1


def test_render_has_source_line_in_every_section():
    import re
    s = rs.summarise([rec("a", "won"), rec("b", "lost")], lambda cid: [])
    md = rs.render(s)
    parts = re.split(r"^## ", md, flags=re.M)
    assert len(parts) > 1
    for p in parts[1:]:
        assert "Source:" in p
    assert "\u2014" not in md
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_response_speed.py -q 2>&1 | tail -3
```

Expected: `ModuleNotFoundError: No module named 'scripts.growth_os.response_speed'`.

- [ ] **Step 3: Write `response_speed.py`**

```python
"""Does early responsiveness predict the outcome? Reads cached timelines only, no network.

Cycle length is largely an effect: lost deals linger before anyone marks them lost. This
measures signals observable inside the early window instead, so a rule drawn from it can be
applied while a deal is still live.
"""
from __future__ import annotations
import argparse
import datetime as dt
import json
import pathlib
import re
import sys
from typing import Callable, Dict, List, Optional

from . import common as c

BUCKETS = ["0-3", "4-14", "15+", "none"]
RATIO_BUCKETS = ["0", "under 0.3", "0.3-0.7", "over 0.7"]
VOLUME_BUCKETS = ["0-2", "3-5", "6-10", "11+"]
MIN_CELL = 20


def bucket_days(d: Optional[int]) -> str:
    if d is None:
        return "none"
    if d <= 3:
        return "0-3"
    if d <= 14:
        return "4-14"
    return "15+"


def bucket_ratio(r: Optional[float]) -> Optional[str]:
    if r is None:
        return None
    if r == 0:
        return "0"
    if r < 0.3:
        return "under 0.3"
    if r <= 0.7:
        return "0.3-0.7"
    return "over 0.7"


def bucket_volume(n: int) -> str:
    if n <= 2:
        return "0-2"
    if n <= 5:
        return "3-5"
    if n <= 10:
        return "6-10"
    return "11+"


def default_load_timeline(company_id: str) -> List[dict]:
    p = c.HISTORY / f"{company_id}.json"
    if not p.exists():
        return []
    with p.open() as f:
        return json.load(f).get("timeline", [])


def deal_signals(rec: dict, load_timeline: Callable[[str], List[dict]]) -> dict:
    start, end = c.parse_ts(rec.get("window_start")), c.parse_ts(rec.get("window_end"))
    created = c.parse_ts(rec.get("createdate"))
    events = []
    for cid in c.ids(rec.get("company_ids")):
        for e in load_timeline(cid):
            t = c.parse_ts(e.get("ts"))
            if t and start and end and start <= t <= end:
                events.append((t, e))
    inbound = [(t, e) for t, e in events
               if e.get("type") == "email" and e.get("direction") == "INCOMING_EMAIL"
               and str(e.get("auto_reply")) != "True"]
    outbound = [(t, e) for t, e in events
                if e.get("type") == "email" and e.get("direction") == "EMAIL"]
    meetings = [(t, e) for t, e in events if e.get("type") == "meeting"]
    first = min((t for t, _ in inbound), default=None)
    return {
        "deal_id": rec.get("deal_id"),
        "won": rec.get("outcome") == "won",
        "n_in": len(inbound),
        "n_out": len(outbound),
        "n_mtg": len(meetings),
        "days_to_first_reply": (first - created).days if first and created else None,
        "reply_ratio": (len(inbound) / len(outbound)) if outbound else None,
        "source": rec.get("source") or "",
        "campaign": rec.get("campaign") or "",
        "dealtype": rec.get("dealtype") or "",
        "segment": rec.get("segment") or "",
    }


def _cell(rows: List[dict]) -> dict:
    n = len(rows)
    won = sum(1 for r in rows if r["won"])
    return {"n": n, "won": won, "rate": round(won / n, 4) if n else None, "thin": n < MIN_CELL}


def summarise(records: List[dict], load_timeline: Callable[[str], List[dict]]) -> dict:
    sig = [deal_signals(r, load_timeline) for r in records]
    by_speed = {b: _cell([s for s in sig if bucket_days(s["days_to_first_reply"]) == b]) for b in BUCKETS}
    by_ratio = {b: _cell([s for s in sig if bucket_ratio(s["reply_ratio"]) == b]) for b in RATIO_BUCKETS}
    by_volume = {b: _cell([s for s in sig if bucket_volume(s["n_out"]) == b]) for b in VOLUME_BUCKETS}
    noreply = [s for s in sig if s["days_to_first_reply"] is None]
    replied = [s for s in sig if s["days_to_first_reply"] is not None]

    def split(rows: List[dict], field: str) -> dict:
        groups: Dict[str, List[dict]] = {}
        for r in rows:
            groups.setdefault(r.get(field) or "Unknown", []).append(r)
        return {k: _cell(v) for k, v in sorted(groups.items(), key=lambda kv: -len(kv[1]))}

    won_total = sum(1 for s in sig if s["won"])
    return {
        "generated_at": dt.datetime.utcnow().isoformat(timespec="seconds"),
        "min_cell": MIN_CELL,
        "population": {"deals": len(sig), "won": won_total, "lost": len(sig) - won_total,
                       "replied": len(replied), "no_reply": len(noreply)},
        "base_rate": round(won_total / len(sig), 4) if sig else None,
        "by_reply_speed": by_speed,
        "by_reply_ratio": by_ratio,
        "by_outbound_volume": by_volume,
        "anomaly": {"n": len(noreply), "cell": _cell(noreply),
                    "by_source": split(noreply, "source"),
                    "by_dealtype": split(noreply, "dealtype"),
                    "by_campaign": split(noreply, "campaign"),
                    "replied_by_source": split(replied, "source")},
    }


def _pct(x: Optional[float]) -> str:
    return "n/a" if x is None else f"{x * 100:.1f}%"


SRC = ("Source: ~/Documents/GTM Project/outputs/growth_os/response_speed.json; "
       "computed from outputs/customer_truth/deals_early_text.jsonl and outputs/sdr_history/; "
       "method in customer-truth/method.md")


def render(summary: dict) -> str:
    pop, speed = summary["population"], summary["by_reply_speed"]
    L = ["# How fast a school replies", "",
         f"Generated {summary['generated_at']} UTC. {pop['deals']} closed deals, base win rate {_pct(summary['base_rate'])}.",
         "", "## The question", "",
         "Deals that won closed in a median 27 days against 73 for deals that lost. That gap is mostly an effect, "
         "because a deal that is going nowhere sits open until somebody closes it. The useful question is whether "
         "anything visible early, while the deal is still live, predicts the outcome.",
         "", SRC, "", "## How fast the school replies", "",
         "| Days from deal creation to the school's first reply | Win rate | Deals | Note |", "|---|---|---|---|"]
    labels = {"0-3": "0 to 3", "4-14": "4 to 14", "15+": "15 or more", "none": "no reply in the early window"}
    for b in BUCKETS:
        cell = speed[b]
        L.append(f"| {labels[b]} | {_pct(cell['rate'])} | {cell['n']} | {'thin' if cell['thin'] else ''} |")
    L += ["", SRC, "", "## How hard we pushed", "",
          "| Wellio emails sent in the early window | Win rate | Deals | Note |", "|---|---|---|---|"]
    for b in VOLUME_BUCKETS:
        cell = summary["by_outbound_volume"][b]
        L.append(f"| {b} | {_pct(cell['rate'])} | {cell['n']} | {'thin' if cell['thin'] else ''} |")
    L += ["", SRC, "", "## The group that does not fit", ""]
    an = summary["anomaly"]
    top = list(an["by_source"].items())[:4]
    L.append(f"{an['n']} deals had no school reply at all in the early window, and they win {_pct(an['cell']['rate'])}, "
             f"which is higher than the deals that replied slowly. That has to be explained before any rule is drawn, "
             f"because a rule that deprioritises silence would be wrong for this group.")
    L += ["", "| Source of the deal | Win rate | Deals |", "|---|---|---|"]
    for k, cell in top:
        L.append(f"| {k or 'unknown'} | {_pct(cell['rate'])} | {cell['n']} |")
    L += ["", SRC, "", "## Limits", "",
          f"- Cells below {summary['min_cell']} deals are marked thin and carry no conclusion.",
          "- Replies are counted from cached company timelines, so a reply by phone or in person is invisible here.",
          "- Auto-replies are excluded, and the early window ends when the deal first reached Evaluating.",
          "- This is an association measured on closed deals, not a controlled test.",
          "", SRC, ""]
    return "\n".join(L).replace("\u2014", "-")


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="Does early responsiveness predict the outcome?")
    ap.add_argument("--out-json", default=str(c.GOS_OUT / "response_speed.json"))
    ap.add_argument("--out-md", default=str(c.GROWTH_OS / "customer-truth" / "response-speed.md"))
    args = ap.parse_args(argv)
    early = c.read_jsonl(c.OUT / "deals_early_text.jsonl")
    tagged = {r["deal_id"]: r for r in c.read_jsonl(c.OUT / "deals_tagged.jsonl")}
    scope = [r for r in early if r.get("outcome") in ("won", "lost")
             and tagged.get(r["deal_id"], {}).get("error") != "no_text"]
    cache: Dict[str, List[dict]] = {}

    def loader(cid: str) -> List[dict]:
        if cid not in cache:
            cache[cid] = default_load_timeline(cid)
        return cache[cid]

    s = summarise(scope, loader)
    jp = pathlib.Path(args.out_json)
    jp.parent.mkdir(parents=True, exist_ok=True)
    jp.write_text(json.dumps(s, indent=1))
    mp = pathlib.Path(args.out_md)
    mp.parent.mkdir(parents=True, exist_ok=True)
    mp.write_text(render(s))
    print(f"population {s['population']} base rate {s['base_rate']}")
    for b in BUCKETS:
        print(f"  {b:>6}: {_pct(s['by_reply_speed'][b]['rate'])} n={s['by_reply_speed'][b]['n']}")
    print(f"WROTE {jp} and {mp}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_response_speed.py -q 2>&1 | tail -3
```

Expected: `6 passed`. Then the whole suite: `python3 -m pytest scripts/growth_os/tests -q` expecting `137 passed`.

- [ ] **Step 5: Run it on the real data and read the anomaly**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.response_speed
```

Expected population around 842 with base rate near 0.433 and the four speed buckets matching the domain facts above. Then open `outputs/growth_os/response_speed.json` and read `anomaly.by_source` and `anomaly.by_dealtype` against `anomaly.replied_by_source`. Write one or two sentences of resolution into the "The group that does not fit" section of the generated markdown by editing `render` to append the finding, not by hand-editing the generated file. If the no-reply group is concentrated in sources that differ from the replied group, say which and state the rule as conditional on deal source. If it is not concentrated, say the anomaly is unexplained and that no rule is drawn.

- [ ] **Step 6: Add the rule and the method note**

Append to `render` a final section `## The rule` stating, in plain words with no statistical vocabulary, what a rep should do differently and for which deals, followed by `SRC`. Then append one bullet to the `## Rules` section of `/Users/cleonard1998/Documents/growth-os/customer-truth/method.md` recording that response speed is measured from cached timelines inside the same early window as the hook pass, and one run-log line at the end of that file in the existing format.

- [ ] **Step 7: Check and commit**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.checks
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: response-speed analysis, does early responsiveness predict the outcome

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
cd ~/Documents/growth-os && git add -A && git commit -m "customer-truth: response speed, the rule and its limits

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

Expected: checks print `OK` before either commit.

---

### Task 2: AU sequence inventory

**Files:**
- Create: `/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/sequence_inventory.py`
- Create: `/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/tests/test_sequence_inventory.py`
- Create (generated): `/Users/cleonard1998/Documents/growth-os/outbound-engine/sequence-inventory.md`

**Interfaces:**
- Consumes: `common.py` (`FUNNEL`, `GOS_OUT`, `GROWTH_OS`, `read_jsonl`, `parse_ts`, `ids`, `SALES_PIPELINE_AU`, `is_new_business`, `outcome`)
- Produces, used by Task 3:
  - `build_index(emails, leads, deals) -> dict` with `sends_by_seq`, `first_send`, `replies_by_company`, `meeting_dates_by_company`, `deals_by_company`
  - `metrics_for(company_first_send, index) -> dict` with `schools`, `replied`, `reply_rate`, `meetings`, `meeting_rate`, `deals_created`, `deals_won`, `amount_won`
  - `summarise(emails, leads, deals) -> dict` with `generated_at`, `coverage`, `sequences` (list of dicts each with `sequence_id`, `name`, `sends`, plus the metrics), `tail`, `control`
  - `render(summary) -> str`
  - `main(argv)`

- [ ] **Step 1: Write the failing tests**

Create `scripts/growth_os/tests/test_sequence_inventory.py`:

```python
from __future__ import annotations
from scripts.growth_os import sequence_inventory as si


def email(id_, ts, direction="EMAIL", seq=None, company="c1", auto=None):
    row = {"id": id_, "hs_timestamp": ts, "hs_email_direction": direction,
           "company_ids": [company] if company else []}
    if seq:
        row["hs_sequence_id"] = seq
    return row


def lead(company, booked):
    return {"id": "L" + company, "company_ids": [company],
            "date_first_entered_meeting_booked": booked}


def deal(id_, company, created, won, amount="1000"):
    return {"id": id_, "company_ids": [company], "createdate": created,
            "pipeline": "41942802", "dealtype": "newbusiness", "amount": amount,
            "hs_is_closed": "true", "hs_is_closed_won": "true" if won else "false"}


def test_reply_must_follow_a_send_within_14_days():
    emails = [email("s1", "2026-01-01T00:00:00Z", seq="S1"),
              email("r1", "2026-01-05T00:00:00Z", direction="INCOMING_EMAIL"),
              email("r2", "2026-03-01T00:00:00Z", direction="INCOMING_EMAIL", company="c2"),
              email("s2", "2026-01-01T00:00:00Z", seq="S1", company="c2")]
    s = si.summarise(emails, [], [])
    seq = s["sequences"][0]
    assert seq["sequence_id"] == "S1"
    assert seq["schools"] == 2
    assert seq["replied"] == 1
    assert seq["reply_rate"] == 0.5


def test_meeting_counted_within_30_days_of_first_send():
    emails = [email("s1", "2026-01-01T00:00:00Z", seq="S1"),
              email("s2", "2026-01-01T00:00:00Z", seq="S1", company="c2")]
    leads = [lead("c1", "2026-01-20"), lead("c2", "2026-04-01")]
    s = si.summarise(emails, leads, [])
    assert s["sequences"][0]["meetings"] == 1


def test_deal_counted_within_60_days_and_won_tracked():
    emails = [email("s1", "2026-01-01T00:00:00Z", seq="S1")]
    deals = [deal("d1", "c1", "2026-02-01T00:00:00Z", True, "2500")]
    s = si.summarise(emails, [], deals)
    seq = s["sequences"][0]
    assert seq["deals_created"] == 1 and seq["deals_won"] == 1 and seq["amount_won"] == 2500.0


def test_control_group_is_companies_with_no_sequenced_send():
    emails = [email("s1", "2026-01-01T00:00:00Z", seq="S1"),
              email("m1", "2026-01-01T00:00:00Z", company="c9"),
              email("r9", "2026-01-03T00:00:00Z", direction="INCOMING_EMAIL", company="c9")]
    s = si.summarise(emails, [], [])
    assert s["control"]["schools"] == 1
    assert s["control"]["replied"] == 1


def test_coverage_reports_sequenced_share():
    emails = [email("s1", "2026-01-01T00:00:00Z", seq="S1"),
              email("m1", "2026-01-01T00:00:00Z")]
    s = si.summarise(emails, [], [])
    assert s["coverage"]["sequenced_sends"] == 1
    assert s["coverage"]["total_sends"] == 2


def test_thin_sequences_are_flagged():
    emails = [email(f"s{i}", "2026-01-01T00:00:00Z", seq="S1", company=f"c{i}") for i in range(5)]
    s = si.summarise(emails, [], [])
    assert s["sequences"][0]["thin"] is True


def test_render_has_source_line_in_every_section_and_no_owner_ids():
    import re
    emails = [email(f"s{i}", "2026-01-01T00:00:00Z", seq="S1", company=f"c{i}") for i in range(3)]
    md = si.render(si.summarise(emails, [], []))
    for p in re.split(r"^## ", md, flags=re.M)[1:]:
        assert "Source:" in p
    assert "hubspot_owner_id" not in md
    assert "\u2014" not in md
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_sequence_inventory.py -q 2>&1 | tail -3
```

Expected: `ModuleNotFoundError` on `sequence_inventory`.

- [ ] **Step 3: Write `sequence_inventory.py`**

```python
"""What the AU outbound sequences actually did. Reads cached HubSpot objects only.

Attribution is touch-based, not causal: a school touched by several sequences is credited to
each, so column totals exceed reality. The unsequenced control group is the baseline to read
sequence performance against.
"""
from __future__ import annotations
import argparse
import csv
import datetime as dt
import json
import pathlib
import re
import sys
from typing import Dict, List, Optional

from . import common as c

REPLY_WINDOW_DAYS = 14
MEETING_WINDOW_DAYS = 30
DEAL_WINDOW_DAYS = 60
MIN_SCHOOLS = 20
TOP_N = 20


def _seq(row: dict) -> Optional[str]:
    v = row.get("hs_sequence_id")
    return None if v in (None, "", "None") else str(v)


def build_index(emails: List[dict], leads: List[dict], deals: List[dict]) -> dict:
    sends_by_seq: Dict[str, List[dict]] = {}
    first_send: Dict[str, Dict[str, dt.datetime]] = {}
    replies: Dict[str, List[dt.datetime]] = {}
    all_send_companies = set()
    for e in emails:
        t = c.parse_ts(e.get("hs_timestamp"))
        if not t:
            continue
        comps = c.ids(e.get("company_ids"))
        direction = e.get("hs_email_direction")
        if direction == "INCOMING_EMAIL":
            for cid in comps:
                replies.setdefault(cid, []).append(t)
            continue
        if direction != "EMAIL":
            continue
        for cid in comps:
            all_send_companies.add(cid)
        s = _seq(e)
        if not s:
            continue
        sends_by_seq.setdefault(s, []).append(e)
        for cid in comps:
            prev = first_send.setdefault(s, {}).get(cid)
            if prev is None or t < prev:
                first_send[s][cid] = t
    meetings: Dict[str, List[dt.datetime]] = {}
    for l in leads:
        d = c.parse_ts(l.get("date_first_entered_meeting_booked"))
        if not d:
            continue
        for cid in c.ids(l.get("company_ids")):
            meetings.setdefault(cid, []).append(d)
    by_company_deals: Dict[str, List[dict]] = {}
    for d in deals:
        if str(d.get("pipeline")) != c.SALES_PIPELINE_AU or not c.is_new_business(d):
            continue
        for cid in c.ids(d.get("company_ids")):
            by_company_deals.setdefault(cid, []).append(d)
    return {"sends_by_seq": sends_by_seq, "first_send": first_send, "replies_by_company": replies,
            "meeting_dates_by_company": meetings, "deals_by_company": by_company_deals,
            "all_send_companies": all_send_companies}


def metrics_for(company_first_send: Dict[str, dt.datetime], index: dict) -> dict:
    schools = len(company_first_send)
    replied = meetings = deals_created = deals_won = 0
    amount_won = 0.0
    for cid, t0 in company_first_send.items():
        if any(t0 <= r <= t0 + dt.timedelta(days=REPLY_WINDOW_DAYS)
               for r in index["replies_by_company"].get(cid, [])):
            replied += 1
        if any(t0 <= m <= t0 + dt.timedelta(days=MEETING_WINDOW_DAYS)
               for m in index["meeting_dates_by_company"].get(cid, [])):
            meetings += 1
        for d in index["deals_by_company"].get(cid, []):
            created = c.parse_ts(d.get("createdate"))
            if created and t0 <= created <= t0 + dt.timedelta(days=DEAL_WINDOW_DAYS):
                deals_created += 1
                if c.outcome(d) == "won":
                    deals_won += 1
                    try:
                        amount_won += float(d.get("amount") or 0)
                    except ValueError:
                        pass
    rate = lambda x: round(x / schools, 4) if schools else None
    return {"schools": schools, "replied": replied, "reply_rate": rate(replied),
            "meetings": meetings, "meeting_rate": rate(meetings),
            "deals_created": deals_created, "deals_won": deals_won,
            "amount_won": round(amount_won, 2)}


def summarise(emails: List[dict], leads: List[dict], deals: List[dict],
              names: Optional[Dict[str, str]] = None) -> dict:
    index = build_index(emails, leads, deals)
    names = names or {}
    rows = []
    for s, sends in index["sends_by_seq"].items():
        m = metrics_for(index["first_send"].get(s, {}), index)
        m.update({"sequence_id": s, "name": names.get(s, ""), "sends": len(sends),
                  "thin": m["schools"] < MIN_SCHOOLS})
        rows.append(m)
    rows.sort(key=lambda r: -r["sends"])
    sequenced_companies = {cid for m in index["first_send"].values() for cid in m}
    control_companies = {cid: None for cid in index["all_send_companies"] - sequenced_companies}
    ctrl_first: Dict[str, dt.datetime] = {}
    for e in emails:
        if e.get("hs_email_direction") != "EMAIL" or _seq(e):
            continue
        t = c.parse_ts(e.get("hs_timestamp"))
        if not t:
            continue
        for cid in c.ids(e.get("company_ids")):
            if cid in control_companies and (cid not in ctrl_first or t < ctrl_first[cid]):
                ctrl_first[cid] = t
    total_sends = sum(1 for e in emails if e.get("hs_email_direction") == "EMAIL")
    return {"generated_at": dt.datetime.utcnow().isoformat(timespec="seconds"),
            "windows": {"reply_days": REPLY_WINDOW_DAYS, "meeting_days": MEETING_WINDOW_DAYS,
                        "deal_days": DEAL_WINDOW_DAYS},
            "min_schools": MIN_SCHOOLS,
            "coverage": {"total_sends": total_sends,
                         "sequenced_sends": sum(len(v) for v in index["sends_by_seq"].values()),
                         "sequences": len(rows)},
            "sequences": rows,
            "tail": _aggregate(rows[TOP_N:]),
            "control": metrics_for(ctrl_first, index)}


def _aggregate(rows: List[dict]) -> dict:
    if not rows:
        return {"sequences": 0, "sends": 0, "schools": 0, "replied": 0, "reply_rate": None,
                "meetings": 0, "deals_won": 0}
    schools = sum(r["schools"] for r in rows)
    replied = sum(r["replied"] for r in rows)
    return {"sequences": len(rows), "sends": sum(r["sends"] for r in rows), "schools": schools,
            "replied": replied, "reply_rate": round(replied / schools, 4) if schools else None,
            "meetings": sum(r["meetings"] for r in rows), "deals_won": sum(r["deals_won"] for r in rows)}


def _pct(x: Optional[float]) -> str:
    return "n/a" if x is None else f"{x * 100:.1f}%"


SRC = ("Source: ~/Documents/GTM Project/outputs/growth_os/sequence_inventory.csv and "
       "sequence_inventory.json; computed from outputs/sdr_funnel/emails.jsonl, leads.jsonl and deals.jsonl")


def render(summary: dict) -> str:
    cov, w = summary["coverage"], summary["windows"]
    share = 100 * cov["sequenced_sends"] / cov["total_sends"] if cov["total_sends"] else 0
    L = ["# What the outbound sequences did", "",
         f"Generated {summary['generated_at']} UTC. {cov['sequences']} sequences, "
         f"{cov['sequenced_sends']} sequenced sends of {cov['total_sends']} outbound emails ({share:.1f}%).",
         "", "## What this covers, and what it does not", "",
         f"Only {share:.1f} percent of outbound emails carry a sequence id. The rest are written by hand, "
         "so this describes the sequenced part of outbound and nothing else. A school touched by several "
         "sequences is counted under each of them, so the columns add up to more than reality. "
         f"A reply counts when it arrives within {w['reply_days']} days of the first send to that school, "
         f"a meeting within {w['meeting_days']} days, and a deal within {w['deal_days']} days. "
         "This is touch attribution, not proof that the sequence caused anything.",
         "", SRC, "", "## The twenty largest sequences", "",
         "| Sequence | Sends | Schools | Reply rate | Meetings | Deals won | Note |", "|---|---|---|---|---|---|---|"]
    for r in summary["sequences"][:TOP_N]:
        label = r["name"] or r["sequence_id"]
        L.append(f"| {label} | {r['sends']} | {r['schools']} | {_pct(r['reply_rate'])} | "
                 f"{r['meetings']} | {r['deals_won']} | {'thin' if r['thin'] else ''} |")
    t = summary["tail"]
    L.append(f"| everything else, {t['sequences']} sequences | {t['sends']} | {t['schools']} | "
             f"{_pct(t['reply_rate'])} | {t['meetings']} | {t['deals_won']} | |")
    ctrl = summary["control"]
    L += ["", SRC, "", "## Against the schools we emailed without a sequence", "",
          "| Group | Schools | Reply rate | Meetings | Deals won |", "|---|---|---|---|---|",
          f"| no sequence, written by hand | {ctrl['schools']} | {_pct(ctrl['reply_rate'])} | "
          f"{ctrl['meetings']} | {ctrl['deals_won']} |", "", SRC, "", "## Limits", "",
          f"- A sequence touching fewer than {summary['min_schools']} schools is marked thin and carries no conclusion.",
          "- Performance is reported by sequence, never by the person who sent it.",
          "- Replies by phone or in person are invisible here.",
          "- The windows above are choices, not facts; a different window would move the numbers.",
          "", SRC, ""]
    return "\n".join(L).replace("\u2014", "-")


def fetch_names() -> Dict[str, str]:
    """Sequence names over REST, read-only GET. Returns {} on any failure."""
    try:
        sys.path.insert(0, str(c.GTM / "scripts" / "hubspot" / "funnel"))
        import pull_funnel as pf
        s, b = pf.call("GET", "/automation/v4/sequences?limit=100")
        if s >= 400:
            return {}
        return {str(r.get("id")): r.get("name") or "" for r in b.get("results", [])}
    except Exception:
        return {}


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="AU outbound sequence inventory, read-only.")
    ap.add_argument("--names", action="store_true", help="fetch sequence names over REST (GET only)")
    ap.add_argument("--out-md", default=str(c.GROWTH_OS / "outbound-engine" / "sequence-inventory.md"))
    args = ap.parse_args(argv)
    emails = c.read_jsonl(c.FUNNEL / "emails.jsonl")
    leads = c.read_jsonl(c.FUNNEL / "leads.jsonl")
    deals = c.read_jsonl(c.FUNNEL / "deals.jsonl")
    names = fetch_names() if args.names else {}
    s = summarise(emails, leads, deals, names)
    c.GOS_OUT.mkdir(parents=True, exist_ok=True)
    (c.GOS_OUT / "sequence_inventory.json").write_text(json.dumps(s, indent=1))
    cols = ["sequence_id", "name", "sends", "schools", "replied", "reply_rate", "meetings",
            "meeting_rate", "deals_created", "deals_won", "amount_won", "thin"]
    with (c.GOS_OUT / "sequence_inventory.csv").open("w", newline="") as f:
        wtr = csv.writer(f)
        wtr.writerow(cols)
        for r in s["sequences"]:
            wtr.writerow([r.get(k) for k in cols])
    mp = pathlib.Path(args.out_md)
    mp.parent.mkdir(parents=True, exist_ok=True)
    mp.write_text(render(s))
    print(f"sequences {s['coverage']['sequences']} | sequenced sends {s['coverage']['sequenced_sends']} "
          f"of {s['coverage']['total_sends']} | control schools {s['control']['schools']}")
    for r in s["sequences"][:5]:
        print(f"  {r['sequence_id']} sends {r['sends']} schools {r['schools']} reply {_pct(r['reply_rate'])}")
    print(f"WROTE {mp}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_sequence_inventory.py -q 2>&1 | tail -3
```

Expected: `7 passed`. Then the whole suite expecting `144 passed`.

- [ ] **Step 5: Confirm the read-only posture, then run it**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.checks readonly
source ~/.hubspot_prod_env && python3 -m scripts.growth_os.sequence_inventory --names
```

Expected: `OK` from the check, then a line reporting around 401 sequences and roughly 62,167 sequenced sends of about 121,473 outbound emails. If the names fetch returns nothing the table shows ids, which is acceptable; note it in the report.

- [ ] **Step 6: Check and commit**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.checks
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: AU sequence inventory from cached email, lead and deal objects

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
cd ~/Documents/growth-os && git add -A && git commit -m "outbound-engine: sequence inventory, top twenty against the unsequenced baseline

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

---

### Task 3: The 90-day Term 4 plan

**Files:**
- Create: `/Users/cleonard1998/Documents/growth-os/outbound-engine/term-4-plan.md`

**Interfaces:**
- Consumes: `customer-truth/response-speed.md` and `outputs/growth_os/response_speed.json` from Task 1; `outbound-engine/sequence-inventory.md` and `outputs/growth_os/sequence_inventory.json` from Task 2; `results/baseline-2026-09.md`; `customer-truth/what-the-market-is-telling-us.md`.
- Produces: the document Task 4's README points at.

This task is hand-written prose. There is no code and no test, so the gate is the structure below plus the checks.

- [ ] **Step 1: Read the evidence before writing a word**

Read all four files named above in full, plus `outputs/growth_os/sequence_inventory.json` for the figures the markdown summarises. Write nothing until the three moves in step 3 are chosen from what those files actually say. A move that cannot cite a figure from them does not go in.

- [ ] **Step 2: Derive the number, showing the arithmetic**

Build the chain from the inventory and the funnel baseline. The form is: schools reachable in the window, times the reply rate the inventory measured, times the meeting rate the inventory measured, gives meetings booked. State each input, its value, and the file it came from. Where the response-speed rule changes where effort goes, state the assumed effect and label it an assumption rather than a measurement. Record the arithmetic in the plan itself so a reader can check it in their head. If the chain gives an implausible number, say so and use a range, rather than presenting a single figure you do not believe.

- [ ] **Step 3: Write the document, in exactly this order**

1. `## The number`. the commitment in qualified meetings booked over the 90 days to mid-December, with the baseline it is measured against, in the first two lines.
2. `## Why this window`. Term 4, using the recorded figure that 204 of 284 wins in 2025 (70 percent) landed on or after 4 September, worth $1,654,495 of the $2,348,300 full year.
3. `## The three moves`. each with what it is, the evidence that motivates it, what it is worth at current conversion, who does it, and when.
4. `## What I need`. access, approvals, and one decision, addressed to roles not people.
5. `## What I will stop doing`. the renewals book hand-over, stated concretely.
6. `## How you will know, weekly`. the measurement, its source, and the fact that it is already automated.
7. `## What this plan does not claim`. no hook has been proven, attribution does not exist yet, and the analysis is retrospective because running anything live before the decision was not permitted.

Each section ends with a `Source:` line. Roughly 900 words total. No statistical vocabulary anywhere: no strata, no quartile, no multiple-comparison correction, no volume-adjusted. Where a statistical idea is load-bearing, say it plainly, for example "we compared like with like by grouping deals by how much conversation was logged".

- [ ] **Step 4: Check and commit**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.checks
cd ~/Documents/growth-os && grep -c $'\u2014' outbound-engine/term-4-plan.md
git add -A && git commit -m "outbound-engine: the 90-day Term 4 plan, with the number derived from the repo

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

Expected: checks `OK`, em-dash count `0`.

---

### Task 4: Rewrite the front door

**Files:**
- Modify: `/Users/cleonard1998/Documents/growth-os/README.md`
- Modify: `/Users/cleonard1998/Documents/growth-os/results/README.md`

**Interfaces:**
- Consumes: the plan from Task 3 and the two analyses from Tasks 1 and 2.

- [ ] **Step 1: Rewrite `README.md`**

Structure: one sentence saying what this is; then `## What I recommend`, pointing at the plan with its number in the first line; then `## What the evidence says`, three short findings in plain words, the null hook result, the response-speed rule, and what the sequences did; then `## Read next`, three links; then `## What this is not`, containing the sentence that running anything live before the decision was not permitted so the work to date is retrospective, and the plan is the forward half; then `## What it cost`, one line with hours, supplied by Caelum, never invented.

Remove every instance of these words from this file: strata, stratum, quartile, multiple-comparison, volume-adjusted, branch, scope, scopes, superpowers, promotion bar. Replace "promotion bar" with "the evidence bar". Keep a `Source:` line on every `## ` section.

- [ ] **Step 2: Rewrite `results/README.md` so it contains results**

Structure: `## What we found`, the three findings with their numbers, each one sentence; `## What changed as a result`, what is now recommended and what is now measured that was not before; `## The numbers behind it`, linking `baseline-2026-09.md` with the four headline figures; `## What is measured, and what is not`, linking `metric-definitions.md` and naming hook attribution as the gap; `## What has been built`, a four-line list moved here from the top and demoted. Remove the duplication with the README: the coverage percentages, the pipeline counts and the loop description all belong on one page only, and that page is not this one. Every `## ` section ends with a `Source:` line.

- [ ] **Step 3: Verify, check and commit**

```bash
cd ~/Documents/growth-os && for w in strata stratum quartile multiple-comparison volume-adjusted superpowers "promotion bar"; do printf "%-20s README %s results %s\n" "$w" "$(grep -ic "$w" README.md)" "$(grep -ic "$w" results/README.md)"; done
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.checks
cd ~/Documents/growth-os && git add -A && git commit -m "growth-os: front door leads with the recommendation, in the reader's language

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

Expected: every count `0`, checks `OK`.

---

### Task 5: Cold-read verification

**Files:** none modified unless the read fails.

- [ ] **Step 1: Run a fresh cold read**

Dispatch a subagent with no context of this work, give it only the paths to `README.md`, `results/README.md` and `outbound-engine/term-4-plan.md`, the same sceptical co-founder persona used before, and a five-minute limit. Ask it for: what was built and found, what it would do on Monday, the number being committed to, where it stumbled, and whether it would give the person the role.

- [ ] **Step 2: Judge against the bar**

The read passes when the reader can state the recommendation and the number without re-reading, and does not ask "what do I do on Monday". If it fails on either, fix the specific sentences it names and run it once more. Record the verdict, pass or fail, in `decisions/README.md` under a `## Cold read` heading with the date.

- [ ] **Step 3: Commit the verdict**

```bash
cd ~/Documents/growth-os && git add -A && git commit -m "decisions: cold-read verdict on the rewritten front door and the plan

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

---

## Self-review

**Spec coverage.** Spec 4.1 is Task 1, including the anomaly resolution in step 5 and the rule in step 6. Spec 4.2 is Task 2, including coverage honesty, touch-attribution caveat, control group, thin rule, names fetch and the top-20 shape. Spec 4.3 is Task 3, with the required section order and the derivation requirement. Spec 4.4 is Task 4, with the cold-read fix table turned into concrete edits. Spec 6 verification is spread across the check-and-commit steps plus Task 5. Spec 7 sequencing is the task order; Tasks 1 and 2 are independent and may run in parallel as the spec allows.

**Placeholders.** None. The hours figure in Task 4 step 1 is explicitly supplied by Caelum and the plan forbids inventing it, which is a constraint rather than a placeholder.

**Type consistency.** `metrics_for` returns the keys `schools`, `replied`, `reply_rate`, `meetings`, `meeting_rate`, `deals_created`, `deals_won`, `amount_won`, and `summarise` adds `sequence_id`, `name`, `sends`, `thin`; the render and the CSV column list both use exactly those names. `deal_signals` returns `days_to_first_reply` and `reply_ratio`, and `bucket_days`, `bucket_ratio` and `bucket_volume` consume exactly those. `common.py` helpers used are all existing: `read_jsonl`, `parse_ts`, `ids`, `outcome`, `is_new_business`, `SALES_PIPELINE_AU`, `OUT`, `FUNNEL`, `HISTORY`, `GOS_OUT`, `GROWTH_OS`, `GTM`.

**One known gap, deliberate.** Task 1's test `test_summarise_buckets_and_base_rate` defines a `loader` helper it does not use, kept from an earlier draft of the fixture. The implementer should delete that unused helper when writing the test rather than reproduce it.
