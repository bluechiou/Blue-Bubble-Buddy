---
name: bbb-knowledge-graph
description: Build a deterministic call-graph of an unfamiliar codebase with the third-party graphify CLI, then use it to find entry points and trace call paths in seconds instead of reading files blind. Load this skill when onboarding into a large or unfamiliar repo and the file tree is too big to read; when the user asks "where do I start", "what are the core abstractions", "what calls X", "how does X reach Y", or "map this codebase"; or when a discovery pass is stalling on which files matter. Provides the verified install, the scoping rule that decides whether output is signal or noise, the explain/path runbook, and the honest limits of its query command. Optional accelerator only — the manual survey in bbb-project-discovery is the fallback and stays authoritative. Not for reasoning about a bug (bbb-debugging-playbook) and not for cross-session state (bbb-state-and-memory).
---

# Knowledge Graph — accelerated codebase orientation

## Purpose

A large unfamiliar repo costs a session hundreds of thousands of tokens of
blind file reading before it knows which files matter. `graphify` parses the
repo into a call-graph with tree-sitter (a deterministic parser — no model, no
API call, no tokens) and answers three structural questions directly: what are
the most connected abstractions, what does this symbol call, and how does A
reach B. Use it to decide *what to read*; keep reading the source to decide
*what is true*.

This skill is an **optional accelerator**. Everything here is skippable: if
`graphify` is not installed, or its output looks like noise, fall back to the
manual investigation sequence in `bbb-project-discovery`, which remains the
authoritative discovery runbook.

## When NOT to use this skill

- **Diagnosing a bug or regression** — a graph shows structure, never causality.
  Use `bbb-debugging-playbook`.
- **A repo you already know, or a one-file fix** — building a graph to change
  one function is pure overhead. Read the file and its callers.
- **Carrying state between sessions** — the graph maps *code*, not *work in
  progress*. `bbb-state-and-memory` owns STATE.md; a graph never replaces it.
- **The full discovery pass** — manifest, CI, git scars, and TODO hotspots are
  owned by `bbb-project-discovery`. This skill only accelerates its "which
  files matter" step.
- **Anything requiring installs you cannot make** — no network or no `uv`
  means fall back; do not block a task on this tool.

## Install (verified 2026-07-27)

```bash
uv tool install graphifyy        # note the trailing "yy" — package name differs from the command
graphify --version               # expect: graphify 0.9.27
```

`graphify install` additionally copies a graphify skill into your assistant's
config directory and writes a `CLAUDE.md` section. That is optional and
independent of this skill — the CLI alone provides everything below.

## The scoping rule — the single thing that decides signal vs noise

**Point graphify at source directories, never at a repo root that contains
vendored or minified files.** Minified bundles explode into thousands of
one-letter nodes that dominate the report and bury real code.

Measured on one repo (139 tracked files, Cloudflare Workers + JS, 2026-07-27):

| Build target | Nodes | Communities | Top navigation hubs |
|---|---|---|---|
| Repo root (contains `three.min.js`, vendored skill scripts) | 2915 | 174 | `vt`, `sn`, `At`, `St`, `jn` — minified junk |
| Root minus vendor/minified/lockfiles | 327 | 19 | `worker.js`, `chart-api.js`, `flow.js` |
| `src/` only | 229 | 14 | `flow.js`, `chart-api.js`, `worker.js`, `createBooking` |

Prefer scoping to deleting. Scoping is read-only and reversible:

```bash
graphify update src/            # good — indexes only source
graphify update .               # only if the repo has no vendored/minified assets
```

**Gotcha: output follows the path argument.** `graphify update src/` writes to
`src/graphify-out/`, not to the current directory. Every read command defaults
to `graphify-out/graph.json`, so pass `--graph` when you scoped:

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

If a vendored or minified file tops that list, rescope and rebuild. On the
unscoped build above, `public/vendor/three.min.js` alone produced 1492 of 2915
nodes — 51% of the entire graph was one bundled dependency.

