# Changelog

## 0.5.0 — Feature Doc Overview Section

### Added
- **Feature template: Overview section** with Description, Core Functions, and Usage Scenarios. Gives feature docs a human-readable explanation beyond just technical implementation details.
- **Setup Step 6.9 enhanced:** When generating feature docs for existing features, setup now also populates the Overview section (description, core functions, usage scenarios) from code analysis.

## 0.4.0 — Sidebar Auto-Generation & Docsify Link Fix

### Added
- **`scripts/generate-sidebar.sh`:** Setup now creates a shell script that regenerates `docs/_sidebar.md` from the directory structure. Scans feature docs, domain specs, and all other doc directories. Extracts H1 titles for sidebar labels.
- **PostToolUse Hook:** Setup configures a `Write(docs/**)` hook in `.claude/settings.json` that runs the sidebar generator automatically after every doc change. New feature docs appear in navigation without manual intervention.

### Fixed
- **Docsify link resolution:** All links in `docs/agentforge/` files now use full paths from docs root (`agentforge/features/...`) instead of relative paths (`features/...`). Docsify resolves all links relative to `docs/` root, not the current file — relative paths from subdirectories caused 404s.
- **FEATURES.md template:** Added comment documenting the path rule so orchestrator and agents generate correct links.
- **Orchestrator Phase 6:** Now runs `scripts/generate-sidebar.sh` instead of manually regenerating sidebar.

## 0.3.0 — docs/agentforge/ Namespace & Enhanced Docsify

### Changed
- **Docs namespace:** All agentforge-managed documentation now lives under `docs/agentforge/` instead of `docs/memory/`. This keeps the project's `docs/` root clean for other documentation.
- **Path updates:** All agents (orchestrator), skills (state-management, finishing-branch), and setup references updated from `docs/memory/` to `docs/agentforge/`.

### Added
- **Top-down documentation arc** (inspired by codebase-to-course): Three new templates guide readers from user-visible behavior → architecture → data flows.
  - `docs/agentforge/README.md` — "What does this project do?" entry point
  - `docs/agentforge/overview.md` — Architecture with Mermaid diagrams
  - `docs/agentforge/workflows.md` — End-to-end sequence diagrams with tabs
- **Docsify plugins:** Mermaid diagrams, flexible-alerts (NOTE/TIP/WARNING callouts), tabs, pagination (Previous/Next), sidebar-collapse, copy-code
- **Setup steps 6.2–6.4:** Generate the three top-down documentation pages from detected project structure

## 0.2.0 — Docsify & Feature Doc Fixes

### Fixed
- **Docsify 404 on root:** Setup now always creates `docs/README.md` as landing page (Docsify requires it)
- **Incomplete sidebar:** `docs/_sidebar.md` now scans the ENTIRE `docs/` directory recursively, not just `docs/agentforge/`
- **index.html improvements:** Pinned docsify@4 CDN, removed `relativePath: true` (breaks nested sidebar links), added proper HTML5 meta tags

### Added
- **Step 6.5 — Feature Docs for existing features:** Setup now generates individual feature docs (`docs/agentforge/features/<domain>/<slug>/README.md`) for each detected existing feature, pre-filled with routes, endpoints, and key files
- **README.md template:** Includes instructions for dynamic generation during setup
- **index.html template comments:** PROJECT_NAME placeholders for easier customization

## 0.1.0 — Initial Release

- 7 agent definitions (orchestrator, researcher, challenger, planner, dev-backend, dev-frontend, verifier)
- 12 skills (10 runtime + 2 commands)
- /setup command with repo analysis and coding guideline extraction
- /self-improve command with Superpowers diff, online research, community scan
- Docsify integration with search
- Session continuity via CURRENT-STATE.md
- Feature documentation templates
