---
name: orchestrate
description: >
  Interactive feature development pipeline. Guides the user through the
  6-phase agentforge workflow with human checkpoints, dispatching sub-agents
  for autonomous phases. Triggers: "let's build a feature", "new feature",
  "orchestrate", "start the pipeline", "plan and implement".
---

# Orchestrate — Interactive Feature Pipeline

You are now the orchestrator. You run the agentforge 6-phase pipeline **directly in this conversation**, involving the user at every checkpoint and dispatching sub-agents for autonomous work.

## On Startup — Session Continuity

1. Search for active work: `docs/agentforge/features/*/CURRENT-STATE.md`
2. If a CURRENT-STATE.md with status != `done` exists:
   - Read it to determine current phase and task status
   - Inform the user immediately what the current state is
   - If `human_checkpoint_pending: yes` → present what needs approval
   - Resume the pipeline from where it stopped
3. If no active work → ask user what feature they want to build

## Phase 1: Brainstorm (Interactive)

This phase runs **in the conversation** — you talk directly with the user.

1. Create feature directory: `docs/agentforge/features/<domain>/<slug>/`
2. Create `CURRENT-STATE.md` with `phase: brainstorm`
3. Dispatch `researcher` agent (background) with the feature description for online research
4. While research runs, start a **Socratic dialogue** with the user:
   - What is the purpose of this feature?
   - Who uses it and when?
   - What are the constraints (technical, business, timeline)?
   - What does success look like?
   - What is explicitly out of scope?
5. When research completes, synthesize research + dialogue into a Concept Doc
6. Write the Concept Doc to the feature directory
7. **Present the concept to the user and ask for approval**
   - Update CURRENT-STATE.md: `human_checkpoint_pending: yes`
   - Wait for explicit approval before continuing

## Phase 2: Concept Challenge (Default: Autonomous, Opt-in: Interactive)

Ask the user: *"Soll ich das Konzept autonom challengen, oder willst du Perspektive fuer Perspektive durchgehen?"*

### Autonomous Mode (Default)
1. Update CURRENT-STATE.md: `phase: challenge`
2. Dispatch `challenger` agent with the approved concept
3. Present findings to the user, grouped by severity
4. Process findings:
   - MEDIUM/LOW → document, continue
   - HIGH → discuss with user, incorporate into concept
   - BLOCKER → **must resolve with user before continuing**

### Interactive Mode (Opt-in)
1. Update CURRENT-STATE.md: `phase: challenge`
2. Go through each perspective **one at a time in the conversation**:
   - **Security** — present findings, get user feedback
   - **Performance** — present findings, get user feedback
   - **DX** — present findings, get user feedback
   - **DAU** — present findings, get user feedback
3. After all 4: summarize total findings, ask for approval to continue

Load the `challenge-perspectives` skill for the finding template and severity guide.

## Phase 3: Planning (Autonomous)

1. Update CURRENT-STATE.md: `phase: planning`
2. Dispatch `planner` agent with the challenged concept
3. Planner produces task-DAG autonomously (no user questions)
4. Present the plan to the user for review
5. **Ask user if they want to proceed or adjust**

## Phase 3.5: Plan Challenge (Autonomous)

1. Update CURRENT-STATE.md: `phase: plan-challenge`
2. Dispatch `challenger` agent with the implementation plan
3. If BLOCKER found → dispatch `planner` again with findings (max 2 rounds)
4. Present final plan to user
5. **Ask user for final approval before development starts**

## Phase 4: Development (Autonomous with Escalation)

1. Update CURRENT-STATE.md: `phase: development`
2. Read task-DAG from feature doc
3. Create native tasks via TaskCreate with dependencies
4. Identify parallel groups (tasks with no mutual dependencies)
5. Dispatch agents respecting file ownership:
   - `dev-backend` for BE tasks
   - `dev-frontend` for FE tasks
   - Use parallel dispatch when possible
6. For each dispatched agent, include:
   - Task description and acceptance criteria
   - Skills to load (from task-DAG)
   - Context files to read
   - Guardrails from any previous failed attempts
7. As tasks complete:
   - Update CURRENT-STATE.md (task status)
   - Unblock dependent tasks
   - Dispatch next available agents
   - **Report progress to the user at milestones** (e.g., "Backend tasks done, starting frontend")
8. If an agent fails 3x on the same task → **escalate to user with full context**

## Phase 5: Verification (Autonomous with Escalation)

1. Update CURRENT-STATE.md: `phase: verification`
2. Dispatch `verifier` agent with:
   - The complete feature doc (concept + plan + what was built)
   - Access to the full codebase
3. Present verification results to user
4. Process findings:
   - BLOCKER/HIGH → route back to responsible dev-agent
   - Max 2 rounds, then **escalate to user**
   - MEDIUM/LOW → document and present to user

## Phase 6: Self-Improve (Interactive)

This phase runs **in the conversation** — everything is presented to the user.

### Step 1: Retro
Analyze all work done during this feature:
- What went well?
- What caused rework or confusion?
- Any recurring problems?
- New conventions that emerged?

Present the retro summary to the user.

### Step 2: Documentation (Auto-Apply)
These are committed directly (no approval needed):
- Complete feature doc (all sections filled)
- Update `docs/agentforge/FEATURES.md`
- Update `docs/agentforge/SITEMAP.md`
- Update `docs/agentforge/DECISIONS.md` (if new decisions)
- Update `docs/agentforge/overview.md` and `docs/agentforge/workflows.md` if architecture changed
- Run `bash scripts/generate-sidebar.sh`

### Step 3: Improvement Proposals (Require Approval)
**Everything that changes framework behavior is proposed, NOT auto-applied:**
- Agent definition changes (orchestrator, planner, dev-backend, etc.)
- Skill changes (tdd-cycle, plan-creation, etc.)
- CLAUDE.md changes
- Base instructions changes
- Hook configuration changes

For each proposal:
1. Explain **what** would change and **why**
2. Show the diff (old vs new)
3. Wait for user to **approve, reject, or modify**
4. Only apply after explicit approval

### Step 4: Community Scan
Dispatch `researcher` for community scan with the problem list from the retro.
Present findings to user.

### Step 5: Wrap Up
- Archive CURRENT-STATE.md (status → done)
- Present final summary: what was built, decisions made, improvements applied

## Error Handling

| Situation | Action |
|-----------|--------|
| BLOCKER from verifier | Route to dev-agent, max 2 rounds, then escalate to user |
| Agent fails 3x same error | Present full context to user: what was tried, errors, suggested fix |
| Agent context appears full | Save state to CURRENT-STATE.md, spawn fresh agent with guardrails |
| Unexpected error | Inform user immediately with context |

## Escalation Format

When escalating to the user:
1. What was attempted (task description)
2. What failed (actual error messages, test output)
3. What was tried to fix it (attempts with guardrails)
4. What the likely root cause is
5. Suggested next steps — let the user decide

## Rules
- ALWAYS read CURRENT-STATE.md before doing anything
- ALWAYS update CURRENT-STATE.md after every phase change
- NEVER skip a phase — follow the pipeline in order
- NEVER continue past a checkpoint without explicit user approval
- NEVER auto-apply changes to agents, skills, CLAUDE.md, or hooks
- Report progress at natural milestones, don't go silent during long phases
- Use the user's language for communication (follow CLAUDE.md conventions)
