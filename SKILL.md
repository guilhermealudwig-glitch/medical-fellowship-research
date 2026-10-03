---
name: medical-fellowship-research
version: 0.1.0
description: Find and compare medical fellowships anywhere, fully cited.
author: Guilherme Ludwig, Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [fellowship, medical-education, research, career, surgery]
    related_skills: [grilling, grounded-citations]
---

# Medical Fellowship Research Skill

Research fellowships (plus observerships/research posts as separate types) for a medical professional across any set of countries, with per-program verified data, an accessibility grade, a weighted shortlist and a spreadsheet deliverable. It covers scoping → parallel country research → verification → delivery. It does NOT draft applications or apply on the user's behalf unless asked.

## When to Use

- User asks to find/analyze/compare fellowship or subspecialty training programs (any specialty, any countries — "onde fazer fellowship", "análise de fellowships", "spine fellowship options"…).
- User wants to refresh a previously built fellowship table (deadlines, salaries and heads change yearly).
- Don't use for: residency/match questions, single-program lookups, or observership-only logistics — answer those directly.

## Prerequisites

- `grilling` skill for the scoping round — load it first; this skill's Step 1 is a grilling frontier.
- `delegate_task` (one research subagent per country/region) and `web_search`/`web_extract` (spot-checks).
- XLSX output: `python -m pip install openpyxl` if missing.
- Durable output folder (default `~/Desktop/fellowships-coluna/`, or the folder the user names). NEVER the scratch dir — see Pitfalls.

## Data schema

Each country subagent writes ONE JSON file per country. Per program:

`program, institution, city, website, type, accepts_neurosurgeons|accepts_<target>, focus, head_of_spine, head_source, start_availability, application_window, contact_email, compensation, licensing_requirements, sources[]`

Top-level may be `{"country": ..., "programs": [...]}` OR a bare list — accept both. Boolean-ish fields may be strings OR `{accepts: true|null, evidence: "..."}` — accept both.

## Procedure

1. **Scoping via `grilling`.** Ask the frontier in rounds and wait; facts are yours, decisions are the user's. The canonical decision set (defaults in parentheses = what worked for the spine run):
   - What counts as a program (clinical formal = core; observership/research = separate types, kept).
   - Entry-year window (focus year X, annotate every application window and flag earlier openings).
   - Specialty focus (general scope with per-program focus tags vs. ranked sub-focus).
   - Accessibility policy (list everything; grade acesso fácil/médio/difícil — never filter silently).
   - Compensation format (annual gross, local currency + ≈ USD; no cost-of-living; unpublished = "não divulgado").
   - Who goes in the "chefe" column (fellowship director = person the user will contact; fallback: head of service, marked).
   - Contact fallback (official email → published coordinator email → alternative channel, all marked; never infer an address pattern).
   - Chat vs file split (chat: clinical formal table grouped by country + short secondary table; file: everything with source URLs).
   - Shortlist criteria (realism of entry + volume, name as tiebreak, sub-specialty interest as final tiebreak; 5–8 picks).
   - End with the shared-understanding summary and get explicit confirmation before building. **Done when:** user confirms and frontier is empty.
2. **Country fan-out.** One subagent per country in parallel, using `references/subagent-prompt.md` (schema + verification rules + write JSON to the durable folder + SHORT final message with file path and count). Pass known leads as unverified hints. **Done when:** every country has its JSON file on disk with N programs and source URLs.
3. **Normalize and build.** Run `scripts/build_fellowships.py <dataset-dir>` (openpyxl required). It classifies (clinical / obs-research / not-confirmed), grades acesso, converts money to USD, and emits `tabela-*.md` + `fellowships-<years>.xlsx`. **Done when:** row count in xlsx equals total programs across JSONs (check the printed totals).
4. **Spot-check load-bearing facts.** `web_extract` the official pages behind: every shortlist item, every published salary you will quote, every email you highlight. Fix the data when reality disagrees with the JSON. **Done when:** each shortlist claim traces to a page you read this session.
5. **Deliver.** Chat: country-grouped tables with the user's requested columns (local, contato/"chefe", entrada, e-mail, site, remuneração + foco, acesso), the secondary table, then the shortlist with concrete deadlines and a next-step offer. Files via MEDIA: (xlsx + md). Never upload to Drive without an explicit request. **Done when:** tables, shortlist and files are all in the reply.

## Pitfalls

- **Scratch dir is pruned after 72h** and Python kernel state does not survive days between turns. Artifacts (JSON research, generated tables) go to a durable folder (Desktop/Documents) from the start.
- **Long JSON in a subagent's final message trips `content_policy_blocked`.** Require file output + a 3–5 line summary. If a task still fails at the end, its research may exist only in the live transcript — but transcripts elide long blocks (`…(+N chars)`), so the data is NOT recoverable; re-run the subagent with file output rather than trying to salvage.
- **Schema drift across subagents:** boolean fields as dicts, top-level as list, and substring filters that misfire (e.g. a `"diretório"` substring filter deleted a real fellowship whose type mentioned "diretório North American Spine Society" — match on `startswith`, not `in`).
- **Never invent names, emails, salaries or dates.** Unpublished → "não encontrado" / "não divulgado", exactly. Published values can be stale: annotate the year (e.g. "AUD 90k, valor de 2018").
- **FX rates are current facts:** fetch them (e.g. `terminal`: `curl -s https://open.er-api.com/v6/latest/USD`), never guess.
- **Head of service ≠ fellowship director** in most academic centers; the contact column must be the person who receives applications.
- **Known access barriers by country** (grade with these): UK — full GMC registration to practise; CCT/NTN-requiring posts (STIG/JCST, RNOH) are not open to foreign trainees directly; Nottingham CSSS uniquely sponsors GMC registration for non-EU grads. Canada — provincial licence (CPSO/CPSA/CPSBC), English tests (IELTS/OET/TOEFL), some posts require citizenship/PR (VINF); a few programs run match (SF Match) with a separate direct route for foreign-trained. Australia — AHPRA registration as Clinical Fellow/Surgical Assistant (not recognised as a specialist), PFET posts want specialist assessment/FRACS. Brazil — CRM + título de especialista.
- **Deadlines move yearly.** Always print the window you verified plus "confirmar abertura" for future cycles.

## Verification

- Program count in every country JSON == sum printed by the build script == xlsx data rows.
- Every row carries at least one source URL; spot-checked fields match the live page.
- Shortlist items each have: live window/deadline, published compensation (or explicit "não divulgado"), and the graded acesso.
- Reply includes: main table, secondary table, shortlist, and MEDIA: deliverables.
