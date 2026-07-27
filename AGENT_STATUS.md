# AGENT_STATUS.md — ÆTHNOUS Project Network

**Compiled:** 2026-07-23 · **Compiled by:** Claude (session `project-status-compilation`) · **For:** any AI agent (Claude, Gemini, ChatGPT, or other) picking up work in this repo or a sibling repo

This file is a handoff briefing about the wider project network, distinct from this repo's own `STATE.md` (which is this repo's *canonical* per-session state doc — read that one first, per this repo's own `CLAUDE.md`). This file exists so an agent landing in *any* of Blue's repos, including a sibling repo, understands where this one — and the whole network — currently stands. Update it whenever this repo's status changes materially.

## 1. Who you're working for

**Blue Chiou** (bluechiou@gmail.com, commits as "Blue.X") is the founder and sole owner-operator of **ÆTHNOUS**, a Zi Wei Dou Shu (紫微斗數 / Purple Star Astrology) chart-reading brand that blends the 占驗派 tradition with Jungian depth psychology, plus a small portfolio of side products, of which this skill library is one (built to be reusable by others, not ÆTHNOUS-specific). Blue builds everything through AI-assisted development (primarily Claude Code) and does not have a formal software-engineering background — explain tradeoffs plainly, don't assume prior engineering context, and default to asking rather than guessing on anything ambiguous or high-stakes.

Blue runs work through named AI agent personas, each with its own skill file:
- **Raziel (密典)** — Chief Technical Executor: engineering, deploys, API/security architecture.
- **Cassian (紫曜)** — Head Analyst: ZWDS chart reading, synastry, flow-year/decade prediction. Runs the 汎天派 (Fan Tian Pai) school, v3-Ultra form.
- Other named collaborators referenced elsewhere in the network: **Gabriel**, **Uriel**, **Vergil**, **Thoth**, **Raphael**.

## 2. The ÆTHNOUS project network (7 repos)

| Repo | What it is | Production | Snapshot |
|---|---|---|---|
| **Blue_Astral_Nexus_Engine** | Core ZWDS calculation engine + API (`chart-api.js`), extends `iztro` with Blue's corrected 四化/亮度/宮名 rules | engine.aethnous.co (API), chart.aethnous.co (UI) | Active — formation catalog + monetization infra + EN-market prep |
| **Blue_ANE_Owner_Ext** | Private owner-only extension: proprietary reading-lens + Tier-2 judgment-rule bundle, pulled into the Engine at build time | (bundled into engine.aethnous.co for Blue only) | Active — category/filter UI hardened, Tier-2 daily-fortune generator mid-calibration |
| **Blue-Booking** | Booking + member portal + CRM for the consultation service (Cloudflare Workers + D1 + R2), vendors a copy of the Engine | booking.bluechiou.com | Active — just shipped a large `/gate` 3D landing experience |
| **Blue-OS** | Claude Code dashboard/control panel, runs on Blue's Mac, phone-reachable PWA | localhost:4173 (Blue's Mac) | Stable/idle — booking subsystem just split out to its own repo |
| **aethnous-landing** | Public ÆTHNOUS marketing/landing site, Next.js (customized build) + Three.js solar-system hero | (Vercel, ÆTHNOUS domain) | Active — building EN-market acquisition funnel (`/start`, `/quiz`) |
| **stardust** | 星塵夢汐 Stardust DreamTide — separate wellness/journaling PWA (mood tracking, AI companion, crystal encyclopedia) | stardust.bluechiou.com | Active, brand-new — v1 just shipped, v1.5+ roadmapped |
| **Blue-Bubble-Buddy** (this repo) | Portable Claude Code skill library (17 skills) teaching engineering discipline to other AI agents/smaller models | (skill library, no deployment) | Active — see §4; STATE.md current as of 2026-07-27 |

Unlike the other six repos, this one isn't an ÆTHNOUS product — it's a general-purpose engineering-discipline skill library Blue built, usable in any project (including the other six).

## 3. Rules that hold across the network

- **Personal data never enters git** anywhere in the network — no real birth data (Engine/Booking/Owner-Ext), no real wellness data (stardust).
- **Secrets never hardcoded** anywhere in the network.
- **Blue's Version is the single authority for ZWDS logic** in the astrology repos — not applicable to this repo's own content.
- **IP boundaries are release blockers.** Agent skill files (`*_SKILL.md`, e.g. `RAZIEL_SKILL.md`) are Blue's personal IP with a canonical home in Google Drive, not git, in the *other* repos. This repo's own skills are the product and are meant to be public/reusable — different posture, don't confuse the two.
- **Surgical changes only, everywhere.** Every task should have a stated, verifiable success criterion before coding starts.
- **Self-scheduled check-ins may only be armed when they'll deliver genuinely new value** (stated explicitly in the Engine repo's CLAUDE.md as applying to every session).
- **This repo's own, stricter rules** (from its `CLAUDE.md`, which take precedence here): read `STATE.md` before doing anything; every change to `.claude/skills/` goes through Class 1–4 change control (see the `bbb-change-control` skill); ground truth only — never publish an unverified command or path; one home per fact — check the README's skill inventory before adding duplicate content; anything that took real time to figure out gets an `ARCHAEOLOGY.md` entry.