## The runbook

**1. Build.** Deterministic, no LLM, no tokens. Took 8.1s for 2915 nodes and
under 2s for a scoped 229-node build (2026-07-27):

```bash
graphify update src/
```

Confirm the summary line reports `Token cost: 0 input · 0 output` — if it does
not, an LLM backend engaged and the run is no longer deterministic.

**2. Orient — read the report, not the graph.**

```bash
sed -n '1,45p' src/graphify-out/GRAPH_REPORT.md
```

Two sections carry the value:

- **God Nodes (most connected)** — the repo's core abstractions, ranked by
  edge count. On the measured repo these were `route()`, `json()`,
  `generateChart()`, `readBody()`, `dispatch()`, `createBooking()` — the actual
  router, response helper, and engine entry point. This is the read-first list.
- **Community Hubs** — file-level clusters, i.e. the module map.

**3. Expand a symbol.** Lists every caller (`<--`) and callee (`-->`) with
file:line and an `EXTRACTED` / `INFERRED` provenance tag:

```bash
graphify explain "createBooking()" --graph src/graphify-out/graph.json
```

**4. Trace a route between two symbols:**

```bash
graphify path "route()" "generateChart()" --graph src/graphify-out/graph.json
```

Returns the shortest hop chain with the relation on each edge
(`contains`, `imports_from`, `calls`). It warns when a name match was
ambiguous — when it does, re-run with a more exact symbol name.

**5. Verify before acting.** Open the file at the line the graph cited. On the
measured repo a 6-line spot check (L240/241/242/266/288/289) matched the source
exactly, but accuracy on one repo is not a guarantee on yours, and a graph is
evidence *about structure only*. `bbb-verification-and-evidence` sets the bar
for what you may then claim.

## Trust boundaries — what this tool does not do

- **`query` returns a map, not an answer.** `graphify query "how is overselling
  prevented?"` returned a truncated 85-node neighborhood (with unrelated files
  mixed in), not a mechanism. It is a fuzzy entry-point finder: useful for "which
  symbols relate to this topic", useless as an explanation. It truncates to a
  token budget and says so; `--budget N` raises the cap. Treat every result as a
  list of files to open.
- **Edges are syntactic.** Tree-sitter sees call *sites*. Dynamic dispatch,
  string-keyed routing, DI containers, and reflection are invisible — the graph
  under-reports real coupling. An absent edge is not proof of no relationship.
- **`INFERRED` edges are guesses.** The report prints an EXTRACTED/INFERRED
  split with an average confidence (4% INFERRED at ~0.5 confidence on the
  measured build). Cite `EXTRACTED` edges; treat `INFERRED` as a hint to check.
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
  pillar), on owner sign-off for a Class 3 change (`bbb-change-control`),
  with the explicit scope decision: optional accelerator, fallback mandatory.
- All commands above were executed on Linux 2026-07-27 against a working copy
  of a real repo (139 tracked files), with graphify 0.9.27 installed via
  `uv tool install graphifyy`. Node/community/timing figures and the God Node
  and hub names are that repo's measured output on that date — they illustrate
  the scoping effect and will differ on any other repo.
- Explicitly unverified: the `graphifyy[sql]` extra; `watch`, `merge-graphs`,
  `clone`, and the Obsidian/wiki exports were not executed and are deliberately
  omitted from the runbook rather than described secondhand.
- Volatile facts: graphify is pre-1.0, so the version, the CLI flags
  (`--graph`, `--budget`), the `graphify-out/` layout, and the GRAPH_REPORT.md
  section headings are all drift-prone. Re-verify the whole surface with:
  `graphify --version && graphify --help | head -20`
- Re-verify the core loop end-to-end in any repo with source in `src/`:
  `graphify update src/ && sed -n '1,20p' src/graphify-out/GRAPH_REPORT.md`
- Sibling names cited here — re-verify with `ls .claude/skills/` at the library
  root.
