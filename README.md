# medical-fellowship-research

Hermes skill: research and compare medical fellowships (spine, skull base, any specialty) across countries — fully cited, with an accessibility grade, a weighted shortlist and a spreadsheet deliverable.

## What it does

1. **Scoping** (via the `grilling` skill): settles program scope, entry-year window, accessibility policy, compensation format, contact definition and shortlist criteria with the user.
2. **Country fan-out**: one research subagent per country collects every program that accepts the target profile, with verified names, emails, windows and salaries — never invented; unpublished data is marked `não encontrado` / `não divulgado`.
3. **Build**: `scripts/build_fellowships.py` normalizes the per-country JSONs into a country-grouped Markdown table plus a full `.xlsx` (with source URLs per row and licensing/visa notes).
4. **Verification**: load-bearing facts (shortlist, salaries, emails) are spot-checked against official pages before delivery.

## Install (Hermes)

Copy this folder to your profile's skills directory, e.g.:

```
~/.hermes/skills/research/medical-fellowship-research/
```

or, on Windows:

```
%LOCALAPPDATA%\hermes\profiles\<profile>\skills\research\medical-fellowship-research\
```

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Full workflow, decision checklist, per-country access barriers, pitfalls |
| `scripts/build_fellowships.py` | Table + XLSX generator (needs `openpyxl`) |
| `references/subagent-prompt.md` | Prompt template for the per-country research subagents |

## Notes

- Data (the actual program table) is a deliverable, not part of the skill — deadlines and salaries change yearly.
- Built from a real run: 76 spine fellowship programs across Brazil, Canada, the UK and Australia (Sept/Oct 2026).
