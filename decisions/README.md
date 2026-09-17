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

## Week 2 gate

Date: 2026-09-17.

- Test suite: `python3 -m pytest scripts/growth_os/tests -q` from `~/Documents/GTM Project`, 100 passed.
- Checks: `python3 -m scripts.growth_os.checks`, OK (citations, de-identification, read-only, people, all clear).
- Plugin scope: `gtm-skills@gtm-plugins` confirmed absent from the user-level `~/.claude/settings.json` `enabledPlugins`; the plugin remains project-scoped to this repo only.
- Cold read timing: pending, Caelum to time.

Source: scripts/growth_os/tests; scripts/growth_os/checks.py; ~/.claude/settings.json.
