---
name: bbb-project-discovery
description: Investigate an unfamiliar repository like an incoming principal engineer BEFORE changing anything in it. Load this skill when starting work in a repo for the first time; when the user says "get familiar with this codebase", "look at this project", "onboard", "what does this project do", or hands over a repo with no context; when inheriting a project from another team or session; or before the first non-trivial change in any codebase the session has not worked in. Provides a verified command sequence for mining the manifest, build system, test reality, CI config, git history (deletions, reverts, dead branches), and TODO hotspots; the Project Brief output template; and the at-most-five-questions discipline for what the repo cannot tell you. Not for repos you already know or one-line fixes — and if STATE.md exists, read it first (see bbb-state-and-memory).
---

# Project Discovery

## Purpose

Before changing an unfamiliar repository, spend one focused pass learning what
it is, how it is actually built/run/tested, and where its scars are. The output
is a written Project Brief and at most five questions for the human. Changes
made before this pass routinely violate conventions the repo already documents,
re-fight battles the history already settled, or break workflows the CI reveals.

## When NOT to use this skill

- You already know the repo (you have a current Project Brief or STATE.md
  you've read this session).
- A trivially scoped fix in a file you were pointed at — do a targeted read of
  that file and its callers instead; a full survey wastes context.
- The repo has memory files: read `STATE.md` / `CLAUDE.md` FIRST
  (`bbb-state-and-memory`); discovery fills the gaps they leave.

## The investigation sequence

Run from the repo root. Every command below was verified 2026-07-07; all are
read-only. Commands that can legitimately return nothing are marked (∅ ok).

**0. Project memory first.**
```bash
ls CLAUDE.md STATE.md AGENTS.md .claude/ docs/ 2>/dev/null   # (∅ ok)
```
Read whatever exists before anything else — it is the previous engineers
talking directly to you.

**1. Orientation — what is this?**
```bash
cat README.md
ls package.json pyproject.toml Cargo.toml go.mod Makefile pom.xml 2>/dev/null  # (∅ ok)
```
The manifest tells you the language, entry points, and declared dependencies.

**2. Build and test reality.** What the docs SAY and what CI DOES can differ;
CI is ground truth for "how tests are actually run".
```bash
ls .github/workflows/ 2>/dev/null        # (∅ ok — then check .gitlab-ci.yml, Jenkinsfile…)
```
Read the workflow files and note the exact build/test commands they invoke.
Cross-check against the manifest's script section (e.g. `"scripts"` in
package.json) and any Makefile targets. Record BOTH if they differ — that
difference is a finding.

**3. Git history — size, shape, and scars.**
```bash
git log --oneline --graph --all | head -50    # shape and recency
git shortlog -sn --all | head -5              # who built this
git log -1 --format="%h %ad %s" --date=short  # how alive is it
```
Then mine the scars — deletions, reverts, stalled branches — with the verified
commands in `bbb-failure-archaeology` (it owns history mining). Deletions and
reverts are the highest-value lines in the whole survey: each is a decision
someone made under pressure. Record what you find as chronicle entries there.

**4. Open-work signals.**
```bash
grep -rn "TODO\|FIXME" --exclude-dir=.git --exclude-dir=node_modules . | wc -l
grep -rn "TODO\|FIXME" --exclude-dir=.git --exclude-dir=node_modules . | head -20
```
A cluster of TODOs in one module marks a hot area; treat it as fragile.

**5. Conventions.** Skim 2–3 recently-touched source files for naming, comment
density, and idiom; note generated directories (don't hand-edit those) and any
deploy/data conventions the docs or CI reveal. In a repo too large to tell which
files those should be, `bbb-knowledge-graph` is an optional accelerator for
picking them — it never replaces this sequence.

## The Project Brief

Write this down (in STATE.md if the project keeps one, else your working notes)
before the first change:

```markdown
## Project Brief — <repo> (<date>)
- Purpose: <one sentence — what this exists to do>
- Build:  <exact command>   (verified: yes/no)
- Test:   <exact command>   (verified: yes/no — CI file: <path>)
- Run:    <exact command or "library — not runnable">
- Conventions: <naming/layout/idiom notes; generated dirs>
- Hot areas: <recent churn, TODO clusters>
- Scars: <deletions/reverts worth knowing — link chronicle entries>
- Unknowns: <what the repo could not tell you>
```

Mark each command "verified" only after running it yourself
(`bbb-verification-and-evidence` defines the bar).

## The at-most-five-questions discipline

After — never before — the investigation, ask the human AT MOST five questions,
drawn only from the Unknowns list. The five canonical archetypes:

| # | Archetype | Why the repo can't answer it |
|---|---|---|
| 1 | What is the hardest live problem right now? | Priority lives in heads, not files |
| 2 | What unwritten rules exist (things you must not do that no doc states)? | Unwritten by definition |
| 3 | Who is the audience for this work, and what do they NOT know? | Context outside the repo |
| 4 | Which past failures cost the most time? | Only partially visible in history |
| 5 | What does "great" (beyond state of the art) mean here? | A judgment, not a fact |

Rules: never ask anything the repo can tell you (asking reads as not having
looked); batch all questions into one message; if the repo answered everything,
ask nothing and proceed. Fold every answer into the Project Brief, and turn
recurring answers into durable rules (`bbb-rule-distillation`).

## Provenance and maintenance

- Authored 2026-07-07 for the Blue-Bubble-Buddy skill library (Primitives
  pillar). All commands verified that day against a live repo on Linux with
  standard git; they are stable porcelain and unlikely to drift.
- Volatile facts: CI locations (`.github/workflows/` assumes GitHub Actions —
  check GitLab/Jenkins equivalents); the manifest list in step 1 reflects
  common ecosystems as of 2026 and should grow with new ones.
- Re-verify the whole sequence in one line (expect no errors, some ∅):
  `git log --oneline --graph --all | head -5; ls .github/workflows/ 2>/dev/null`
- Sibling names cited here — re-verify with `ls .claude/skills/` at the
  library root.
