# AI-Ready Repo — Agent Guide

This is a **Copilot CLI plugin** — not a traditional application. It contains no source code to build or test. The deliverable is a skill definition (`SKILL.md`) and plugin manifest (`plugin.json`) that teach Copilot CLI how to make any repository AI-ready.

## Repository Structure

```
ai-ready/
├── .github/
│   ├── plugin/plugin.json          # Plugin manifest (name, version, skill references)
│   ├── copilot-instructions.md     # Conventions for contributing to THIS repo
│   ├── workflows/copilot-setup-steps.yml  # Cloud agent setup (checkout only — no build)
│   ├── dependabot.yml              # GitHub Actions dependency updates
│   ├── workflows/ci.yml            # PR validation (plugin integrity checks)
│   ├── ISSUE_TEMPLATE/             # Bug reports, feature requests, new skill ideas
│   ├── PULL_REQUEST_TEMPLATE.md    # PR checklist (integrity checks, test evidence)
│   └── CODEOWNERS                  # @johnpapa owns all paths
├── hooks/
│   └── hooks.json                  # sessionStart hook — shows daily tip on launch
├── skills/
│   ├── ai-ready/SKILL.md           # The 12-step skill procedure
│   └── daily-tip/SKILL.md          # Daily AI-ready tip shown on session start
├── docs/
│   └── how-it-works.md             # Detailed explanation of the 3 mechanisms + 9 assets
├── AGENTS.md                       # This file
├── CHANGELOG.md                    # Version history
├── README.md                       # Project overview, quick start, what gets generated
├── SECURITY.md                     # Vulnerability reporting policy
└── LICENSE                         # MIT
```

## Tech Stack

- **Content format:** Markdown, YAML, JSON
- **Plugin system:** Copilot CLI plugin spec (`.github/plugin/plugin.json`)
- **No runtime, build system, or test framework** — this is a documentation-driven project

## Build & Run

There is no build step. This repo ships markdown and JSON files that Copilot CLI reads directly.

**To test the plugin locally:**

```bash
# Load from local directory (no install needed)
copilot --plugin-dir /path/to/ai-ready

# Or install from GitHub
copilot plugin install johnpapa/ai-ready
```

Then start Copilot and invoke the skill:

```bash
copilot
```

```
make this repo ai-ready
```

## Testing

There is no automated test suite. Validation is:

1. **Plugin integrity** — `plugin.json` parses, all referenced skill paths exist, SKILL.md frontmatter is valid
2. **Smoke test** — install the plugin, invoke the skill on a sample repo, verify the analysis is correct and files are generated properly
3. **CI** — the workflow validates JSON/YAML syntax and skill path references on every PR

## Key Patterns and Conventions

- **Skills live in `skills/<name>/SKILL.md`** — each skill is a markdown file with YAML frontmatter (`name`, `description`) and step-by-step instructions
- **The skill is self-sufficient** — it uses Copilot's built-in tools (glob, grep, view, create) to analyze repos and generate files. No custom extensions or code required
- **plugin.json references skills by path** — the `skills` array in plugin.json must match the actual directory structure
- **Never overwrite existing files** — the skill checks for existing assets before generating

## Adding a New Skill

1. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: skill-name
   description: What this skill does and when to invoke it
   ---
   ```
2. Write step-by-step instructions in the markdown body
3. Register the skill in `.github/plugin/plugin.json` by adding `"./skills/<skill-name>"` to the `skills` array
4. Update `README.md` to mention the new skill
5. Update this file (`AGENTS.md`) to reflect the new structure
6. Bump the `version` in `plugin.json`

## Common Pitfalls

- **Don't add build/test/runtime dependencies** — this is a markdown-only project. Agents should not invent `npm install`, `pip install`, or any setup commands for this repo
- **Keep plugin.json and skill paths in sync** — if you rename or move a skill directory, update plugin.json
- **SKILL.md frontmatter is required** — the `name` and `description` fields in the YAML frontmatter are how Copilot discovers and matches the skill to user requests
- **Test on real repos** — the only meaningful test is invoking the skill on different repo types (Node.js, Python, Go, Rust, etc.) and verifying the output
