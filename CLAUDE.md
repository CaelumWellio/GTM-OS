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