## 4. This repo: Blue-Bubble-Buddy

### Purpose
A portable Claude Code skill library — the product *is* the `.claude/skills/` tree (17 BBB-authored skills, plus one vendored third-party skill so the directory count is 18), teaching junior/mid-level engineers and smaller (Sonnet-class) models to carry a project forward with principal-engineer discipline across four pillars: Primitives (communicate/act/verify/stop), Orchestration (goal→action→check→repeat, delegation), Memory (state, handoffs, failure chronicles), and Self-Improvement (rule distillation, skill authoring, change control, research method). No `package.json` — pure Markdown/skill library, not an npm package.

### Current status: active — one unmerged branch still needs a decision
`STATE.md` on `main` is current (2026-07-27). The open item below is not a
staleness problem any more; it is a branch that was never merged:

- **`main`'s `STATE.md`** (now dated "verified 2026-07-27") still lists the FACTUAL review pass over the original 16 skills as open, because the work that closed it has never landed here.
- **An unmerged branch, `claude/file-reading-setup-dtg41w`** (single commit `c88323e`, dated 2026-07-17 — ten days newer), contains an updated `STATE.md` claiming that debt is **paid**: every command in all 16 skills was re-executed, and one real Class 2 defect was found and fixed (`bbb-rule-distillation`'s audit regex widened from bullet-only `^- ` to `^(- |[0-9]+\. )` so it also catches numbered-list rules in CLAUDE.md files). That branch's `STATE.md` also names the next roadmap item: **agent-skill ↔ BBB integration** (e.g., per-agent skills referencing `b5-mode`).
- **This work has never been merged to `main`.** As of 2026-07-27 `main`'s `STATE.md` open-items table names the branch explicitly and says not to redo the pass blindly, so the picture is no longer misleading — but the branch is still unmerged and the fix it carries is still not in `main`.

**Recommended next action for whoever picks this up:** review and merge `claude/file-reading-setup-dtg41w` into `main` (or cherry-pick its `STATE.md` + the `bbb-rule-distillation` fix), so the canonical `STATE.md` reflects reality before anyone wastes time re-doing the already-finished factual review pass.

### Other open items
- **No `ARCHAEOLOGY.md` exists yet**, despite two skills (`bbb-failure-archaeology`, `bbb-project-discovery`) describing it as a standing convention — it hasn't actually been created in this repo yet.
- Five skills sit under the library's own 120-line target — accepted as-is per `main`'s STATE.md, not blocking.
- Next planned feature-level work (per the unmerged branch): tighter integration between BBB skills and other per-agent skill sets.

### Skill inventory
17 BBB skills under `.claude/skills/`: `b5-mode`, `bbb-campaign-planning`, `bbb-change-control`, `bbb-debugging-playbook`, `bbb-eval-loop`, `bbb-failure-archaeology`, `bbb-install-and-use`, `bbb-knowledge-graph`, `bbb-project-discovery`, `bbb-research-frontier`, `bbb-research-methodology`, `bbb-rule-distillation`, `bbb-skill-authoring`, `bbb-state-and-memory`, `bbb-subagent-orchestration`, `bbb-verification-and-evidence`, `bbb-visual-self-check`. A vendored third-party skill, `ui-ux-pro-max` (PR #8, 2026-07-27), also sits in the tree but is not authored to this library's format spec — so `ls` returns 18 while the library is 17. Most recently touched (library-wide readability pass, latest merged commit): 9 of the 16, for heading/section-order normalization only — no command-text changes.

### Branches
`main` plus this session's branch are current and in sync. `claude/docs-ai-agent-readability-3p8jyz` and `claude/workflow-improvement-review-sxjfwd` are already merged. **`claude/file-reading-setup-dtg41w` is the one unmerged branch with real, stranded work** — see above.

---

## 5. If you're an agent picking this up cold

1. Read `STATE.md` first, per this repo's own `CLAUDE.md` — but know it may be stale (see §4 above) before trusting it fully.
2. Check section 4 above, and resolve the `STATE.md` discrepancy before starting new work if you're able to.
3. Any change to `.claude/skills/` goes through this repo's own change-control process (`bbb-change-control` skill) — this file doesn't override that.
4. If you materially change this repo's status, update both this file's section 4 *and* `STATE.md` (per `bbb-state-and-memory`) before you stop — don't let the two drift apart again.
