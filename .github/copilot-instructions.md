# Copilot Instructions — ai-ready

## Project Type

This is a **Copilot CLI plugin** containing only markdown, YAML, and JSON files. There is no source code, no build system, no test framework, and no runtime dependencies.

**Do not** generate or suggest build commands, test commands, package installs, or runtime setup for this repo.

## Writing Conventions

### Markdown

- Use ATX-style headings (`#`, `##`, `###`)
- Use fenced code blocks with language identifiers (```yaml, ```bash, ```json)
- Use tables for structured data — always include a header row
- Keep lines under 120 characters where practical
- Use `**bold**` for emphasis, `_italic_` for terms, `` `backticks` `` for file paths and commands

### YAML (skill frontmatter, workflows, issue templates)

- Use 2-space indentation
- Quote strings that contain special YAML characters
- Always include `name` and `description` in skill frontmatter

### JSON (plugin.json)

- Use 2-space indentation
- Keep `plugin.json` fields in this order: name, description, version, author, repository, license, keywords, skills
- Bump `version` on every meaningful change

## Skill Writing Conventions

- Each step should be independently actionable — the AI should be able to execute it without context from other steps
- Include explicit "check if exists" guards before creating files
- Use real file paths and real commands, not placeholders
- Prefer structured output (tables) over prose for analysis results
- Always end with a summary step listing what was created, skipped, and what to do next

## Maintenance Matrix

| When this changes... | Also update... |
|---|---|
| `skills/ai-ready/SKILL.md` | `README.md` (if skill behavior changed), `docs/how-it-works.md`, `AGENTS.md`, `CHANGELOG.md` |
| New skill added to `skills/` | `.github/plugin/plugin.json` (register in `skills` array), `README.md`, `AGENTS.md` (structure section), `CHANGELOG.md` |
| `.github/plugin/plugin.json` metadata | `README.md` (version, description), `AGENTS.md`, `CHANGELOG.md` |
| `docs/how-it-works.md` | Verify consistency with `SKILL.md` steps and `README.md` |
| `README.md` problem statement or architecture | Verify consistency with `docs/how-it-works.md` |
| Repo structure changes (new dirs, moved files) | `AGENTS.md` (structure section), `plugin.json` (skill paths), `CHANGELOG.md` |
| `hooks/hooks.json` | `plugin.json` (hooks path), `AGENTS.md` (structure section), `CHANGELOG.md` |
| Any version bump | `plugin.json` (version), `CHANGELOG.md` (new entry) |
