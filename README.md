# agentforge

A self-improving multi-agent development framework for Claude Code.

## What it does

agentforge orchestrates 7 specialized agents through a 6-phase pipeline:

1. **Brainstorm** — Online research + Socratic dialogue → Concept Doc
2. **Challenge** — Security, Performance, DX, DAU stress test
3. **Planning** — Autonomous task-DAG creation with dependencies
4. **Development** — Parallel TDD via Agent Teams (backend + frontend)
5. **Verification** — Code review, E2E tests, DAU-UI test, compound gate
6. **Self-Improve** — Retro, documentation, agent/skill improvements, community scan

## Installation

Two steps: add the marketplace, then install the plugin.

```bash
# Step 1: Add the agentforge marketplace
claude plugin marketplace add hagch/agentforge

# Step 2: Install the plugin
claude plugin install agentforge
```

For local development/testing:

```bash
# Validate the plugin structure
claude plugin validate /path/to/agentforge

# Or run Claude with the plugin loaded directly
claude --plugin-dir /path/to/agentforge
```

## Setup

After installing, run in your project:

```
/setup
```

This analyzes your repository, extracts coding conventions, generates agents and skills configured for your project, and sets up Docsify documentation.

## Commands

| Command | Purpose |
|---------|---------|
| `/setup` | Initialize framework in a project (or update existing) |
| `/self-improve` | Manual improvement: Superpowers diff, online research, community scan, health check |

## Agents

| Agent | Model | Role |
|-------|-------|------|
| orchestrator | opus | Pipeline control, state management, retro |
| researcher | sonnet | Online research, community scan |
| challenger | sonnet | 4-perspective stress test (Security, Performance, DX, DAU) |
| planner | opus | Autonomous task-DAG creation |
| dev-backend | sonnet | Backend TDD implementation |
| dev-frontend | sonnet | Frontend TDD implementation |
| verifier | opus | Code review, E2E, DAU-UI test, compound gate |

## Key Features

- **Session Continuity** — CURRENT-STATE.md tracks exact progress, resume after restart
- **Ralph Pattern** — Fresh context per task, state in filesystem not LLM context
- **Self-Improvement** — Every feature cycle improves agents, skills, and documentation
- **Plugin + Local Hybrid** — Plugin installs templates, local copies evolve via self-improvement
- **Docsify Integration** — All docs browsable as a website with search
- **Compound Verification** — Multi-pass review + E2E + DAU-UI test before shipping

## Prerequisites

Enable Agent Teams (experimental):

```json
// ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Start with:

```bash
claude --permission-mode auto --chrome
```

| Prerequisite | Why |
|-------------|-----|
| Agent Teams env var | Parallel dev-backend + dev-frontend with shared task list |
| `--permission-mode auto` | Agents don't block on permission prompts |
| `--chrome` (optional) | DAU-UI visual testing in verification phase |

## Inspired By & Research

agentforge draws from these ideas, frameworks, and patterns:

### Compound AI Systems
- **Berkeley BAIR** — [The Shift from Models to Compound AI Systems](https://bair.berkeley.edu/blog/2024/02/18/compound-ai-systems/) — Multiple interacting components outperform single monolithic models
- **DSPy (Stanford)** — [dspy.ai](https://dspy.ai/) — Programming, not prompting, language models; treating multi-agent pipelines as optimizable programs

### Superpowers Framework
- **Jesse Vincent (obra)** — [Superpowers for Claude Code](https://github.com/obra/superpowers) — The foundational skills library for Claude Code: TDD enforcement, systematic debugging, verification-before-completion, plan-driven development. agentforge adapts and extends several Superpowers skills (tdd-cycle, systematic-debugging, plan-creation, verification-review, finishing-branch, writing-skills).
- [Blog: Superpowers](https://blog.fsck.com/2025/10/09/superpowers/)

### Multi-Agent Orchestration
- **Addy Osmani** — [The Code Agent Orchestra](https://addyosmani.com/blog/code-agent-orchestra/) — "The bottleneck is no longer generation. It's verification." 3-4 builders per 1 reviewer, file ownership discipline, token budgeting
- **Google ADK** — [Multi-Agent Patterns](https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/) — Sequential, parallel, loop, and DAG orchestration patterns
- **Claude Code Agent Teams** — [Official Docs](https://code.claude.com/docs/en/agent-teams) — Shared task lists, peer-to-peer messaging, file locking

### Ralph Loop Pattern
- **Geoffrey Huntley / snarktank** — [ralph on GitHub](https://github.com/snarktank/ralph) — External shell loop: state in filesystem, fresh context per iteration, confusion accumulation prevention
- [From ReAct to Ralph Loop](https://www.alibabacloud.com/blog/from-react-to-ralph-loop-a-continuous-iteration-paradigm-for-ai-agents_602799) — Continuous iteration paradigm for AI agents

### Self-Improving Agents
- **Yohei Nakajima** — [Better Ways to Build Self-Improving AI Agents](https://yoheinakajima.com/better-ways-to-build-self-improving-ai-agents/) — Reflection loops, persistent skill representations, self-generated in-context examples
- **Shinn et al.** — [Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366) — Verbal self-reflection without weight updates, Actor-Evaluator-Reflection architecture

### Additional Influences
- **CrewAI** — Role-based agent teams with structured task handoff
- **LangGraph** — Graph-based workflow orchestration with state checkpointing
- **Docsify** — [docsify.js.org](https://docsify.js.org/) — Zero-build documentation rendering from plain markdown
