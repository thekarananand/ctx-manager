---
name: ctx
description: Project knowledge base ("second brain") for AI agents — root CLAUDE.md with hot FACTS + DISPATCH table, append-only Logbook.csv incident ledger, numbered gotcha.md, and a typed context/ directory with indexes. Use to (1) INIT one ("set up context", "create a knowledge base", "structure our docs for agents"), (2) RECALL from one — any project infra/ops/architecture question, "has this happened before", "what is the IP/branch/DB for X", or before debugging any error, (3) SAVE into one ("save into context", "document this", "add a gotcha", "update the facts", "write a runbook", "snapshot state"), (4) log an INCIDENT ("RCA", "post-mortem", "log this issue", "add to logbook", after any non-trivial debug), (5) AUDIT one ("audit context", "check indexes", "context hygiene", periodic maintenance).
---

# ctx — project knowledge base

Goal: any agent, even a small model, answers ~80% of questions from one file and the rest from at most 2 more, without browsing.

Pick the mode, jump to its section:

| User wants… | Mode |
|-------------|------|
| new / restructured knowledge base | §1 INIT |
| an answer, or about to debug | §2 RECALL |
| to record a fact, runbook, snapshot, ADR, gotcha | §3 SAVE |
| to record an incident / RCA | §4 INCIDENT |
| to check / clean the knowledge base | §5 AUDIT |

Templates: `templates/CLAUDE.md`, `templates/gotcha.md` (in this skill dir).

---

## 0. The model

| Tier | File | Holds | Read when |
|------|------|-------|-----------|
| 0 | `CLAUDE.md` (root) | operating rules, hot FACTS, hot gotchas, DISPATCH table | every session, auto-loaded |
| 1 | `Logbook.csv` | one row per incident, append-only | any bug/error, BEFORE debugging |
| 1 | `gotcha.md` | every gotcha, numbered `G<n>`, canonical | gotcha not in hot subset |
| 2 | `context/<type>/*.md` | detail, one topic per file | only via DISPATCH row |
| — | `__OLDFILES__/` | untriaged legacy | only on explicit user ask |

Why: hot facts inline → common case needs zero reads. Ledger grep before debugging → recurrences caught, not re-debugged. Dispatch maps keyword → exactly one file → no `ls -R`. Status per file tells whether it can be trusted as "now".

File status:
- `durable` — won't change.
- `current` — mutable truth, edit in place.
- `frozen` — dated, **never edit body**; when state moves, write a new dated file.

Never write project knowledge to `~/.claude/projects/.../memory/` — it lives in the repo.

---

## 1. INIT

1. **Inventory first.** List what exists (root files, docs dirs, READMEs, runbooks, chat dumps). Delete nothing. Anything not triaged now → `__OLDFILES__/`.
2. **Skeleton:**
   ```
   CLAUDE.md
   Logbook.csv          header only: date,env,severity,symptom,root_cause,fix,incident_file,status,tags
   gotcha.md
   context/INDEX.md     dispatch mirror for agents that did not load CLAUDE.md
   context/GLOSSARY.md
   context/{architecture,infra,runbooks,incidents,snapshots,decisions,integrations,data,features,research}/
   context/{incidents,snapshots,decisions}/INDEX.md
   ```
   Only create subdirs needed now; add others when their first file arrives. No empty placeholder files.
3. **CLAUDE.md** from `templates/CLAUDE.md`. Fill §3 FACTS only with values verified this session (command run or file read), stamped `live YYYY-MM-DD`. Unknown → `?` + ⚠, never a guess.
4. **gotcha.md** from `templates/gotcha.md`.
5. **Migrate docs** one at a time via §3 SAVE routing: frontmatter, rename, dispatch row. Old point-in-time notes → `frozen` snapshots.
6. **Run §5 AUDIT** to confirm every dispatch row resolves, nothing orphaned.
7. Other agents (Cursor/Gemini/Codex) on the repo → one-line `AGENTS.md` pointing at `CLAUDE.md`. Don't duplicate content.

Canonical vs hot: `context/infra/*` is canonical; CLAUDE.md §3 is the hot copy. Never two copies without knowing which wins.

---

## 2. RECALL

```
question
  ├─ in CLAUDE.md §3 FACTS? ───────────► answer, cite "§3.x". done.
  ├─ bug / error / symptom? ───────────► grep Logbook.csv (symptom + tags + root_cause)
  │                                       row found → read its incident_file → answer
  │                                       status=recurring → say so, push root cause
  ├─ §4 DISPATCH row matches? ─────────► read that ONE file → answer
  ├─ context/INDEX.md → ONE file ──────► answer
  └─ still unclear ────────────────────► ask user. do not browse.
```

