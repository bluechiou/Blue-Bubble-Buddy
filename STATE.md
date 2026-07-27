# STATE.md — Blue-Bubble-Buddy

Read this before doing anything in this repo; update it before you stop.
Format and rules: `.claude/skills/bbb-state-and-memory/SKILL.md`.

## Current snapshot (verified 2026-07-27)

- The library: **17** skills at `.claude/skills/<name>/SKILL.md` (was 16 —
  `bbb-knowledge-graph` added 2026-07-27). Re-verify: `ls .claude/skills/ | wc -l`.
- `bbb-knowledge-graph` is the library's **first skill with an external
  dependency** (the third-party `graphify` CLI, v0.9.27, pre-1.0). Per owner
  decision it is an *optional accelerator with a mandatory fallback* — BBB's
  portability promise means no skill may ever require it. Re-verify the tool:
  `graphify --version`.
- The behavior contract is **`b5-mode`**; remaining `f5-mode` mentions are
  historical facts about the V1.0 artifact only. Re-verify:
  `grep -rn "f5-mode" .claude/skills/ README.md | grep -v b5-mode`
  (expect 4 historical lines).
- One helper script: `bbb-visual-self-check/scripts/screenshot.mjs` (verified).
- Root docs: README.md (inventory + quick start), CLAUDE.md (session rules), this file.

## Last run — 2026-07-27 (graphify evaluation + bbb-knowledge-graph authoring)

- **Goal:** owner asked whether graphify (github.com/Graphify-Labs/graphify)
  could be integrated into BBB. Evaluate empirically, then integrate if warranted.
- **Owner decisions this run (both recorded under Decisions):** new standalone
  skill, positioned as pure optional accelerator with mandatory fallback.
- **Evidence gathered** (all against a scratchpad working copy of Blue-Booking,
  139 tracked files — no repo was modified):
  - Build is deterministic (tree-sitter, no LLM): 2915 nodes in 8.1s, 0 tokens.
  - **Scoping is decisive:** repo root → 2915 nodes / 174 communities, whose top
    navigation hubs were minified `three.min.js` internals; `graphify update src/`
    → 229 nodes / 14 communities with real hubs. One vendored file
    (`public/vendor/three.min.js`) produced 1492 of 2915 nodes — 51% of the graph.
  - `explain` / `path` are precise: a 6-line spot check of cited line numbers
    (L240/241/242/266/288/289) matched the source exactly.
  - `query` returns a truncated node neighborhood, **not an answer** — documented
    in the skill as a limit rather than a feature.
- **Authored:** `.claude/skills/bbb-knowledge-graph/SKILL.md` (183 lines;
  description 876 chars; frontmatter validated, name = directory).
- **Self-correction worth keeping:** the first draft published a python
  one-liner that printed graph.json's top-level keys instead of the indexed
  file list. Caught by executing it before shipping (non-negotiable #1) and
  replaced with a verified per-file node ranking.
- **Class 3 gate: three-reviewer workflow DISPATCHED, results not yet collected.**
  FACTUAL / DOCTRINE / USABILITY subagents were launched in parallel against a
  shared brief. **Unlike the 2026-07-07 run, independent reviewers WERE
  available — no author-equals-reviewer deviation applies to this change.**
- **Handoff:**
  - **Next action:** read the three review files, then run the single-fixer pass
    (blocking + important findings applied by ONE fixer, minor ones deferred):
    `cat /tmp/claude-0/-home-user/69135081-54ab-5898-b0ec-7ea55c0ef2fe/scratchpad/review-{factual,doctrine,usability}.md`
    If that scratchpad is gone (it is session-scoped and does not survive the
    container), the reviews must be re-run — the skill has NOT yet passed its
    Class 3 gate, and must not be merged to `main` until it has.
  - **Then apply the staged companion edits**, deliberately held back so they
    would not shift the ground under the running reviews:
    README.md inventory row + "Sixteen" → "Seventeen"; a one-clause
    cross-reference from `bbb-project-discovery` (one clause only — the graphify
    runbook lives in exactly one home).
  - **Blocking issue:** none.
  - **Files in flight:** none uncommitted — verify with `git status --porcelain=v1`.
  - **Re-verify state:** `ls .claude/skills/ | wc -l` (expect 17);
    `graphify --version` (expect 0.9.27).

## Open items / escalations

| Item | Owner | Blocked on | Since |
|---|---|---|---|
| FACTUAL review pass over the original 16 skills — re-execute every published command, fix stale ones as Class 2 | next session | session time only | 2026-07-07 |
| Collect the 3 dispatched reviews for `bbb-knowledge-graph` and run the fixer pass; skill is unmerged until then | next session | the review files (see Handoff) | 2026-07-27 |
| `b5-mode` always-on trigger and `bbb-state-and-memory` "or should have" trigger: precision unmeasured | next session | frontier problem F4 | 2026-07-07 |
| Five skills sit below the 120-line body target (95–119) — accepted; the floor is a target, not a gate. Do not pad. | — | — | 2026-07-07 |

## Decisions (one line each; stories in the archaeology chronicle / git)

- 2026-07-27 (owner): graphify is integrated as a **new standalone skill**
  (`bbb-knowledge-graph`), not as a section inside an existing skill.
- 2026-07-27 (owner): that skill is a **pure optional accelerator with a
  mandatory fallback** — BBB advertises itself as a portable library, so no
  skill may take a hard dependency on an external pre-1.0 tool.
- 2026-07-07 (owner): the behavior contract is `b5-mode`, English-only triggers;
  Fable-lineage story removed from README and the skill body.
- 2026-07-07 (owner, implicit by edit): b5-mode keeps its full Rule 3 / Rule 7
  text — overlap with `bbb-verification-and-evidence` /
  `bbb-subagent-orchestration` accepted; those skills own the mechanics.
- 2026-07-07: non-negotiables split — seven content rules in
  `bbb-change-control`, behavior rules owned by `b5-mode`.
- 2026-07-07: `.skill` ZIPs are never committed to the repo (AR-001).
