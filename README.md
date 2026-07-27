# Blue-Bubble-Buddy

A portable skill library that equips junior/mid-level engineers and smaller AI
models (Sonnet-class) to carry any project forward with the discipline of a
principal engineer. Seventeen skills teach four pillars: **Primitives** (how to
communicate, act, verify, stop), **Orchestration** (goal → action → check →
repeat; delegation), **Memory** (state, handoffs, failure chronicles), and
**Self-Improvement** (rule distillation, skill authoring, change control,
research method).

Each skill is a self-contained runbook at `.claude/skills/<name>/SKILL.md`,
written for a zero-context reader: imperative voice, verified copy-pasteable
commands, when-NOT-to-use boundaries, and a provenance section with
re-verification one-liners.

## The skills

### Primitives — the behavior core
| Skill | One line |
|---|---|
| `b5-mode` | The behavior contract: conclusion-first replies, immediate action, evidence-backed progress reports, minimum scope, disciplined turn endings. |
| `bbb-project-discovery` | Investigate an unfamiliar repo like an incoming principal engineer before changing anything; Project Brief template; ask the human at most five questions. |
| `bbb-knowledge-graph` | Build a call-graph of an unfamiliar repo with the third-party graphify CLI to find entry points and trace call paths; optional accelerator for one step of project discovery, with a mandatory fallback. |
| `bbb-debugging-playbook` | Reproduce first, rank mechanisms, run discriminating experiments one change at a time; a fix must explain every observation including the negatives. |

### Orchestration — goals, checks, delegation
| Skill | One line |
|---|---|
| `bbb-eval-loop` | The Goal → Action → Check → Repeat loop: define the check before acting, gate every iteration, know when to stop polishing or escalate. |
| `bbb-verification-and-evidence` | What counts as evidence, per-change-type verification recipes, and how to report verified vs unverified work. |
| `bbb-visual-self-check` | Verify visual output by rendering and screenshotting (verified Playwright script included), never by assuming code looks right. |
| `bbb-subagent-orchestration` | When to delegate to subagents vs work inline; prompt contracts, shared briefs, parallel review workflows, graceful degradation. |

### Memory — surviving across sessions
| Skill | One line |
|---|---|
| `bbb-state-and-memory` | STATE.md discipline: read before start, update after every run; handoff notes a zero-context successor can resume from. |
| `bbb-failure-archaeology` | The append-only chronicle (symptom → root cause → evidence → status) so no settled battle is refought; git-history mining commands. |

### Self-Improvement — how the library learns
| Skill | One line |
|---|---|
| `bbb-rule-distillation` | Turn repeated corrections and incidents into durable rules; placement, promotion into skills, and retirement of stale rules. |
| `bbb-skill-authoring` | The canonical skill format, trigger-rich description writing, house template, and pre-submission checklist. |
| `bbb-change-control` | Class 1–4 change gating for this library, the three-reviewer workflow, and the seven non-negotiables with their rationale. |
| `bbb-install-and-use` | Installing skills per surface (project, user, claude.ai `.skill` packaging) and troubleshooting skills that won't load. |

### Advanced — hard problems and research
| Skill | One line |
|---|---|
| `bbb-campaign-planning` | Decision-gated campaigns for multi-session problems: phases with pre-committed expected observations, branches, fenced wrong paths. |
| `bbb-research-methodology` | Hypothesis-predicts-numbers-before-running, adversarial refutation, and the idea lifecycle ending in adoption or documented retirement. |
| `bbb-research-frontier` | The open problems (eval harness, cross-model parity, automated distillation…) with first steps and falsifiable milestones — all labeled open/candidate. |

## Quick start

Copy the starter set into any repo (full options: `bbb-install-and-use`):

```bash
mkdir -p "$TARGET/.claude/skills"
cp -r .claude/skills/b5-mode .claude/skills/bbb-eval-loop \
      .claude/skills/bbb-verification-and-evidence .claude/skills/bbb-state-and-memory \
      "$TARGET/.claude/skills/"
```

Add the rest when a real need appears — first hard bug → `bbb-debugging-playbook`,
first UI work → `bbb-visual-self-check`, and so on.

## Working on this library

1. Read `STATE.md` first; update it before you stop (`bbb-state-and-memory`).
2. Any change to `.claude/skills/` is classified and gated — see
   `bbb-change-control`. New skills follow `bbb-skill-authoring`.
3. Ground truth only: never publish a command you did not run.

License: MIT.
