# Message Map Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A point of view on what to say to schools, organised by who you are writing to, with the evidence generated from 842 closed deals and 612 role-attributed quotes, and three bets that can be settled inside one term.

**Architecture:** One new analysis module, `role_voice.py`, follows the shape of the existing ones: pure functions, a `summarise`, a `render_block`, a `main`. It writes only between two sentinel comments in `outbound-engine/message-map.md`, so the hand-written point of view and bets below can be regenerated around without being disturbed. Roles are collapsed from free-text job titles on cached contacts; a role counts as engaged on a deal when someone in that family wrote back inside the deal's early window.

**Tech Stack:** Python 3.9 standard library only, pytest 8 via `python3 -m pytest`, git.

## Global Constraints

- **HubSpot is read-only. No writes, no deletes, no changes of any kind, ever.** This plan makes no network calls at all; every input is a local cache file.
- Python 3.9 syntax: module docstring first, then `from __future__ import annotations`; `Optional[...]`; no `match`; no `X | Y` at runtime.
- Paths come from `common.py` constants, never hard-coded elsewhere.
- No school name, no staff name, no email address in any rendered file. Quotes pass through `write_truth.deidentify` and `tag_deals.scrub_people`, and the existing quote blocklist is respected.
- Every `## ` section of the rendered file ends with a `Source:` line. `checks.py citations` already covers `customer-truth/*.md`; this file lives in `outbound-engine/`, so Task 1 extends the check to cover it.
- Thin rule: fewer than 20 deals in a cell is thin and carries no conclusion.
- Roles only, never an individual, never a rep. `hubspot_owner_id` never appears.
- Prose: no em-dashes, sentence case headings, and no statistical vocabulary in the hand-written sections.
- Git: code commits go to the home-directory repo staging only `Documents/GTM Project/scripts/growth_os`; content commits go to `~/Documents/growth-os` and are pushed. Every commit message ends with `Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>`.
- Never delete or clean anything under `outputs/`. Fixtures use `tmp_path`.

## Domain facts (verified 18 September 2026)

- 39,291 cached contacts across `outputs/sdr_history/<company_id>.json`, of which 26,389 carry a `jobtitle` and 25,066 have both a title and an email. Top titles: principal 5,083, deputy principal 4,833, assistant principal 3,432, chaplain 614, counsellor 597, head of wellbeing 397, associate principal 382, wellbeing leader 345, guidance officer 338, acting principal 273, head 247, wellbeing coordinator 231, and noise including "ms" 213, "mr" 161, "it" 153.
- `outputs/sdr_funnel/emails.jsonl` has 48,942 inbound rows; 18,671 of them, 38 percent, have a sender that maps to a titled contact. Each row carries `id`, `hs_email_from_email`, `hs_timestamp`, `company_ids`, `hs_email_direction`.
- `deals_tagged.jsonl` holds 3,426 school-voiced tagged quotes. 689 of them, 20 percent, have an `event_id` matching an email object, and 612 of those have a titled sender. Spread: deputy principal 83, head of wellbeing 70, wellbeing leader 61, assistant principal 37, principal 34, head teacher wellbeing 13, wellbeing coordinator 11.
- In-scope population is 842 closed deals, base win rate 43.3 percent.
- The robust negative: `time_saved` is minus 13.0 points adjusted on 108 deals overall, minus 26.4 in primary, minus 29.1 in the unknown sector, minus 18.0 in independent.
- Existing suite: 169 tests, `python3 -m pytest scripts/growth_os/tests -q` from `/Users/cleonard1998/Documents/GTM Project`.

## File structure

| Path | Responsibility |
|---|---|
| `scripts/growth_os/role_voice.py` | Role taxonomy, deal-level engagement, quote attribution, render the generated block |
| `scripts/growth_os/tests/test_role_voice.py` | Unit tests on `tmp_path` fixtures |
| `scripts/growth_os/checks.py` | Extend the citations target list to cover `outbound-engine/*.md` |
| `growth-os/outbound-engine/message-map.md` | Generated evidence block plus hand-written point of view and bets |
| `GTM Project/outputs/growth_os/role_voice.json` | Machine output, never committed |

---

### Task 1: `role_voice.py`

