# AgentForge

Self-improving multi-agent development framework for Claude Code. Coordinates 7 specialized agents through 6 development phases with compound verification, session continuity, and continuous self-improvement.

## What It Does

AgentForge structures your development workflow into **7 agents** working across **6 phases**:

### Agents

| Agent | Model | Role |
|-------|-------|------|
| Orchestrator | Opus | Session management, phase coordination, agent dispatch |
| Researcher | Opus | Codebase analysis, architecture understanding, context gathering |
| Challenger | Opus | Assumption testing, edge case discovery, design review |
| Planner | Opus | Task breakdown, implementation plans, dependency analysis |
| Dev-Backend | Sonnet | Backend implementation, TDD, API development |
| Dev-Frontend | Sonnet | Frontend implementation, TDD, UI components |
| Verifier | Sonnet | Test execution, compound verification, quality gates |

### Phases

1. **Research** - Understand the codebase, gather context, analyze requirements
2. **Challenge** - Question assumptions, identify risks, explore alternatives
3. **Plan** - Break down into tasks, define implementation order, set acceptance criteria
4. **Implement** - TDD cycles (Red/Green/Refactor), backend and frontend in parallel
5. **Verify** - Compound verification (unit, integration, E2E), quality gates
6. **Document** - Feature docs, CURRENT-STATE.md update, session handoff

## Installation

```bash
claude plugin add agentforge
```

## Setup

Run the setup command after installation to analyze your repository and extract coding guidelines:

```bash
/setup
```

This scans your project structure, detects conventions, and configures AgentForge for your codebase.

## Commands

| Command | Description |
|---------|-------------|
| `/setup` | Analyze repo structure, extract coding guidelines, configure agents |
| `/self-improve` | Compare with Superpowers diff, research online improvements, scan community patterns |

## Key Features

- **Session Continuity** - CURRENT-STATE.md tracks progress across sessions so work is never lost
- **Ralph Pattern** - Challenger agent questions plans before implementation to catch issues early
- **Self-Improvement** - `/self-improve` evolves the framework by researching new patterns and community practices
- **Plugin + Local Hybrid** - Works as a Claude Code plugin with local project overrides
- **Docsify Integration** - Auto-generates searchable documentation site for your project
- **Compound Verification** - Multi-layer testing (unit + integration + E2E) before any phase completes

## Prerequisites

- **Claude Code** with Agent Teams support
- **Environment variable:** `CLAUDE_AGENT_TEAMS=1` must be set
- **Permission mode:** Set to allow agent spawning (e.g., `--dangerously-skip-permissions` for local dev or configure appropriately)
- **Chrome/Chromium** - Required for `/self-improve` online research capability
