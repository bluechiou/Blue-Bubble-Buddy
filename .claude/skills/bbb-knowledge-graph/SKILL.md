---
name: bbb-knowledge-graph
description: Build a call-graph of an unfamiliar codebase with the third-party graphify CLI, then use it to find entry points and trace call paths in seconds instead of reading files blind. Load this skill when onboarding into a large or unfamiliar repo and the file tree is too big to read; when the user asks "where do I start", "what are the core abstractions", "what calls X", "how does X reach Y", or "map this codebase"; or when a discovery pass is stalling on which files matter. Provides the verified install, the scoping rule that decides whether output is signal or noise, the explain/path runbook, and the honest limits of its query command. This is an optional accelerator for ONE step of bbb-project-discovery, which remains the primary onboarding runbook and the fallback whenever this tool is unavailable or noisy. Not for reasoning about a bug (bbb-debugging-playbook) and not for cross-session state (bbb-state-and-memory).
---

# Knowledge Graph — accelerated codebase orientation

## Purpose

A large unfamiliar repo can burn a lot of a session's context on blind file
reading before it knows which files matter. `graphify` parses the repo into a
call-graph with tree-sitter (a deterministic parser — no model, no API call) and
answers three structural questions directly: what are the most connected
abstractions, what does this symbol call, and how does A reach B. Use it to
decide *what to read*; keep reading the source to decide *what is true*.

This skill is an **optional accelerator** for a single step of
`bbb-project-discovery` — its "which files matter" step. Discovery remains the
primary onboarding runbook and the fallback: if `graphify` is not installed, or
its output looks like noise, run the manual investigation sequence there and
lose nothing but speed.

## When NOT to use this skill

- **Diagnosing a bug or regression** — a graph shows structure, never causality.
  Use `bbb-debugging-playbook`.
- **A repo you already know, or a one-file fix** — building a graph to change
  one function is pure overhead. Read the file and its callers.
- **Carrying state between sessions** — the graph maps *code*, not *work in
  progress*. `bbb-state-and-memory` owns STATE.md; a graph never replaces it.
- **The full discovery pass** — manifest, CI, git scars, and TODO hotspots are
  owned by `bbb-project-discovery`. This skill accelerates one of its steps.
- **A confidential repo, for any subcommand outside this runbook** — see
  "Trust boundaries" before running `extract`, `label`, or `cluster-only`.
- **Anything requiring installs you cannot make** — no network or no `uv`
  means fall back; do not block a task on this tool.

## Install (verified 2026-07-27)

```bash
uv tool install graphifyy        # note the trailing "yy" — package name differs from the command
graphify --version               # expect: graphify 0.9.27
```

`graphify install` additionally copies a graphify skill into your assistant's
config directory and writes a `CLAUDE.md` section. **You do not need it** — the
CLI alone provides everything below. Two cautions if you run it anyway:

- **Never run `graphify install --project --strict`** in a repo that uses this
  library. That flag installs a hook that blocks the session's first raw file
  read until a `graphify query` runs, which converts an optional accelerator
  into a hard gate on ordinary work and makes the mandated fallback unreachable
  (verified 2026-07-27 via `graphify install --help`).
- The vendor skill it installs is scoped more aggressively than this one: its
  description tells a model to treat *any* codebase question as a graphify query
  first. In a repo using this library, this skill's optional-accelerator rule and
  `bbb-project-discovery` take precedence.

## The scoping rule — the single thing that decides signal vs noise

**Point graphify at source directories, never at a repo root that contains
vendored or minified files.** Minified bundles explode into thousands of
one-letter nodes that dilute the report.

Measured on one repo (139 tracked files, Cloudflare Workers + JS, 2026-07-27).
The hub column quotes the **first five entries verbatim, in report order**:

| Build target | Nodes | Communities | Community Hubs, first five as printed |
|---|---|---|---|
| Repo root (contains `three.min.js`, vendored skill scripts) | 2915 | 174 | `three.min.js`, `vt`, `worker.js`, `en`, `sn` |
| Root minus vendor/minified/lockfiles | 327 | 19 | `worker.js`, `chart-api.js`, `flow.js`, `liushi.js`, `EffectComposer` |
| `src/` only | 229 | 14 | `flow.js`, `chart-api.js`, `route`, `liushi.js`, `worker.js` |