**Files:**
- Create: `/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/role_voice.py`
- Create: `/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/tests/test_role_voice.py`
- Modify: `/Users/cleonard1998/Documents/GTM Project/scripts/growth_os/checks.py` (citations target list)
- Create (generated): `/Users/cleonard1998/Documents/growth-os/outbound-engine/message-map.md`

**Interfaces:**
- Consumes: `common.py` (`OUT`, `FUNNEL`, `HISTORY`, `GROWTH_OS`, `GOS_OUT`, `read_jsonl`, `parse_ts`, `ids`), `write_truth.py` (`deidentify`, `school_label`, `load_blocklist` if present), `tag_deals.py` (`scrub_people`)
- Produces, used by Task 2:
  - `ROLES = ["wellbeing", "curriculum", "operations", "leadership"]`
  - `role_for(title: Optional[str]) -> str` returning a role or `"other"`
  - `build_email_roles(load_contacts) -> Dict[str, str]` mapping lowercased email to role
  - `deal_roles(rec, inbound_by_company, email_roles) -> set`
  - `summarise(early, tagged, inbound, email_roles) -> dict`
  - `render_block(summary) -> str`
  - `write_between_sentinels(path, block) -> None`
  - `main(argv)`

- [ ] **Step 1: Write the failing tests**

Create `scripts/growth_os/tests/test_role_voice.py`:

```python
from __future__ import annotations
import json
from scripts.growth_os import role_voice as rv


def test_role_for_collapses_titles():
    assert rv.role_for("Deputy Principal") == "leadership"
    assert rv.role_for("Head of Wellbeing") == "wellbeing"
    assert rv.role_for("Chaplain") == "wellbeing"
    assert rv.role_for("Head of Department") == "curriculum"
    assert rv.role_for("Business Manager") == "operations"
    assert rv.role_for("IT") == "operations"
    assert rv.role_for("Ms") == "other"
    assert rv.role_for("") == "other"
    assert rv.role_for(None) == "other"


def test_role_for_resolves_overlap_towards_wellbeing():
    assert rv.role_for("Deputy Principal, Wellbeing") == "wellbeing"
    assert rv.role_for("Wellbeing Coordinator") == "wellbeing"
    assert rv.role_for("Curriculum Coordinator") == "curriculum"


def test_role_for_uses_word_boundaries():
    assert rv.role_for("Visitor Liaison") == "other"
    assert rv.role_for("Director of Learning") in ("curriculum", "leadership")


def test_build_email_roles_from_contacts():
    def load_contacts():
        yield {"email": "A.Person@school.edu.au", "jobtitle": "Head of Wellbeing"}
        yield {"email": "b@school.edu.au", "jobtitle": ""}
        yield {"email": "", "jobtitle": "Principal"}
    m = rv.build_email_roles(load_contacts)
    assert m == {"a.person@school.edu.au": "wellbeing"}


def rec(deal_id="d1", outcome="won", start="2025-03-01T00:00:00", end="2025-05-01T00:00:00"):
    return {"deal_id": deal_id, "outcome": outcome, "window_start": start, "window_end": end,
            "company_ids": ["c1"], "company_names": [], "contact_names": [],
            "segment": "Catholic", "state": "NSW", "dealtype": "newbusiness"}


def inbound(id_, ts, sender, company="c1"):
    return {"id": id_, "hs_timestamp": ts, "hs_email_direction": "INCOMING_EMAIL",
            "hs_email_from_email": sender, "company_ids": [company]}


def test_deal_roles_counts_only_inbound_inside_the_window():
    by_company = {"c1": [inbound("e1", "2025-03-10T00:00:00Z", "w@s.edu"),
                         inbound("e2", "2025-06-10T00:00:00Z", "p@s.edu")]}
    roles = rv.deal_roles(rec(), by_company, {"w@s.edu": "wellbeing", "p@s.edu": "leadership"})
    assert roles == {"wellbeing"}


def test_summarise_win_rate_by_role_engaged():
    early = [rec("w1", "won"), rec("w2", "won"), rec("l1", "lost")]
    tagged = [{"deal_id": d, "tags": [], "error": None} for d in ("w1", "w2", "l1")]
    inb = [inbound("e1", "2025-03-10T00:00:00Z", "w@s.edu"),
           inbound("e2", "2025-03-10T00:00:00Z", "w@s.edu"),
           inbound("e3", "2025-03-10T00:00:00Z", "p@s.edu")]
    for i, d in zip(inb, ("w1", "w2", "l1")):
        i["company_ids"] = [d]
    early[0]["company_ids"] = ["w1"]; early[1]["company_ids"] = ["w2"]; early[2]["company_ids"] = ["l1"]
    s = rv.summarise(early, tagged, inb, {"w@s.edu": "wellbeing", "p@s.edu": "leadership"})
    wb = s["roles"]["wellbeing"]
    assert wb["deals"] == 2 and wb["won"] == 2 and wb["rate"] == 1.0 and wb["thin"] is True
    assert s["population"]["deals"] == 3


def test_quotes_attributed_to_the_sender_role_and_scrubbed():
    early = [rec("d1", "won")]
    early[0]["company_names"] = ["Alpha Grammar School"]
    tagged = [{"deal_id": "d1", "error": None, "tags": [
        {"tag": "no_prep", "speaker": "school", "quote": "Alpha Grammar School has no time", "event_id": "e1"},
        {"tag": "no_prep", "speaker": "wellio", "quote": "ours", "event_id": "e1"}]}]
    inb = [inbound("e1", "2025-03-10T00:00:00Z", "w@s.edu", company="d1")]
    early[0]["company_ids"] = ["d1"]
    s = rv.summarise(early, tagged, inb, {"w@s.edu": "wellbeing"})
    qs = s["roles"]["wellbeing"]["quotes"]
    assert len(qs) == 1
    assert "Alpha Grammar" not in qs[0]["quote"]
    assert qs[0]["outcome"] == "won"


def test_render_block_has_source_in_every_section_and_no_em_dash():
    import re
    early = [rec("d1", "won"), rec("d2", "lost")]
    tagged = [{"deal_id": d, "tags": [], "error": None} for d in ("d1", "d2")]
    md = rv.render_block(rv.summarise(early, tagged, [], {}))
    for part in re.split(r"^## ", md, flags=re.M)[1:]:
        assert "Source:" in part
    assert "\u2014" not in md


def test_write_between_sentinels_preserves_hand_written_text(tmp_path):
    p = tmp_path / "message-map.md"
    p.write_text("# Title\n\n" + rv.START + "\nold\n" + rv.END + "\n\n## My opinion\n\nkeep me\n")
    rv.write_between_sentinels(p, "new block")
    out = p.read_text()
    assert "keep me" in out and "## My opinion" in out
    assert "old" not in out and "new block" in out
    assert out.count(rv.START) == 1 and out.count(rv.END) == 1


def test_write_between_sentinels_creates_the_file_when_absent(tmp_path):
    p = tmp_path / "new.md"
    rv.write_between_sentinels(p, "block")
    out = p.read_text()
    assert rv.START in out and "block" in out and rv.END in out
```

