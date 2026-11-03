# ctx-manager

A Claude Code skill that gives a project a **knowledge base built for AI agents** — and keeps it honest.

Most project docs fail agents the same way: facts are scattered, nobody knows which file is current, the same bug gets re-debugged every few months, and the index rots. `ctx` fixes that with a fixed structure and a small set of rules the agent follows every time it reads or writes project knowledge.

Goal: any agent — even a small model — answers ~80% of questions from **one** file and the rest from **at most two more**, without browsing directories.

## The structure

```
project/
  CLAUDE.md       hot facts + operating rules + DISPATCH table (auto-loaded every session)
  Logbook.csv     append-only incident ledger — grep BEFORE debugging
  gotcha.md       every known trap, numbered G1, G2, … (canonical)
  context/        detail, one topic per file, typed by folder
    INDEX.md      mirror of the dispatch table
    architecture/ infra/ runbooks/ incidents/ snapshots/
    decisions/ integrations/ data/ features/ research/
  __OLDFILES__/   untriaged legacy — read only on explicit ask
```

| Tier | File | Holds | Read when |
|------|------|-------|-----------|
| 0 | `CLAUDE.md` | hot facts, hot gotchas, dispatch table | every session |
| 1 | `Logbook.csv` | one row per incident | any bug or error, before debugging |
| 1 | `gotcha.md` | every gotcha | gotcha not in the hot subset |
| 2 | `context/<type>/*.md` | detail | only via a dispatch row |

Every context file carries frontmatter with a **status**:

- `durable` — won't change
- `current` — mutable truth, edited in place
- `frozen` — dated (`YYYY-MM-DD-slug.md`), never edited; new state means a new file

## The five modes

| Mode | Triggered by | What it does |
|------|--------------|--------------|
| **INIT** | "set up context", "create a knowledge base" | Builds the skeleton from templates, migrates existing docs, verifies facts before writing them |
| **RECALL** | any project question, any bug | Facts → Logbook → one dispatch file → INDEX → ask. Hard 2-file limit, no browsing, cites sources |
| **SAVE** | "save into context", "add a gotcha", "write a runbook" | Routes by kind, adds frontmatter, stamps facts (`live YYYY-MM-DD`), updates dispatch + indexes in the same change |
| **INCIDENT** | "RCA", "post-mortem", "log this issue" | Recurrence check (`status=recurring`), frozen incident file, validated Logbook row, extracts gotchas and corrects wrong facts |
| **AUDIT** | "audit context", periodic hygiene | 12 checks: dead links, orphans, index drift, frontmatter, stale stamps, contradictions, open ⚠ loops, gotcha bloat, Logbook schema, size, strays |

## Key rules

- **Facts are stamped.** `live 2026-10-04` + how verified. Corrections keep one clause of history: `corrected 2026-08-14 (was X — wrong)`. Unknowns are `?` + ⚠, never guesses.
- **Ledger before debugging.** Same root cause seen twice → `recurring` → fix the system, not the symptom.
- **Gotchas are numbered and never reused.** Promoted to the hot list in `CLAUDE.md` only if always-true *and* load-bearing; hot rows stay ≤ 3 lines.
- **Bookkeeping in the same change.** New file ⇒ dispatch row + INDEX row + `updated:` bump. This is the part that rots if skipped; AUDIT catches it when it does.
- **Knowledge lives in the repo**, not in the agent's private memory.

## Install

```bash
git clone https://github.com/thekarananand/ctx-manager.git ~/.claude/skills/ctx
```

Project-only install instead:

```bash
git clone https://github.com/thekarananand/ctx-manager.git .claude/skills/ctx
```

Claude Code picks it up automatically. Invoke with `/ctx`, or just say "set up context for this project", "save into context", "write the RCA", or "audit context".

Update: `git -C ~/.claude/skills/ctx pull`

## Files

```
SKILL.md              the skill (all five modes)
templates/CLAUDE.md   root file skeleton: rules, facts, dispatch
templates/gotcha.md   gotcha file skeleton + entry format
```

## Using it with other agents

Add a one-line `AGENTS.md` at the project root pointing at `CLAUDE.md` so Cursor, Codex, Gemini, etc. follow the same protocol. Don't duplicate content there.
