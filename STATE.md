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
- **Class 3 gate: COMPLETE.** FACTUAL / DOCTRINE / USABILITY reviewers ran
  independently in parallel from a shared brief; all three returned
  SHIP WITH FIXES (4 blocking findings total, 13 important, 15 minor). One
  fixer (this session) applied every blocking and important finding; minors are
  listed below as deferred. **Unlike the 2026-07-07 run, independent reviewers
  WERE available — no author-equals-reviewer deviation applies to this change.**
- **What review caught that self-review had not** (the reason the gate exists):
  - *Blocking:* the skill published `graphify query "how is overselling
    prevented?"` as returning 85 nodes. The author had actually run a different
    string; the published one returns `No matching nodes found`. Two reviewers
    found this independently. The corrected rule is better than the original
    claim: **`query` seed-matches identifiers, not English prose.**
  - *Blocking:* step 1 told the reader to confirm `Token cost: 0 input · 0
    output` from the build output. That line never appears in the terminal — it
    is in `GRAPH_REPORT.md`. A literal reader would have aborted good runs.
  - *False claim:* "output goes to `src/graphify-out/`, not the current
    directory" — a scoped build *also* writes a stub `graphify-out/manifest.json`
    in the CWD, so the default path looks valid while holding no graph.
  - *Oversell:* an unmeasured "hundreds of thousands of tokens" headline, and a
    hub table whose "minified junk" column silently omitted that `worker.js`
    still ranks #3 unscoped. Both corrected to verbatim, attributed output.
  - *Scope risk:* `graphify install --project --strict` installs a hook that
    blocks file reads until a query runs — it would convert the owner-mandated
    optional accelerator into a hard gate. Now explicitly fenced off.
  - *Confidentiality:* `extract`, `label`, and `cluster-only --backend` ship
    code-derived content to an external LLM provider. Now a trust boundary.
- **Determinism, measured this run:** two independent builds of identical input
  gave identical nodes/edges (2915 / 5146) but different community counts
  (174 vs 175). The skill now says to treat nodes and edges as reproducible and
  clustering as advisory.
- **Handoff:**
  - **Next action:** none required for this change — it is complete and pushed.
    PR #9 is a draft; converting it to ready-for-review is the owner's call.
  - **Deferred minor findings** (fix opportunistically, Class 1–2): the `path`
    relation list may still be incomplete beyond the four documented; build
    timings in the skill are not independently reproduced; the vendor graphify
    skill at `/root/.claude/skills/graphify/` competes for the same triggers in
    any session where both are loaded (documented in the skill, not resolvable
    from inside this repo).
  - **Blocking issue:** none.
  - **Files in flight:** none uncommitted — verify with `git status --porcelain=v1`.
  - **Re-verify state:** `ls .claude/skills/ | wc -l` (expect 18 — 17 BBB
    skills plus the vendored `ui-ux-pro-max`);
    `graphify --version` (expect 0.9.27);
    `grep -c bbb-knowledge-graph README.md` (expect 1).

## Open items / escalations

| Item | Owner | Blocked on | Since |
|---|---|---|---|
| FACTUAL review pass over the original 16 skills — **may already be done**: unmerged branch `claude/file-reading-setup-dtg41w` (commit `c88323e`, 2026-07-17) carries a STATE.md declaring the debt PAID, with one Class 2 fix to `bbb-rule-distillation`. Decide whether to merge that branch or redo the pass; do not redo it blindly | owner | decision on that branch | 2026-07-27 |
| Vendored `ui-ux-pro-max` (PR #8) sits in `.claude/skills/` but is absent from the README inventory and does not follow this library's format spec. Decide whether it is in scope for the library or should be marked as vendored/exempt | owner | scope decision | 2026-07-27 |
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