- [ ] **Step 2: Run the tests to verify they fail**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_role_voice.py -q 2>&1 | tail -3
```

Expected: `ModuleNotFoundError: No module named 'scripts.growth_os.role_voice'`.

- [ ] **Step 3: Write `role_voice.py`**

```python
"""Who writes back, what they say, and what it went with. Reads cached files only, no network.

The hook analysis came back null: no angle separates winners from losers. This does not retry
that question. It reports who engaged on a deal and the language they used, so a point of view
about what to say can be written on top by hand.
"""
from __future__ import annotations
import argparse
import collections
import datetime as dt
import json
import pathlib
import re
import sys
from typing import Callable, Dict, Iterable, List, Optional

from . import common as c
from .tag_deals import scrub_people
from .write_truth import deidentify, school_label

START = "<!-- generated:role-voice:start -->"
END = "<!-- generated:role-voice:end -->"
MIN_CELL = 20
MAX_QUOTES = 8

ROLES = ["wellbeing", "curriculum", "operations", "leadership"]
ROLE_PATTERNS = [
    ("wellbeing", ("wellbeing", "well-being", "chaplain", "chaplaincy", "counsellor", "counselor",
                   "guidance", "psychologist", "pastoral", "student services", "welfare")),
    ("curriculum", ("curriculum", "head of department", "head teacher", "learning", "teaching",
                    "faculty", "coordinator", "co-ordinator")),
    ("operations", ("business manager", "finance", "bursar", "it", "ict", "administration",
                    "registrar", "operations")),
    ("leadership", ("principal", "head of school", "headmaster", "headmistress", "director of",
                    "head of college", "deputy head")),
]


