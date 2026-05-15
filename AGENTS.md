# AI-Ready Repo — Agent Guide

This is a **Copilot CLI skill** — not a traditional application. It contains no source code to build or test. The deliverable is a skill definition (`SKILL.md`) that teaches Copilot CLI how to make any repository AI-ready.

## Repository Structure

```
ai-ready/
├── .github/
│   ├── copilot-instructions.md     # Conventions for contributing to THIS repo
│   ├── workflows/copilot-setup-steps.yml  # Cloud agent setup (checkout only — no build)
│   ├── dependabot.yml              # GitHub Actions dependency updates
│   ├── workflows/ci.yml            # PR validation (skill integrity checks)
│   ├── ISSUE_TEMPLATE/             # Bug reports, feature requests, new skill ideas
│   ├── PULL_REQUEST_TEMPLATE.md    # PR checklist (integrity checks, test evidence)
│   └── CODEOWNERS                  # @johnpapa owns all paths
├── skills/
│   └── ai-ready/
│       ├── SKILL.md                   # The 12-step skill procedure (<500 lines)
│       └── references/               # Detailed reference material (loaded on demand)
│           ├── github-discovery.md   # GitHub API tables, PR mining, health gaps
│           ├── detection-tables.md   # Manifest detection, course/monorepo heuristics
│           ├── report-template.md    # Report format, HTML spec, badge, PR flow
│           └── training-repos.md     # Repos used to validate skill heuristics
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
- **No runtime, build system, or test framework** — this is a documentation-driven project

## Build & Run

There is no build step. This repo ships markdown and JSON files that Copilot CLI reads directly.

**To test the skill locally:**

```bash
# Copy the skill to your personal skills directory
cp skills/ai-ready/SKILL.md ~/.copilot/skills/ai-ready/SKILL.md
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

1. **Skill integrity** — SKILL.md exists and frontmatter is valid
2. **Smoke test** — install the skill, invoke it on a sample repo, verify the analysis is correct and files are generated properly
3. **CI** — the workflow validates YAML syntax and skill frontmatter on every PR

## Key Patterns and Conventions

- **Skills live in `skills/<name>/SKILL.md`** — each skill is a markdown file with YAML frontmatter (`name`, `description`) and step-by-step instructions
- **The skill is self-sufficient** — it uses Copilot's built-in tools (glob, grep, view, create) to analyze repos and generate files. No custom extensions or code required
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
3. Update `README.md` to mention the new skill
4. Update this file (`AGENTS.md`) to reflect the new structure

## Common Pitfalls

- **Don't add build/test/runtime dependencies** — this is a markdown-only project. Agents should not invent `npm install`, `pip install`, or any setup commands for this repo
- **SKILL.md frontmatter is required** — the `name` and `description` fields in the YAML frontmatter are how Copilot discovers and matches the skill to user requests
- **Test on real repos** — the only meaningful test is invoking the skill on different repo types (Node.js, Python, Go, Rust, etc.) and verifying the output
