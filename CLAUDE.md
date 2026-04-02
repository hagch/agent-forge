# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

agentforge is a Claude Code plugin that orchestrates 15 specialized agents through an 8-phase skill-chained feature development pipeline. It's a self-improving multi-agent framework that brings compound AI system patterns to software development.

**Not a compiled application** — this is a plugin distribution. There are no build, test, or lint commands for the framework itself.

## Installation & Validation

```bash
# Validate plugin structure
claude plugin validate /path/to/agentforge

# Install from marketplace
claude plugin marketplace add hagch/agentforge
claude plugin install agentforge

# Run with required flags
claude --permission-mode auto --chrome
```

Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in `~/.claude/settings.json` env.

## Architecture

### 8-Phase Skill Chain

```
/brainstorm → /challenge-concept → /plan-feature → /challenge-plan → /develop-slices → /verify-feature → /finish-branch → /self-improve
```

Each phase is a **self-contained skill** that explicitly invokes the next. The Orchestrator manages state and human checkpoints.

| Phase | Skill | Agents Involved | Purpose |
|-------|-------|----------------|---------|
| 1. Brainstorm | `/brainstorm` | Researcher + UX-Researcher (parallel) | Socratic dialogue, one question at a time → bullet-proof Design-Spec |
| 2. Challenge Concept | `/challenge-concept` | 4× Challenger (parallel) | Security, Performance, DX, DAU stress test on spec |
| 3. Plan Feature | `/plan-feature` | Planner (opus) | ShapeUp vertical slices + micro-tasks + pseudocode → Task-DAG |
| 3.5 Challenge Plan | `/challenge-plan` | 4× Challenger (parallel) | Architecture, Completeness, Maintainability, Risk against codebase |
| 4. Development | `/develop-slices` | DB + Backend + Frontend + Test + DevOps per slice | TDD + conventional commits + 3-stage review per slice |
| 5. Verification | `/verify-feature` | 6× Verifier + DAU-Tester (parallel) | Cross-slice integration, security, performance, DAU journey, compound gate |
| 6. Finish Branch | `/finish-branch` | — | PR/Merge/Keep/Discard |
| 7. Self-Improve | `/self-improve` | Researcher | Retro + online research + health check |

### Key Directories

- **`skills/`** — Runtime command skills (`/setup`, `/self-improve`) with SKILL.md definitions
- **`templates/agents/`** — 15 agent prompt templates + `base-instructions.md` shared by all agents
- **`templates/skills/`** — 15 skill templates copied to target projects during `/setup`
- **`templates/docs/`** — Docsify root templates (index.html with plugins, README.md landing page)
- **`templates/docs/agentforge/`** — Top-down doc templates (README, overview, workflows) + registry files (FEATURES, DECISIONS, SITEMAP, feature-template)
- **`.claude-plugin/`** — Plugin metadata (`plugin.json`, `marketplace.json`)

### Documentation Structure (in target projects)

All agentforge-managed docs live under `docs/agentforge/` — the project's `docs/` root stays clean for other documentation. Content follows a top-down learning arc inspired by codebase-to-course:

1. `docs/agentforge/README.md` — What does the project do? (user-visible behavior)
2. `docs/agentforge/overview.md` — Architecture with Mermaid diagrams
3. `docs/agentforge/workflows.md` — End-to-end data flows with sequence diagrams
4. `docs/agentforge/FEATURES.md` → `DECISIONS.md` → `SITEMAP.md` — Registry files
5. `docs/agentforge/features/<domain>/<slug>/` — Individual feature docs + CURRENT-STATE.md

Docsify index.html includes plugins: Mermaid, flexible-alerts, tabs, pagination, sidebar-collapse, copy-code.

**Docsify Link Rule:** ALL markdown links in docs are resolved relative to the `docs/` root, NOT relative to the current file. Files in subdirectories (e.g., `docs/agentforge/FEATURES.md`) must use full paths from docs root (`agentforge/features/...`), not relative paths (`features/...`).

**Sidebar Generation:** `/setup` creates `scripts/generate-sidebar.sh` which scans `docs/` recursively and regenerates `docs/_sidebar.md`. A `PostToolUse` hook on `Write(docs/**)` runs this script automatically after every doc change.

### Ralph Pattern (State Management)

State lives in the filesystem, not LLM context:
- **CURRENT-STATE.md** at `docs/agentforge/features/<domain>/<slug>/CURRENT-STATE.md` — single source of truth per feature
- Feature docs follow the pipeline: Concept → Challenge → Plan → Implementation → Verification → Retro
- On session restart: agents read CURRENT-STATE.md and resume from last checkpoint

### Agent Coordination

- All agents inherit `base-instructions.md` (verification rules, problem docs, guardrails, code standards)
- Dev agents respect file ownership boundaries (configured during `/setup`)
- Task dependencies use TaskCreate + addBlockedBy for DAG execution
- Previous failure reasons stored in feature docs, read by agents on retry

## Runtime Commands

- **`/setup`** — Analyzes target repo, extracts conventions, copies templates, configures agents
- **`/orchestrate`** — Interactive feature pipeline: invokes the 8-phase skill chain with human checkpoints. Each phase is a dedicated skill that invokes the next.
- **`/self-improve`** — Compares with Superpowers plugin, researches community patterns, runs health check

## Auto-Apply Policy

Only documentation changes are auto-applied (feature docs, FEATURES.md, SITEMAP.md, DECISIONS.md). All changes to agents, skills, CLAUDE.md, base instructions, and hooks **require explicit user approval** before being applied. This policy is enforced in the orchestrate skill, orchestrator agent (Phase 6), and self-improve skill.

## Design Influences

The framework synthesizes patterns from: Berkeley BAIR (compound AI systems), DSPy (optimizable pipelines), Jesse Vincent/Superpowers (TDD + verification), Addy Osmani (3-4 builders per 1 reviewer), Geoffrey Huntley/Ralph (external state + fresh context), and Yohei Nakajima (self-improving agents).