def role_for(title: Optional[str]) -> str:
    t = (title or "").strip().lower()
    if not t:
        return "other"
    for role, pats in ROLE_PATTERNS:
        for p in pats:
            if re.search(r"\b" + re.escape(p) + r"\b", t):
                return role
    return "other"


def default_load_contacts() -> Iterable[dict]:
    for p in c.HISTORY.glob("*.json"):
        try:
            with p.open() as f:
                d = json.load(f)
        except (ValueError, OSError):
            continue
        for ct in d.get("contacts") or []:
            yield ct


def build_email_roles(load_contacts: Callable[[], Iterable[dict]]) -> Dict[str, str]:
    out: Dict[str, str] = {}
    for ct in load_contacts():
        email = (ct.get("email") or "").strip().lower()
        role = role_for(ct.get("jobtitle"))
        if email and role != "other":
            out[email] = role
    return out


def index_inbound(rows: List[dict]) -> Dict[str, List[dict]]:
    by_company: Dict[str, List[dict]] = {}
    for r in rows:
        if r.get("hs_email_direction") != "INCOMING_EMAIL":
            continue
        for cid in c.ids(r.get("company_ids")):
            by_company.setdefault(cid, []).append(r)
    return by_company


def deal_roles(rec: dict, by_company: Dict[str, List[dict]], email_roles: Dict[str, str]) -> set:
    start, end = c.parse_ts(rec.get("window_start")), c.parse_ts(rec.get("window_end"))
    found = set()
    for cid in c.ids(rec.get("company_ids")):
        for r in by_company.get(cid, []):
            t = c.parse_ts(r.get("hs_timestamp"))
            if not (t and start and end and start <= t <= end):
                continue
            role = email_roles.get((r.get("hs_email_from_email") or "").strip().lower())
            if role:
                found.add(role)
    return found


def _cell(rows: List[dict]) -> dict:
    n = len(rows)
    won = sum(1 for r in rows if r.get("outcome") == "won")
    return {"deals": n, "won": won, "rate": round(won / n, 4) if n else None, "thin": n < MIN_CELL}


def summarise(early: List[dict], tagged: List[dict], inbound: List[dict],
              email_roles: Dict[str, str]) -> dict:
    tag_by_id = {t["deal_id"]: t for t in tagged}
    scope = [r for r in early if r.get("outcome") in ("won", "lost")
             and tag_by_id.get(r["deal_id"], {}).get("error") in (None, "", "None")]
    by_company = index_inbound(inbound)
    email_by_id = {str(r.get("id")): r for r in inbound}
    roles_of: Dict[str, set] = {r["deal_id"]: deal_roles(r, by_company, email_roles) for r in scope}

    out: Dict[str, dict] = {}
    for role in ROLES:
        eng = [r for r in scope if role in roles_of[r["deal_id"]]]
        absent = [r for r in scope if role not in roles_of[r["deal_id"]]]
        cell = _cell(eng)
        a = _cell(absent)
        cell["rate_absent"] = a["rate"]
        cell["lift_pts"] = (round((cell["rate"] - a["rate"]) * 100, 1)
                            if cell["rate"] is not None and a["rate"] is not None else None)
        tags = collections.Counter()
        for r in eng:
            for t in tag_by_id.get(r["deal_id"], {}).get("tags", []):
                if t.get("speaker") == "school":
                    tags[t["tag"]] += 1
        cell["concerns"] = tags.most_common(6)
        cell["quotes"] = []
        out[role] = cell

    attributed = 0
    for r in scope:
        row = tag_by_id.get(r["deal_id"], {})
        names = (r.get("company_names") or []) + [(r.get("dealname") or "").split(" - ")[0]]
        for t in row.get("tags", []):
            if t.get("speaker") != "school" or not t.get("quote"):
                continue
            em = email_by_id.get(str(t.get("event_id")))
            if not em:
                continue
            role = email_roles.get((em.get("hs_email_from_email") or "").strip().lower())
            if not role or role not in out:
                continue
            attributed += 1
            if len(out[role]["quotes"]) >= MAX_QUOTES:
                continue
            q = scrub_people(deidentify(t["quote"], names), r.get("contact_names") or [])
            out[role]["quotes"].append({"quote": q, "tag": t["tag"],
                                        "label": school_label(r), "outcome": r["outcome"]})

    won = sum(1 for r in scope if r["outcome"] == "won")
    total_school_quotes = sum(1 for r in scope
                              for t in tag_by_id.get(r["deal_id"], {}).get("tags", [])
                              if t.get("speaker") == "school" and t.get("quote"))
    return {
        "generated_at": dt.datetime.utcnow().isoformat(timespec="seconds"),
        "min_cell": MIN_CELL,
        "population": {"deals": len(scope), "won": won, "lost": len(scope) - won,
                       "base_rate": round(won / len(scope), 4) if scope else None},
        "coverage": {"school_quotes": total_school_quotes, "attributed_to_a_role": attributed,
                     "titled_contacts": len(email_roles)},
        "roles": out,
        "taxonomy": [{"role": r, "matches": list(p)} for r, p in ROLE_PATTERNS],
    }