Rules:
- **Max 2 context files per question.** Urge to read a third = dispatch gap. Answer, then flag: "no dispatch row for <topic>; suggest `<keywords> → <file>`".
- **Grep big files**, don't read: `grep -n -i '<term>' Logbook.csv gotcha.md`; read gotcha.md only around the matching `### G<n>`.
- **"What is current"** → `durable`/`current` files or CLAUDE.md §3. Never from a `frozen` snapshot/incident.
- **Stale stamp** on a fact that matters (prod change, cost, security) → re-verify with a read-only command before acting; refresh the stamp via §3.
- **Conflict** between CLAUDE.md §3 and canonical file → trust newer stamp, say which, fix the other.
- **Cite** the file path or command for every infra answer. Never answer infra from memory/training.
- **Refuse:** "explore context/", "read all of infra/ to be safe", "read `__OLDFILES__` for background".

Before proposing any fix:
1. Grep Logbook.csv for symptom AND suspected root cause.
2. Same root_cause, `status≠resolved` → prior work exists; read it, extend it, don't duplicate.
3. Grep gotcha.md for the component name.

---

## 3. SAVE

### 3.1 Route

| Kind | Destination | status |
|------|-------------|--------|
| how-to, step-by-step | `context/runbooks/<slug>.md` | current |
| point-in-time audit / state dump | `context/snapshots/YYYY-MM-DD-<slug>.md` + row in `snapshots/INDEX.md` | frozen |
| incident / RCA | **§4 INCIDENT** | frozen |
| site quirk, framework trap, deploy edge case | append `### G<next>` to `./gotcha.md` (§3.5) | — |
| why X over Y | `context/decisions/ADR-NNN-<slug>.md` + row in `decisions/INDEX.md` | durable |
| current infra fact (IP, scope, config, branch) | `context/infra/<area>.md` AND CLAUDE.md §3 if hot | current |
| how it fits together | `context/architecture/<slug>.md` | current |
| third-party integration | `context/integrations/map.md` | current |
| feature spec | `context/features/<feature>.md` | current |
| schema / model | `context/data/` | current |
| market / competitor | `context/research/` | current |

**Update before create.** `grep -ril '<topic>' context/` first. One topic per file; two H1s → split.

### 3.2 Frontmatter (every context file)

```yaml
---
title: short noun phrase
type: architecture|decision|infra|runbook|incident|snapshot|integration|data|feature|research
status: durable|current|frozen
tags: [grep, friendly, keywords]
updated: YYYY-MM-DD
related: [context/path/a.md]   # max 3
---
```
First 10 lines answer: what is this, what does it apply to, when does it apply.

### 3.3 Naming and links

- current/durable: `kebab-case.md`, no date. frozen: `YYYY-MM-DD-short-slug.md`.
- Links absolute from project root: `[deploy](context/runbooks/deploy.md)`. No `../`.

### 3.4 Facts

- Stamp mutable values: `live YYYY-MM-DD` + how verified (command, console, file).
- Correcting: new value + `corrected YYYY-MM-DD (was X — wrong)`. One clause, not a saga; history goes to the canonical file.
- Unknown → `?` + ⚠. Never fill from memory or inference.
- Undocumented resource found → add with `⚠ undocumented until YYYY-MM-DD, purpose unknown`.
- Resource gone → `⚠ gone, verified YYYY-MM-DD`; keep row until nothing references it.

### 3.5 Gotchas

- Next number = highest `### G<n>` + 1. Never renumber or reuse.
- Format: grep-able claim title → Impact → Condition → Fix → Verified → Logbook → Related.
- Promote to CLAUDE.md §3.7 only if **always-true AND load-bearing** (wrong = outage or data loss). Hot row ≤ 3 lines; history stays in gotcha.md.
- Gotcha proved partly wrong → add `Corrected YYYY-MM-DD:` line; never silently rewrite.

### 3.6 Same-change bookkeeping (rots if skipped)

- [ ] `updated:` bumped on every file touched
- [ ] new file → DISPATCH row in CLAUDE.md §4 **and** `context/INDEX.md` (identical rows; INDEX.md is the mirror, CLAUDE.md wins)
- [ ] snapshot/ADR/incident → row in that subdir's `INDEX.md`
- [ ] hot fact changed → CLAUDE.md §3 + its header `Updated:` date
- [ ] `related:` on new file + back-link from closest sibling

Never: edit a frozen body (at most a `> Superseded by <file>` line on top); paste long raw logs (quote decisive line, bulk output into a snapshot).

---

## 4. INCIDENT

### 4.1 Recurrence check first

```
grep -i -E '<symptom words>|<root cause words>' Logbook.csv
```
- Same root_cause exists → occurrence #2+: set prior row's `status` to `recurring` (only that field) AND append a new row. Writeup must push a systemic fix.
- Similar symptom, different cause → new row; incident file says "not the same as <file> because …".

### 4.2 Incident file

Path: `context/incidents/YYYY-MM-DD-<env>-<symptom-slug>.md` (date = first observed).