The unscoped run is **diluted, not useless** — `worker.js` still reaches #3. But
one vendored file (`public/vendor/three.min.js`) produced 1492 of its 2915 nodes:
51% of the graph was a bundled dependency nobody asked about.

Prefer scoping to deleting. Scoping is read-only and reversible:

```bash
graphify update src/            # good — indexes only source
graphify update .               # only if the repo has no vendored/minified assets
```

**Gotcha: a scoped build splits its output across two directories.** The real
artifacts (`graph.json`, `GRAPH_REPORT.md`, `graph.html`) go to
`src/graphify-out/`, but a stub `graphify-out/manifest.json` is also written in
the current directory — so `./graphify-out/` *looks* valid while containing no
graph. Every read command defaults to `graphify-out/graph.json` and will fail
with `error: graph file not found`. Pass `--graph` whenever you scoped:

```bash
graphify explain "createBooking()" --graph src/graphify-out/graph.json
```

**Always check what got indexed before trusting a report.** This ranks files by
how many nodes each contributed (verified 2026-07-27 against both a scoped and
an unscoped build):

```bash
python3 -c "
import json,collections,sys
g=json.load(open(sys.argv[1]))
c=collections.Counter(n.get('source_file','?') for n in g['nodes'])
[print(f'{v:5d}  {k}') for k,v in c.most_common(8)]" src/graphify-out/graph.json
```

If a vendored or minified file tops that list, rescope and rebuild.

## The runbook

**1. Build.** Local and LLM-free:

```bash
graphify update src/
```

Terminal output confirms the mode — expect a first line reading
`Re-extracting code files in src (no LLM needed)...` and a
`[graphify watch] Rebuilt: N nodes, M edges, K communities` summary. Re-running
on unchanged code prints `No code-graph topology changes detected; outputs left
untouched`, so it is cheap to repeat.

**How reproducible it is** (two independent builds of identical input,
2026-07-27): node and edge counts were identical both times (2915 / 5146), but
the community count differed (174 vs 175). **Treat nodes and edges as
reproducible facts and cluster/community groupings as advisory** — never cite a
community count as a stable property of a codebase.

**2. Orient — read the report, not the graph.**

```bash
sed -n '1,45p' src/graphify-out/GRAPH_REPORT.md
```

This file — not the terminal — carries the run's audit line, e.g.
`Token cost: 0 input · 0 output`. Check it to confirm no LLM backend engaged.
Two sections carry the value:

- **God Nodes (most connected)** — the repo's core abstractions, ranked by edge
  count. On the measured repo the top ranks were `route()`, `json()`,
  `generateChart()`, `readBody()`, `dispatch()`, `createBooking()` — the actual
  router, response helper, and engine entry point. This is the read-first list.
- **Community Hubs** — the label of each detected cluster ("community" =
  a group of densely connected nodes, found by clustering). Each label is
  whichever member is most central, so the list mixes file names with symbol
  names rather than being a pure module map.

**3. Expand a symbol.** Lists every caller (`<--`) and callee (`-->`) with
file:line and an `EXTRACTED` / `INFERRED` provenance tag:

```bash
graphify explain "createBooking()" --graph src/graphify-out/graph.json
```

**4. Trace a route between two symbols:**

```bash
graphify path "route()" "generateChart()" --graph src/graphify-out/graph.json
```

Returns the shortest hop chain, each edge labeled with its relation (`contains`,
`imports`, `imports_from`, `calls`, among others). It warns when a name match
was ambiguous — when it does, re-run with a more exact symbol name.

**5. Verify before acting.** Open the file at the line the graph cited. On the
measured repo a 6-line spot check (L240/241/242/266/288/289) matched the source
exactly, but accuracy on one repo is not a guarantee on yours, and a graph is
evidence *about structure only*. `bbb-verification-and-evidence` sets the bar
for what you may then claim.