def _pct(x: Optional[float]) -> str:
    return "n/a" if x is None else f"{x * 100:.1f}%"


SRC = ("Source: ~/Documents/GTM Project/outputs/growth_os/role_voice.json; computed from "
       "outputs/customer_truth/deals_early_text.jsonl, deals_tagged.jsonl, "
       "outputs/sdr_funnel/emails.jsonl and the cached contacts in outputs/sdr_history/")

LABELS = {"leadership": "the principal and their deputies", "wellbeing": "the wellbeing team",
          "curriculum": "curriculum and teaching leads", "operations": "business and IT"}


def render_block(summary: dict) -> str:
    pop, cov = summary["population"], summary["coverage"]
    L = [START, "",
         "## Who writes back, and what happened next", "",
         f"Across {pop['deals']} closed deals, base win rate {_pct(pop['base_rate'])}. A role counts as "
         "engaged when somebody in it wrote back inside the deal's early window. Writing back is not the "
         "same as being involved, and none of this shows that engaging a role causes a win.",
         "",
         "| Who wrote back | Deals | Win rate | Win rate when they did not | Difference | Note |",
         "|---|---|---|---|---|---|"]
    for role in ROLES:
        cell = summary["roles"][role]
        L.append(f"| {LABELS[role]} | {cell['deals']} | {_pct(cell['rate'])} | {_pct(cell['rate_absent'])} "
                 f"| {'n/a' if cell['lift_pts'] is None else cell['lift_pts']} "
                 f"| {'thin' if cell['thin'] else ''} |")
    L += ["", SRC, "", "## What each of them raises", ""]
    for role in ROLES:
        cell = summary["roles"][role]
        if cell["thin"] or not cell["concerns"]:
            continue
        items = ", ".join(f"{t} ({n})" for t, n in cell["concerns"])
        L.append(f"- **{LABELS[role]}**: {items}")
    L += ["", SRC, "", "## In their own words", "",
          f"{cov['attributed_to_a_role']} of {cov['school_quotes']} things schools said can be traced to "
          "the role of the person who wrote them. The rest came from meeting notes, which record no sender. "
          "So this shows what schools write, not what they say in a room, and written language runs to "
          "logistics where spoken language carries more of the reasoning.", ""]
    for role in ROLES:
        qs = summary["roles"][role]["quotes"]
        if not qs:
            continue
        L.append(f"### {LABELS[role]}")
        L.append("")
        for q in qs:
            L.append(f"- \"{q['quote']}\" ({q['label']}, {q['outcome']})")
        L.append("")
    L += [SRC, "", "## How roles were grouped", "",
          "Job titles are free text, so they were collapsed by matching the lowercased title against these "
          "words, in this order, so that a wellbeing title wins over a leadership one:", ""]
    for entry in summary["taxonomy"]:
        L.append(f"- **{entry['role']}**: {', '.join(entry['matches'])}")
    L += ["", f"Anything matching none of them is counted as other and is not reported. "
              f"{cov['titled_contacts']} contacts carry a title that maps to a role.",
          "", SRC, "", END]
    return "\n".join(L).replace("\u2014", "-")


def write_between_sentinels(path, block: str) -> None:
    p = pathlib.Path(path)
    p.parent.mkdir(parents=True, exist_ok=True)
    if not p.exists():
        p.write_text(block + "\n")
        return
    text = p.read_text()
    if START in text and END in text:
        head = text.split(START)[0]
        tail = text.split(END, 1)[1]
        p.write_text(head + block + tail)
    else:
        p.write_text(text.rstrip("\n") + "\n\n" + block + "\n")


