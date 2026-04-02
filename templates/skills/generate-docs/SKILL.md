---
name: generate-docs
description: >
  Use when a feature is verified and ready for user-facing documentation.
  Generates end-user guides, API docs, FAQ/troubleshooting from the Design Spec,
  DAU journey, and verification results. Triggers: after verification passes,
  generate user docs, write documentation, document feature.
---

# Generate Docs — User-Facing Documentation

Generate comprehensive user-facing documentation for the completed feature. This is NOT developer documentation (that already exists in the feature doc) — this is documentation for **end users** of the application.

**Iron Law:** Every doc section is populated from EXISTING artifacts (concept.md, plan.md, DAU journey, verification report, test results). Never invent information — if it's not in the artifacts, mark it `[NEEDS-REVIEW]`.

## Diataxis Framework

All generated docs follow the [Diataxis](https://diataxis.fr/) taxonomy:

| Type | Purpose | Audience | Style |
|------|---------|----------|-------|
| **Tutorial** | Learning-oriented walkthrough | First-time users | "Follow along with me" |
| **How-To Guide** | Task-oriented instructions | Competent users | "Steps to achieve X" |
| **Reference** | Information-oriented lookup | Any user | "Dry facts, complete, accurate" |
| **Explanation** | Understanding-oriented context | Curious users | "Why it works this way" |

## Inputs

Read these artifacts from the feature directory (`docs/agentforge/features/<domain>/<slug>/`):

1. **concept.md** — Design Spec (UX flows, functional areas, scope)
2. **plan.md** — Technical plan (API surface, data model)
3. **verification-report.md** — Verification results (DAU journey, test results)
4. **README.md** — Agent Logs (edge cases discovered, problems solved)

Also read:
5. **Existing user docs** in `docs/` — to match tone, structure, terminology
6. **OpenAPI/Swagger specs** — if they exist, for API reference generation
7. **i18n files** — for exact UI labels and terminology

## Step 1: Determine Doc Scope

Not every feature needs all doc types. Assess based on the feature:

| Feature Type | Docs Needed |
|-------------|-------------|
| User-facing UI feature | Tutorial + How-To + FAQ |
| API-only feature | API Reference + How-To |
| Configuration/Settings | How-To + Reference |
| Background/Invisible | Explanation only (if user-visible effects) |

Present the scope to the user: "For this feature, I'll generate: [list]. Agree?"

## Step 2: Generate Tutorial (if applicable)

**Source:** concept.md UX flows + DAU journey from verification

A tutorial walks a first-time user through the feature step by step.

<tutorial-template>

```markdown
# Getting Started with <Feature Name>

<One sentence: what this feature lets you do.>

## What You'll Learn

By the end of this guide, you'll be able to:
- <outcome 1>
- <outcome 2>

## Prerequisites

- <what the user needs before starting>

## Step 1: <Action>

<Imperative instruction — one action per step.>

You should see: <what appears on screen after this step.>

## Step 2: <Action>

...

## What's Next?

- <Link to How-To guides for advanced usage>
- <Link to Reference for all options>
```

</tutorial-template>

**Rules:**
- One action per step — imperative mood ("Click", "Select", "Enter")
- Reference UI elements by their EXACT label (read from i18n files or code)
- Include "You should see..." confirmation after key steps
- Derive steps from the UX flows in concept.md, validated against DAU journey
- If the DAU tester found confusing moments (frustration >= 3), add extra clarification at those steps

## Step 3: Generate How-To Guides (if applicable)

**Source:** concept.md functional areas + edge cases from Agent Logs

How-To guides solve specific user goals. Generate one per functional area.

<howto-template>

```markdown
# How to <achieve specific goal>

<One sentence: when you'd want to do this.>

## Steps

1. <action>
2. <action>
3. <action>

## Tips

- <helpful shortcut or alternative approach>

## Troubleshooting

If <problem>: <solution>
```

</howto-template>

**Rules:**
- Named as actions: "How to export data", "How to invite a team member"
- Assume a competent user — no hand-holding, just clear steps
- Include troubleshooting from edge cases discovered during testing
- Link to Reference for detailed parameter/option explanations

## Step 4: Generate API Reference (if applicable)

**Source:** plan.md API surface + OpenAPI spec + test fixtures

<api-reference-template>

For each endpoint:

```markdown
## <METHOD> <path>

<One-line description.>

### When to Use

<Common use case in 1-2 sentences.>

### Request

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|

**Example:**
\`\`\`bash
curl -X POST /api/<path> \
  -H "Content-Type: application/json" \
  -d '{ "field": "value" }'
\`\`\`

### Response

**Success (200):**
\`\`\`json
{ "id": "...", "status": "created" }
\`\`\`

**Error (400):**
\`\`\`json
{ "error": "validation_failed", "message": "..." }
\`\`\`

### Error Codes

| Code | Meaning | How to Fix |
|------|---------|-----------|
```

</api-reference-template>

**Rules:**
- Examples must be copy-pasteable (valid JSON, correct headers)
- Derive request/response examples from actual test fixtures, not invented data
- Include both success and error responses
- If OpenAPI spec exists, cross-reference for accuracy

## Step 5: Generate FAQ / Troubleshooting (if applicable)

**Source:** DAU journey (frustration points) + Agent Logs (edge cases) + verification findings

<faq-template>

```markdown
# <Feature Name> — FAQ & Troubleshooting

## Quick Fixes

<Top 3 most likely issues, one line each with solution.>

---

### Q: <Question a user would actually ask>

<Answer in 2-3 sentences. Include steps if actionable.>

### Q: <Question>

<Answer>
```

</faq-template>

**Rules:**
- Translate technical errors to user-friendly language
- Frame as questions the user would ask, not internal error names
- Derive from: DAU frustration >= 3 moments, test edge cases, verification findings
- Group by user workflow, not by error code
- Most common issues first

## Step 6: Write Docs to Docsify

Place generated docs in the Docsify structure:

```
docs/
└── <feature-domain>/
    └── <feature-slug>/
        ├── index.md              (Feature overview — links to all doc types)
        ├── getting-started.md    (Tutorial)
        ├── guides/
        │   ├── <goal-1>.md       (How-To Guide)
        │   └── <goal-2>.md       (How-To Guide)
        ├── reference.md          (API Reference / Configuration Reference)
        └── faq.md                (FAQ & Troubleshooting)
```

**Important:** Place user docs in `docs/<domain>/<slug>/`, NOT in `docs/agentforge/features/<domain>/<slug>/`. The `agentforge/` directory is for developer/feature docs. User docs live at the top level of `docs/`.

After writing, the PostToolUse hook automatically regenerates `docs/_sidebar.md`.

## Step 7: Self-Review

Before completing, verify:

- [ ] All UI element references match actual labels in the codebase
- [ ] All API examples are valid and copy-pasteable
- [ ] No `[NEEDS-REVIEW]` markers remain (or they are flagged to user)
- [ ] Links between docs work (tutorial links to how-to, how-to links to reference)
- [ ] Terminology is consistent with existing docs
- [ ] Docs follow Docsify link rules (paths from `docs/` root)
- [ ] No internal/developer jargon in user-facing docs

## Step 8: Commit

Commit the generated docs:

```bash
git add docs/<domain>/<slug>/
git commit -m "docs(<scope>): add user documentation for <feature>"
```

## Update CURRENT-STATE.md

```yaml
phase: generate-docs
status: completed
docs_generated:
  - tutorial: docs/<domain>/<slug>/getting-started.md
  - howto: docs/<domain>/<slug>/guides/
  - reference: docs/<domain>/<slug>/reference.md
  - faq: docs/<domain>/<slug>/faq.md
```

## Red Flags

| Signal | Problem |
|--------|---------|
| Inventing UI labels not found in code | STOP — read i18n files or component source |
| Writing developer docs instead of user docs | This skill is for END USERS, not developers |
| Long paragraphs of explanation in a How-To | Move explanation to a separate Explanation doc |
| API examples that aren't valid JSON/curl | Must be copy-pasteable |
| `[NEEDS-REVIEW]` still in final docs | Flag to user before completing |

---

## Terminal State

When all docs are written, reviewed, and committed:

> **Invoke `agentforge:finish-branch` skill** to push the branch with all changes.