```yaml
---
title: <env> <symptom>
type: incident
status: frozen
date: YYYY-MM-DD
env: <env>
severity: P0|P1|P2|P3
tags: [...]
updated: YYYY-MM-DD
related: [max 3]
---
```

Sections (skip empty, don't pad):
1. **Summary** — 3 lines: what broke, impact window, root cause in one phrase.
2. **Timeline** — timezone stated on every timestamp. Convert scheduler/local times before correlating with logs.
3. **Evidence** — decisive query/command + shortest decisive output line. Each claim marked **confirmed** or **hypothesis**.
4. **Root cause** — mechanism, not just trigger. Not found → "not identified" + what was ruled out and how.
5. **Fix** — applied vs pending, exact commands/PRs, who approved prod changes.
6. **Follow-ups** — actionable open items.
7. **Gotchas extracted** — `G<n>` numbers.

Frozen after writing. Later findings → append `## Update YYYY-MM-DD`; new occurrence → new file. Mark old conclusions `superseded`, never rewrite them.

### 4.3 Logbook.csv row (same change)

Header (exact): `date,env,severity,symptom,root_cause,fix,incident_file,status,tags`

| col | rule |
|-----|------|
| date | first observed, YYYY-MM-DD |
| env | project env name or `all` |
| severity | P0 outage · P1 degraded · P2 user-visible bug · P3 minor |
| symptom | terse, grep-friendly, the words an operator would type |
| root_cause | one phrase; `unidentified` if so |
| fix | one phrase or `pending` |
| incident_file | path, or empty if too small for a file |
| status | open / resolved / recurring |
| tags | space-separated |

- **Append-only.** Only `status` of old rows may change.
- Quote fields with commas. Validate: `python3 -c "import csv;b=[r for r in csv.reader(open('Logbook.csv')) if len(r)!=9];print(b or 'ok')"`.

### 4.4 Lessons

- Non-obvious trap hit → gotcha (§3.5), linked both ways.
- Fact found wrong during triage → correct it (§3.4).
- Runbook step failed or missing → fix the runbook.
- Row in `context/incidents/INDEX.md` if it exists.

---

## 5. AUDIT

Report first, fix second. Mechanical fixes (dead link to renamed file, missing INDEX row, `updated:` bump) → apply. Judgment calls (delete file, change fact, demote hot gotcha) → list for user. Use shell; don't read whole files.

1. **Dead dispatch links** — every path in CLAUDE.md §4 and `context/INDEX.md` exists.
   `grep -ohE 'context/[A-Za-z0-9_./-]+\.md' CLAUDE.md context/INDEX.md | sort -u | while read f; do [ -f "$f" ] || echo "DEAD $f"; done`
2. **Orphans** — `context/` files reachable from no dispatch row, subdir INDEX, or Logbook row.
3. **Mirror drift** — CLAUDE.md §4 vs `context/INDEX.md` rows differ (missing rows, stale wording, moved section numbers). Regenerate INDEX.md from CLAUDE.md.
4. **Frontmatter** — all fields present; `type` matches subdir; frozen files named `YYYY-MM-DD-*`; `related:` ≤ 3 and resolvable.
5. **Frozen edited** — frozen body rewritten (appended `## Update` sections are fine).
6. **Stale stamps** — hot-fact `live/verified YYYY-MM-DD` older than ~60 days. List with the read-only command that re-verifies each.
7. **Contradictions** — same resource with different values across CLAUDE.md §3, `context/infra/*`, gotcha.md. Newest stamp wins; flag rest.
8. **Open loops** — ⚠ "not yet resolved", "flagged to user", "cause unconfirmed", `?` cells, with age. This is the to-do list.
9. **Gotchas** — no duplicate `G<n>`; every §3.7 row exists in gotcha.md, is always-true + load-bearing, ≤ 3 lines; promotion candidates = gotchas cited by ≥ 2 incidents.
10. **Logbook** — 9 fields, valid dates, severity P0–P3, status valid, `incident_file` exists. Same root_cause on ≥ 2 rows not `recurring` → flag. `open` > 30 days → list.
11. **Size** — CLAUDE.md past ~400 lines or with paragraph-sized cells → move history out to canonical files. Hot = needed on most turns, nothing else.
12. **Strays** — `*.zip`, logs, scratch scripts, dumps in `context/` or root → propose `__OLDFILES__/` or a snapshot.

Output:
```
ctx-audit YYYY-MM-DD
FIXED    <n> (list)
DEAD     <path> ← <file>
ORPHAN   <path>
DRIFT    <row> — CLAUDE.md vs INDEX.md
STALE    <fact> stamped <date> — verify: <command>
CONFLICT <resource>: <A> (<file>) vs <B> (<file>)
OPEN     <note> (<age>d) — <file>
GOTCHA   <issue>
LOGBOOK  <issue>
STRAY    <path>
```
Save as `context/snapshots/YYYY-MM-DD-ctx-audit.md` (frozen) if user wants a record.
