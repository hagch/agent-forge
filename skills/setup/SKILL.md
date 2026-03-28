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
- Create `docs/memory/` directory structure
- Copy templates for SITEMAP.md, FEATURES.md, DECISIONS.md (only if they don't exist)
- Copy `docs/index.html` for Docsify (only if it doesn't exist)
- Copy feature-template.md

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
- Create domain directories in `docs/memory/features/<domain>/`

### Detect Existing Features
- Scan routes/pages for frontend features
- Scan API endpoints for backend features
- Populate `docs/memory/FEATURES.md` with discovered features (status: EXISTING)
- Generate `docs/memory/SITEMAP.md` from discovered routes

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

1. Generate `docs/_sidebar.md` from current `docs/` structure
2. Add script to project:
   - If `package.json` exists: add `"docs:serve": "npx docsify-cli serve docs"`
   - If Nx workspace: suggest Nx target
3. Test: run `npx docsify-cli serve docs` briefly to verify it works

## Step 7: Report

Present summary:
```markdown
## agentforge Setup Complete

### Installed
- 7 agent definitions in .claude/agents/
- 10 skills in .claude/skills/
- Documentation templates in docs/memory/

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

### Next Steps
1. Review .claude/agents/ — adjust if needed
2. Review CLAUDE.md — add missing standards
3. Run `npm run docs:serve` to browse documentation
4. Start your first feature with the orchestrator
```
