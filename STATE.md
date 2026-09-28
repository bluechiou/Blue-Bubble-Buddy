# STATE.md — Blue-Bubble-Buddy

Read this before doing anything in this repo; update it before you stop.
Format and rules: `.claude/skills/bbb-state-and-memory/SKILL.md`.

## Current snapshot (verified 2026-07-27)

- The library: **17 BBB-authored skills** at `.claude/skills/<name>/SKILL.md`
  (was 16 — `bbb-knowledge-graph` added 2026-07-27), plus **one vendored
  third-party skill**, `ui-ux-pro-max`, merged from PR #8 on 2026-07-27 and not
  authored to this library's format spec. So the directory count is **18** and
  the library count is 17 — state which one you mean.
  Re-verify: `ls .claude/skills/ | wc -l` (expect 18);
  `ls .claude/skills/ | grep -c '^bbb-\|^b5-mode'` (expect 17).
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

## Last run — 2026-09-28 (network-wide "Talking to Blue" rule)

- **Goal:** the owner asked that every agent in the ÆTHNOUS network know the new
  "Talking to Blue" rule (engine repo `DECISIONS.md` #20).
- **Outcome:** CLAUDE.md item 6 added; no skill touched, so no change-control
  class applies (CLAUDE.md is not under `.claude/skills/`). Evidence:
  `grep -c "Talking to Blue" CLAUDE.md` → 1.
- **Also corrected:** the previous handoff said PR #9 was still a draft — it was
  merged on 2026-07-27 (verified on GitHub 2026-09-28). The deferred minor
  findings from that run moved to Open items below.
- **Handoff:**
  - **Next action:** none — the change is on a draft PR; merging is the owner's call.
  - **Blocking issue:** none.
  - **Files in flight:** none uncommitted — verify with `git status --porcelain=v1`.
  - **Re-verify state:** `grep -c "Talking to Blue" CLAUDE.md` (expect 1).
    The snapshot above was last verified 2026-07-27 and was not re-verified this run.

## Open items / escalations

| Item | Owner | Blocked on | Since |
|---|---|---|---|
| FACTUAL review pass over the original 16 skills — **may already be done**: unmerged branch `claude/file-reading-setup-dtg41w` (commit `c88323e`, 2026-07-17) carries a STATE.md declaring the debt PAID, with one Class 2 fix to `bbb-rule-distillation`. Decide whether to merge that branch or redo the pass; do not redo it blindly | owner | decision on that branch | 2026-07-27 |
| Vendored `ui-ux-pro-max` (PR #8) sits in `.claude/skills/` but is absent from the README inventory and does not follow this library's format spec. Decide whether it is in scope for the library or should be marked as vendored/exempt | owner | scope decision | 2026-07-27 |
| Deferred minor findings from the 2026-07-27 `bbb-knowledge-graph` review: the `path` relation list may be incomplete beyond the four documented; build timings in the skill are not independently reproduced; the vendor graphify skill at `/root/.claude/skills/graphify/` competes for the same triggers when both are loaded (documented in the skill) | next session (Class 1–2, opportunistic) | — | 2026-07-27 |
| `b5-mode` always-on trigger and `bbb-state-and-memory` "or should have" trigger: precision unmeasured | next session | frontier problem F4 | 2026-07-07 |
| Five skills sit below the 120-line body target (95–119) — accepted; the floor is a target, not a gate. Do not pad. | — | — | 2026-07-07 |

## Decisions (one line each; stories in the archaeology chronicle / git)

- 2026-09-28 (owner): network-wide "Talking to Blue" rule added to CLAUDE.md
  (item 6); source: ÆTHNOUS engine repo `DECISIONS.md` #20.
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