## Trust boundaries — what this tool does not do

- **`query` matches symbol names, not natural language.** A prose question whose
  words are not identifiers returns `No matching nodes found` — verified
  2026-07-27, e.g. `graphify query "how is overselling prevented?"` finds
  nothing. An identifier-shaped query works: `graphify query "slots"` seeds from
  `getSlots()`, `adminListSlots()`, `adminGenerateSlots()` and returns a BFS
  neighborhood (54–57 nodes depending on how the graph was scoped — node counts
  are per-build, not per-codebase). Even then the result is **a map, not an
  answer**: it is a
  list of symbols to go read, truncated to a token budget (`--budget N` raises
  the cap), and it may include unrelated files that merely share a name shape.
- **Only `update`, `explain`, and `path` are local.** `extract`, `label`, and
  `cluster-only --backend` send code-derived content to an external LLM provider
  (gemini/kimi/claude/openai/deepseek/ollama). Do not run them on a confidential
  repo. The `Token cost` line in GRAPH_REPORT.md is how you confirm which ran.
- **Edges are syntactic.** Tree-sitter sees call *sites*. Dynamic dispatch,
  string-keyed routing, DI containers, and reflection are invisible — the graph
  under-reports real coupling. An absent edge is not proof of no relationship.
- **`INFERRED` edges are guesses.** The report prints an EXTRACTED/INFERRED split
  with an average confidence (2–5% INFERRED at ~0.5 confidence across the three
  measured builds, 2026-07-27). Cite `EXTRACTED` edges; treat `INFERRED` as a
  hint to check.
- **Not all languages are equal.** `.sql` files contributed zero nodes and
  emitted: `tree_sitter_sql not installed. Install it with: pip install
  "graphifyy[sql]"`. *Unverified — that install was not executed.* Any repo
  whose logic lives in SQL, templates, or config will be under-represented.
- **Pre-1.0 dependency (0.9.27 as of 2026-07-27).** Flags and output format may
  change without notice. Never make a runbook depend on it; re-verify commands
  when the report format looks unfamiliar.

## Fallback (never block on this tool)

If `graphify --version` fails, the build errors, or the report is dominated by
noise you cannot scope away, stop and run the manual sequence in
`bbb-project-discovery`. Record the fallback and the reason in your run notes.
A session that stalls waiting on an optional accelerator has failed at
minimum-scope discipline (`b5-mode`).

## Provenance and maintenance

- Authored 2026-07-27 for the Blue-Bubble-Buddy skill library (Primitives
  pillar), on owner sign-off for a Class 3 change (`bbb-change-control`), with
  the explicit scope decision: optional accelerator, fallback mandatory.
  Reviewed the same day by independent FACTUAL, DOCTRINE, and USABILITY
  reviewers; their blocking and important findings are applied here.
- All commands above were executed on Linux 2026-07-27 against a working copy of
  a real repo (139 tracked files), with graphify 0.9.27 installed via
  `uv tool install graphifyy`. Node/community figures, hub lists, and God Node
  names are that repo's measured output on that date — they illustrate the
  scoping effect and will differ on any other repo.
- Explicitly unverified: the `graphifyy[sql]` extra; build timings are not
  reproduced here; `watch`, `merge-graphs`, `clone`, and the Obsidian/wiki
  exports were not executed and are deliberately omitted rather than described
  secondhand.
- Volatile facts: graphify is pre-1.0, so the version, the CLI flags
  (`--graph`, `--budget`, `--strict`), the split `graphify-out/` layout, and the
  GRAPH_REPORT.md section headings are all drift-prone. Re-verify the surface:
  `graphify --version && graphify --help | head -20`
- Re-verify the core loop end-to-end in any repo with source in `src/`:
  `graphify update src/ && sed -n '1,20p' src/graphify-out/GRAPH_REPORT.md`
- Re-verify the query-matching rule (expect no matches, then a seeded traversal):
  `graphify query "how is overselling prevented?" --graph src/graphify-out/graph.json`
- Sibling names cited here — re-verify with `ls .claude/skills/` at the library
  root.