def main(argv: Optional[List[str]] = None) -> int:
    ap = argparse.ArgumentParser(description="Who writes back and what they say. Read-only.")
    ap.add_argument("--out-md", default=str(c.GROWTH_OS / "outbound-engine" / "message-map.md"))
    args = ap.parse_args(argv)
    early = c.read_jsonl(c.OUT / "deals_early_text.jsonl")
    tagged = c.read_jsonl(c.OUT / "deals_tagged.jsonl")
    inbound = [r for r in c.read_jsonl(c.FUNNEL / "emails.jsonl")
               if r.get("hs_email_direction") == "INCOMING_EMAIL"]
    email_roles = build_email_roles(default_load_contacts)
    s = summarise(early, tagged, inbound, email_roles)
    c.GOS_OUT.mkdir(parents=True, exist_ok=True)
    (c.GOS_OUT / "role_voice.json").write_text(json.dumps(s, indent=1))
    write_between_sentinels(args.out_md, render_block(s))
    print(f"population {s['population']} | quotes attributed {s['coverage']['attributed_to_a_role']} "
          f"of {s['coverage']['school_quotes']}")
    for role in ROLES:
        cell = s["roles"][role]
        print(f"  {role:12} deals {cell['deals']:4} win {_pct(cell['rate'])} "
              f"vs {_pct(cell['rate_absent'])} quotes {len(cell['quotes'])}")
    print(f"WROTE {args.out_md}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Run the tests to verify they pass**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m pytest scripts/growth_os/tests/test_role_voice.py -q 2>&1 | tail -3
```

Expected: `10 passed`. Then the whole suite: `python3 -m pytest scripts/growth_os/tests -q`, expecting `179 passed`.

- [ ] **Step 5: Extend the citations check to cover the new file**

In `checks.py`, the citations target list currently gathers `customer-truth/*.md` with exclusions plus `markets/*.md` and `results/baseline-2026-09.md`. Add every `.md` directly under `outbound-engine/` except `README.md`. Then run the check.

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.checks citations
```

Expected: `OK`. If it names a section in an existing outbound-engine file that lacks a `Source:` line, add the line rather than narrowing the check.

- [ ] **Step 6: Run it on the real data**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.role_voice
```

Expected: a population near 842, quotes attributed near 612, and four role lines. Read the generated file. Confirm no school or staff name survived, that thin roles are marked, and that the quotes read like things a person actually wrote.

- [ ] **Step 7: Prove regeneration is safe**

Append a temporary line below the end sentinel, re-run the module, and confirm the line survives.

```bash
cd ~/Documents/growth-os && printf '\n## Scratch\n\nkeep me\n\nSource: test\n' >> outbound-engine/message-map.md
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.role_voice >/dev/null && grep -c "keep me" ~/Documents/growth-os/outbound-engine/message-map.md
```

Expected: `1`. Then remove the scratch section by hand before committing.

- [ ] **Step 8: Check and commit**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.checks
git -C ~ add "Documents/GTM Project/scripts/growth_os" && git -C ~ commit -m "growth_os: role voice, who writes back and what they say

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
cd ~/Documents/growth-os && git add -A && git commit -m "outbound-engine: message map evidence, by who you are writing to

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

Expected: checks `OK` before both commits.

---

### Task 2: The point of view and the three bets

**Files:**
- Modify: `/Users/cleonard1998/Documents/growth-os/outbound-engine/message-map.md` (hand-written sections only, below the end sentinel)

**Interfaces:**
- Consumes: the generated block from Task 1, `outputs/growth_os/role_voice.json`, `customer-truth/what-the-market-is-telling-us.md`, `customer-truth/response-speed.md`, `outbound-engine/sequence-inventory.md`, `outbound-engine/term-4-plan.md`.

Hand-written prose. No code, no tests. The gate is the structure below plus the checks.

- [ ] **Step 1: Read the generated evidence before writing**

Read the whole generated block and `role_voice.json`. Read the quotes for each role, all of them, not a sample. Note which roles are thin. A claim about a role whose cell is thin must be labelled as judgement, not measurement.

- [ ] **Step 2: Write a title and a standfirst above the start sentinel**

A heading, then two sentences: that this is a view on what to say, that the evidence behind it is generated below and can be regenerated, and that no angle has been proven to win, so the confident parts are the negative and the language, not a ranked list of hooks.

- [ ] **Step 3: Write `## What I would say to each of them`, below the end sentinel**

One short block per role that is not thin. Each gives: what they actually raise, in their words; what to say back, as an instruction a person could follow; and what to stop saying. Mark clearly which parts rest on the table above and which are judgement. Do not invent a persona for a thin role; say instead that there is not enough to go on.

- [ ] **Step 4: Write `## What to stop saying`**

The one assertion carried by measurement. Talking about saving teachers time goes with losing in every cut: minus 13.0 points on 108 deals overall, minus 26.4 in primary, minus 29.1 in the unknown sector, minus 18.0 in independent. Say what to say instead, and be explicit that this is an association on closed deals, not proof that the words caused the loss. Offer the obvious alternative reading, that a school raising time pressure is already stretched and was never going to buy, and say which of the two you believe and why.

- [ ] **Step 5: Write `## Three bets`**

Three, no more. Each: what we would send, to whom, what would count as it paying off, how we would know, and by when. Each must be settleable inside one term. State that the stamp proposed in the Term 4 plan is what makes them settleable, and that without it none of the three can ever be resolved.

- [ ] **Step 6: Write `## What would change my mind`**

For each bet, the result that would make you drop it. A bet nobody can lose is not a bet.

- [ ] **Step 7: Check and commit**

```bash
cd "/Users/cleonard1998/Documents/GTM Project" && python3 -m scripts.growth_os.checks
cd ~/Documents/growth-os && grep -c $'\u2014' outbound-engine/message-map.md
git add -A && git commit -m "outbound-engine: the message map point of view and three bets

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

Expected: checks `OK`, em-dash count `0`.

---

### Task 3: Link it and cold-read it

**Files:**
- Modify: `/Users/cleonard1998/Documents/growth-os/README.md`, `outbound-engine/term-4-plan.md`, `decisions/README.md`

- [ ] **Step 1: Link from the two places a reader will be**

In `README.md`, under the recommendation, one line pointing at the message map as the view on what to say. In `term-4-plan.md`, move three already proposes stamping every send with the angle it used; add one sentence there naming the message map as where those angles come from. Change no figure in either file.

- [ ] **Step 2: Cold read**

Dispatch a subagent with no context, give it only `outbound-engine/message-map.md`, the same sceptical co-founder persona used before, and five minutes. Ask it: what would you say differently to a school after reading this, what would prove it wrong, and does this change your view of whether the author can decide what to say rather than only measure.

- [ ] **Step 3: Judge and record**

The bar: the reader can name something they would say differently, and can name what would falsify it. If either fails, fix the specific sentences named and read once more. Record the verdict in `decisions/README.md` under the existing `## Cold read` heading with the date.

- [ ] **Step 4: Commit**

```bash
cd ~/Documents/growth-os && git add -A && git commit -m "decisions: cold-read verdict on the message map

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>" && git push
```

---

## Self-review

**Spec coverage.** Spec 3.1 taxonomy is Task 1 step 3 and rendered in the generated block. Spec 3.2 deal-level engagement is `deal_roles` and `summarise`. Spec 3.3 language bank and its coverage limit are in `summarise` and the rendered "In their own words" section. Spec 3.4 point of view is Task 2 steps 3 and 4. Spec 3.5 bets are Task 2 step 5, with step 6 adding the falsification the spec implies. Spec 4's sentinel requirement is `write_between_sentinels`, tested twice and proved on the real file in Task 1 step 7. Spec 5's citations requirement is Task 1 step 5. Spec 6 verification is spread across the check steps and Task 3.

**Placeholders.** None.

**Type consistency.** `role_for` returns the strings used as keys throughout. `summarise` produces `roles[<role>]` carrying `deals`, `won`, `rate`, `rate_absent`, `lift_pts`, `thin`, `concerns`, `quotes`, and `render_block` reads exactly those. `START` and `END` are module constants used by both `render_block` and `write_between_sentinels` and asserted in tests. `deidentify`, `scrub_people` and `school_label` are imported with their existing signatures.

**One thing the implementer must watch.** `test_summarise_win_rate_by_role_engaged` reuses the deal id as the company id so the fixture wiring stays readable. That is a fixture convenience, not a pattern to copy into the module.
