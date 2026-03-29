---
name: setup
description: >
  Use to initialize the agentforge framework in a project. Analyzes the
  repository, extracts coding conventions, copies agent/skill templates,
  configures project-specific settings, and sets up Docsify documentation.
  Triggers: first time using agentforge, /setup command, re-run to update.
---

# /setup — Framework Initialization

Initialize or update the agentforge framework in the current project.

## Step 1: Detect Existing Installation

Check if `.claude/agents/orchestrator.md` exists:
- **Yes:** This is an UPDATE. Read existing files to preserve local modifications.
- **No:** This is a FRESH INSTALL.

## Step 2: Install Framework Files

Copy from plugin templates to project:

### Agents
For each agent template in `templates/agents/`:
- If fresh install: copy to `.claude/agents/`
- If update: compare local vs template. If local was modified, keep local. If identical to previous template version, update.

### Skills
For each skill template in `templates/skills/`:
- Same logic as agents: fresh → copy, update → diff and decide.

### Docs
- Create `docs/agentforge/` directory structure
- Create `docs/agentforge/features/<domain>/` directories for each detected domain
- Copy templates for SITEMAP.md, FEATURES.md, DECISIONS.md (only if they don't exist)
- Copy feature-template.md to `docs/agentforge/`
- Copy course-style templates (overview.md, workflows.md) to `docs/agentforge/` (only if they don't exist)
- **IMPORTANT — Docsify requires these files to avoid 404:**
  - Copy `docs/index.html` (only if it doesn't exist) — includes Mermaid, flexible-alerts, tabs, pagination, sidebar-collapse, copy-code plugins
  - **ALWAYS create `docs/README.md`** — Docsify uses this as the landing page. Without it, the root URL returns 404. Generate it from the project name and detected structure. Link to `agentforge/README.md` as the main knowledge base entry.
  - **ALWAYS create `docs/_sidebar.md`** — must include ALL existing docs, not just the `docs/agentforge/` files. Scan the entire `docs/` directory recursively and generate sidebar entries for every `.md` file found.

## Step 3: Analyze Repository

### Detect Structure
- Is this a monorepo (Nx, Turborepo, Lerna)? Check for `nx.json`, `turbo.json`, `lerna.json`
- Backend language? Check for `build.gradle`, `pom.xml`, `go.mod`, `Cargo.toml`, `requirements.txt`, `package.json`
- Frontend framework? Check for `next.config`, `nuxt.config`, `vite.config`, `angular.json`
- Database? Check for migrations directory, `flyway`, `alembic`, `prisma`
- API specs? Check for `openapi/`, `swagger/`, `graphql/`

### Derive Domains
- From directory structure: `src/modules/`, `src/features/`, `apps/`
- From API specs: separate spec files often = separate domains
- From Spring Modulith / package structure
- Create domain directories in `docs/agentforge/features/<domain>/`

### Detect Existing Features
- Scan routes/pages for frontend features
- Scan API endpoints for backend features
- Populate `docs/agentforge/FEATURES.md` with discovered features (status: EXISTING)
- Generate `docs/agentforge/SITEMAP.md` from discovered routes

## Step 4: Extract Coding Guidelines & Patterns

For each detected layer, read actual code to extract patterns:

### Backend Patterns
- Find entity/model files → extract base class, annotations, naming
- Find repository files → extract interface style, query patterns
- Find service files → extract injection style, transaction handling
- Find controller files → extract API pattern, response format
- Find test files → extract framework, assertion style, mocking approach
- Find migration files → extract naming format, data types

### Frontend Patterns
- Find page/view components → extract structure (thin page? fat component?)
- Find form components → extract form library, validation approach
- Find UI components → extract component library, styling approach
- Find test files → extract framework, mocking setup
- Find i18n files → extract i18n library, key structure

### Integration Patterns
- Find code generation config → extract tools (OpenAPI Generator, Orval, Prisma)
- Find build config → extract build steps, generation hooks
- Find API specs → extract URL patterns, response formats, error handling

### Output
Present findings to user:
```markdown
## Detected Coding Standards

### Backend (<language>)
<pattern with code snippet>

### Frontend (<framework>)
<pattern with code snippet>

### Database (<migration tool>)
<pattern with code snippet>

### API (<spec format>)
<pattern with code snippet>
```

If CLAUDE.md exists: show gaps (standards in code but not in CLAUDE.md).
If CLAUDE.md does not exist: offer to generate one.

Wait for user approval before writing CLAUDE.md changes.

## Step 5: Configure Agents

Set project-specific values:
- `dev-backend.md`: Set file ownership paths based on detected backend directory
- `dev-frontend.md`: Set file ownership paths based on detected frontend directory

## Step 6: Setup Docsify

### 6.1: Generate `docs/README.md` (landing page)
- **CRITICAL:** Docsify returns 404 without a `docs/README.md`. Always create one.
- Content: Project name, short description, link to `agentforge/README.md` as the knowledge base entry point.

### 6.2: Generate `docs/agentforge/README.md` (top-down entry point)
- Content: "What does this project do?" — user-visible behavior, key user flows, reading guide.
- Fill in from detected features, routes, and project description.
- This is **Phase 1** of the top-down documentation arc.

### 6.3: Generate `docs/agentforge/overview.md` (architecture)
- Content: System diagram (Mermaid), component descriptions, tech stack, key patterns.
- Fill in from detected structure (Step 3) and coding patterns (Step 4).
- Use Mermaid `graph TB` for architecture diagrams.
- This is **Phase 2** of the top-down arc.

### 6.4: Generate `docs/agentforge/workflows.md` (data flows)
- Content: End-to-end sequence diagrams (Mermaid), data flow patterns, integration points.
- Fill in from detected API endpoints, routes, and service interactions.
- Use Mermaid `sequenceDiagram` for request flows.
- Use `<!-- tabs:start -->` / `<!-- tabs:end -->` for read/write/error flow tabs.
- This is **Phase 3** of the top-down arc.

### 6.5: Create `scripts/generate-sidebar.sh`

Create a shell script that regenerates `docs/_sidebar.md` from the directory structure. This script is the single source of truth for sidebar generation — never hand-edit the sidebar.

The script must:
1. Write the static agentforge section (README, overview, workflows, FEATURES, DECISIONS, SITEMAP)
2. Scan `docs/agentforge/features/` recursively for `README.md` files
3. Extract the H1 title from each feature doc (`grep -m1 '^# '`)
4. Strip the "Feature: " prefix from titles
5. Generate sidebar entries with full paths from docs root (e.g., `agentforge/features/tenant/tenant-mgmt/README.md`)
6. Scan remaining doc directories (domain specs, branding, plans, specs, etc.)
7. Extract H1 titles from each doc for the sidebar label

Make the script executable: `chmod +x scripts/generate-sidebar.sh`

### 6.6: Configure PostToolUse Hook

Add a hook to `.claude/settings.json` that runs the sidebar generator after every `Write` to `docs/`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(docs/**)",
        "hooks": [
          {
            "type": "command",
            "command": "bash scripts/generate-sidebar.sh"
          }
        ]
      }
    ]
  }
}
```

This ensures the sidebar stays in sync whenever docs are created or modified — including when the orchestrator creates new feature docs in Phase 6.

### 6.7: Copy `docs/index.html` from template (if it doesn't exist)

### 6.8: Suggest serve command
- If `package.json` exists: add `"docs:serve": "npx docsify-cli serve docs"`
- If Nx workspace: suggest Nx target

## Step 6.9: Generate Feature Docs for Existing Features

For each feature detected in Step 3:
1. Create a feature doc from `feature-template.md` in `docs/agentforge/features/<domain>/<slug>/README.md`
2. Set status to `EXISTING`
3. Fill in known information from code analysis:
   - **Routes** from detected frontend pages
   - **API Endpoints** from detected controllers/OpenAPI specs
   - **Key Files** (entities, services, components) in Implementation Notes
4. Leave concept/plan/challenge sections empty (feature predates agentforge)
5. Update the feature link in `docs/agentforge/FEATURES.md` to point to the doc

> **IMPORTANT — Docsify Link Paths:** Docsify resolves ALL markdown links relative to the `docs/` root, NOT relative to the current file. Links in `docs/agentforge/FEATURES.md` to feature docs MUST use full paths from docs root:
> - CORRECT: `[Tenant Management](agentforge/features/tenant/tenant-mgmt/README.md)`
> - WRONG: `[Tenant Management](features/tenant/tenant-mgmt/README.md)`
>
> This applies everywhere: FEATURES.md, SITEMAP.md, overview.md, workflows.md, feature docs linking to other feature docs, etc.

## Step 7: Report

Present summary:
```markdown
## agentforge Setup Complete

### Installed
- 7 agent definitions in .claude/agents/
- 10 skills in .claude/skills/
- Documentation templates in docs/agentforge/

### Detected
- Structure: <monorepo/single>
- Backend: <language/framework>
- Frontend: <framework>
- Domains: <list>
- Existing Features: <count>

### Configured
- File ownership: dev-backend → <paths>, dev-frontend → <paths>
- CLAUDE.md: <generated/updated/unchanged>
- Docsify: <ready at docs:serve>

### Documentation (top-down arc)
- docs/agentforge/README.md — What does this project do?
- docs/agentforge/overview.md — Architecture overview with diagrams
- docs/agentforge/workflows.md — End-to-end data flows
- docs/agentforge/FEATURES.md — Feature registry (<count> features)
- docs/agentforge/DECISIONS.md — Architecture decisions

### Next Steps
1. Review .claude/agents/ — adjust if needed
2. Review CLAUDE.md — add missing standards
3. Run `npm run docs:serve` to browse documentation
4. Start your first feature with the orchestrator
```
