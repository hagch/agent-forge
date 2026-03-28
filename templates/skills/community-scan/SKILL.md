---
name: community-scan
description: >
  Use when searching for Claude Code skills, plugins, libraries, and
  development patterns that could improve the framework or solve
  encountered problems. Triggers: self-improvement phase, /self-improve
  command, after retro identifies gaps.
---

# Community Scan

Targeted search for external improvements to the framework and project.

## Search Targets

### 1. Claude Code Ecosystem
- GitHub: search for `claude-code skill`, `claude-code plugin`, `claude-code agent`
- npm: search for `@claude`, `claude-code`
- Claude Code marketplace / official plugins list
- Look for: skills that solve problems we encountered, new agent patterns

### 2. Library Updates
- For each library/framework the project uses:
  - Check latest version vs. installed version
  - Read changelogs for fixes relevant to encountered problems
  - Check for security advisories

### 3. Development Patterns
- Search for recent articles (last 3 months) about:
  - "Claude Code" + development patterns
  - "AI agent" + software development
  - "multi-agent" + coding
  - "self-improving" + agents
- Sources to check: Addy Osmani's blog, Anthropic's blog, Hacker News, DEV.to

### 4. Framework References
- Check for updates to:
  - Superpowers plugin (compare versions)
  - Ralph loop implementations
  - Agent Teams documentation
  - Compound AI research

## Output Format

```markdown
## Community Scan: <Date>

### Relevant Finds
- **<Title>** — <URL>
  - What: <What it does>
  - Relevance: <How it helps us>
  - Recommendation: INSTALL | EVALUATE | MONITOR

### Library Updates
- **<Library>** <current> → <latest>
  - Changes: <What's new>
  - Relevant: <Does it fix our problems?>

### New Patterns
- **<Pattern>** — <Source URL>
  - What: <Description>
  - Application: <How we could use it>

### No Relevant Finds For
- <Topic we searched but found nothing useful>
```

## Rules
- Include URLs for every find
- "No relevant finds" is a valid and useful result — say it explicitly
- Prioritize finds that match actual problems from the retro
- Do not recommend installing everything — only what clearly helps
