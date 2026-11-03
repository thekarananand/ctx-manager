# <Project> — CLAUDE.md
> <one-line stack summary> | Updated: YYYY-MM-DD

> <operating rules: quality bar, how to SSH, which CLI commands are read-only-OK vs hand-to-user, skills to load first>

---

## 0. How to use this file

Three-step retrieval protocol:

1. **§3 FACTS first.** Stable IPs, URLs, hosts, DB names, repos, branches, project IDs.
2. **Bug/error/incident → grep `./Logbook.csv` first.** Match symptom/tags. Row found → read its incident file.
3. **Else §4 DISPATCH.** Find nearest keyword. Open the single file listed. Do not browse.

Hard rules:
- Max 2 context files per question. Needing more = dispatch is wrong; flag it.
- Infra/ops answers cite the file or command they came from. Never from memory.
- Before proposing a fix, grep Logbook.csv. `status=recurring` → fix root cause, not bandage.
- "save into context" → write under `context/<subdir>/` (ctx skill §3 SAVE). Never to agent memory dirs.
- New incident → also append Logbook.csv row (same change).
- New gotcha → append to `./gotcha.md`. Promote to §3.7 only if always-true + load-bearing.

### Root layout
```
CLAUDE.md  Logbook.csv  gotcha.md  context/  code/  __OLDFILES__/
```

## 1. What this project is (one paragraph)

## 2. Tech stack (table)

## 3. FACTS (inline — no file read needed)

Fact-writing rules:
- Every mutable value carries a verification stamp: `live YYYY-MM-DD` (+ how: command/console).
- Correcting a fact: replace the value, keep one clause of history — `corrected YYYY-MM-DD (was X — wrong)`.
- Unknown → `?` + ⚠ note. Gone resource → ⚠ **gone, verified YYYY-MM-DD**, keep row until nothing references it.
- Detail and history belong in the canonical `context/` file; keep cells short.

### 3.1 Environments / URLs
### 3.2 Accounts / projects
### 3.3 Hosts, IPs, users
### 3.4 Repos and env → branch mapping
### 3.5 Databases
### 3.6 Filesystem layout
### 3.7 Always-true gotchas (hot subset — full list in `./gotcha.md`)

| # | Gotcha | Impact |
|---|--------|--------|

Keep each row ≤ 3 lines. History goes in gotcha.md.

## 4. DISPATCH TABLE (keyword → file)

| If user asks about… | Open this file |
|---------------------|----------------|
| past incident, RCA, "has this happened before" | grep `./Logbook.csv` → linked `context/incidents/<file>` |
| any gotcha not in §3.7 | `./gotcha.md` |
| state at a date | `context/snapshots/INDEX.md` → pick dated file |
| acronym/jargon | `context/GLOSSARY.md` |

No row fits → `context/INDEX.md`. Still unclear → ask user.

## 5. Conventions — see ctx skill §3.2–3.3

## 7. Save-into-context routing — see ctx skill §3.1

## 8. Logbook.csv schema — see ctx skill §4.3
