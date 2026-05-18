# AI Ready

[![AI Ready](https://img.shields.io/badge/AI--Ready-yes-brightgreen?style=flat)](https://github.com/johnpapa/ai-ready)

A Copilot CLI skill that analyzes your repository and generates the configuration files AI agents need to contribute correctly. **GitHub-native** — it auto-discovers your repo's context, community health, and PR review patterns without you explaining anything.

## Quick Start

Install the skill from inside Copilot CLI:

```
copilot plugin install johnpapa/ai-ready
```

Then inside Copilot CLI, type:

```
make this repo ai-ready
```

The skill analyzes your code, CI, tests, docs, and structure, then generates assets customized to your project — not generic templates.


### Run it again anytime

The skill is safe to re-run. On the first run, it creates missing assets. On subsequent runs, it **audits** your existing AI-ready files against the current state of your codebase — flagging drift like outdated build commands, stale repo structure, or new PR review patterns that should become conventions. It never overwrites your files — it suggests updates and lets you decide.

### Keeping updated

Update to the latest version:

```
copilot plugin update ai-ready
```

### Skip what you don't need

Add exclusions to your prompt and the skill will respect them:

```
make this repo ai-ready but skip CI and issue templates
```

```
just generate AGENTS.md and copilot-instructions
```

### Report only

Just want to see where you stand? Ask for a report without generating any files:

```
how ai-ready is this repo?
```

```
score this repo
```

## What to Expect

After you run the skill, you get a full AI-readiness report — analysis, proposed changes, and a projected score. Here's what it looks like for [vscode-peacock](https://github.com/johnpapa/vscode-peacock):

![HTML readiness report](images/report-html.png)

> 🔗 [View the interactive version](https://johnpapa.github.io/ai-ready/examples/sample-report-peacock.html) — collapsible sections, responsive layout, works on mobile.

The report shows:
1. **Your Repo Today** — current score, what's nailed, what's missing, and why it matters
2. **What I'd Like To Do** — proposed files to create (nothing changes until you say so)
3. **If You Accept** — projected score with category breakdown
4. **Ready?** — offer to create a PR with all the changes

> 📄 Also available as [terminal output](examples/sample-report-peacock.md) — same content, rendered in the CLI.

### Scoring

Your score is simple: **how many of the 12 tracked assets are nailed.** That's it — no formulas, no weights.

| Medal | Name | Count | What it means |
|-------|------|-------|---------------|
| 🥉 | **Getting Started** | 1–4 of 12 | Basics are in place but AI agents are mostly guessing |
| 🥈 | **On Track** | 5–7 of 12 | AI agents can help but they'll miss your conventions |
| 🥇 | **Solid** | 8–10 of 12 | AI agents follow your patterns and catch most expectations |
| 🏆 | **AI-Ready** | 11–12 of 12 | AI agents contribute like your best team members |

## Why

Contributors (human and AI) show up to your repo and don't know the conventions. They submit PRs that miss tests, break patterns, skip docs. You leave the same review comments on every PR. AI agents make this more challenging — they generate PRs faster, but without context, those PRs create _more_ review burden.

It's the same gap from both sides: **contributors don't know what maintainers expect, and maintainers keep re-teaching it.** This skill closes that gap by generating repo-level configuration that teaches everyone — human and AI — how to work in the repo correctly. It even mines your PR review comments for repeated feedback and turns them into automated conventions. The result: a 45-minute review becomes a 5-minute review.

### Built from Real Maintainer Experience

This skill isn't theoretical — it's shaped by [John Papa](https://github.com/johnpapa)'s experience maintaining popular open source projects and repos at large enterprises. The skill is tuned to prioritize what actually reduces review burden: maintenance matrices that catch the files contributors always forget, conventions mined from the PR feedback you're tired of repeating, and CI that catches problems before you have to.

## How It Works — GitHub-Native by Default

You shouldn't have to explain to an AI tool that you're in a GitHub repo. This skill assumes it, and leverages everything GitHub already knows about your project.

### Auto-Discovery (zero user input)

The skill starts by pulling context directly from GitHub — no questions asked:

| What it discovers | How | Why it matters |
|------------------|-----|----------------|
| Repo description, topics, languages | GitHub API | Knows what your project is without reading every file |
| Community health score | GitHub API | Instantly knows which config files are missing |
| Contributors | GitHub API | Team size, contribution patterns |
| Recent merged PRs | GitHub API | Understands what typical contributions look like |
| **PR review comments** | GitHub API | **Turns your repeated review feedback into automated conventions** |
| CI/CD workflows | GitHub Actions API | Knows your build/test pipeline |
| Releases | GitHub API | Understands versioning and release cadence |

It then scans your local codebase for deeper details — manifest files, test configs, directory structure, existing AI configuration — and combines both into a complete picture.

### PR Review Mining — The Killer Feature

This is the highest-value thing the skill does. It reads your recent PR review threads and looks for **repeated feedback** — the same comments you leave on every PR:

- _"Please add tests for new features"_ → becomes a test convention rule
- _"Use the X pattern instead of Y"_ → becomes a coding convention rule
- _"Don't forget to update the changelog"_ → becomes a maintenance matrix entry
- _"This breaks on mobile, check responsive layout"_ → becomes a screen size rule

These mined conventions go directly into `copilot-instructions.md`. The next AI-generated PR follows those rules automatically. You stop repeating yourself.

### What Gets Generated

Every file is customized to your repo's actual language, framework, and patterns — not generic boilerplate. The skill only creates files that don't already exist.

| File | What It Does |
| --- | --- |
| **`AGENTS.md`** | Project context for the coding agent — repo structure, build/test commands, architectural decisions, how to add features |
| **`.github/copilot-instructions.md`** | Coding conventions for all Copilot interactions — Chat, completions, PR reviews, CLI. Includes a maintenance matrix of what to update when code changes |
| **`.github/workflows/copilot-setup-steps.yml`** | Cloud agent environment setup — runtime versions, dependencies, build steps |
| **`.github/workflows/ci.yml`** | PR validation pipeline — build, test, lint, typecheck. Skips non-code changes (docs, images, etc.) |
| **`.github/ISSUE_TEMPLATE/bug-report.yml`** | Structured bug report form with fields relevant to your project type |
| **`.github/ISSUE_TEMPLATE/feature-request.yml`** | Structured feature request form |
| **`.github/PULL_REQUEST_TEMPLATE.md`** | PR description template with checklist items derived from the maintenance matrix |
| **`CHANGELOG.md`** | Keep a Changelog format, populated from releases/tags if available |
| **`.mcp.json`** | MCP server config connecting AI agents to your project's databases, APIs, and tools |
| **README `## Contributing` section** | Onramp for new contributors — how to fork, build, test, and submit a PR |
| **AI-Ready badge in README** | Shields.io badge linking back to this skill — added automatically |

## Two Layers of PR Quality

The assets this skill generates enable two complementary layers of PR quality — one you get automatically, one you enable:

| Layer | What it catches | How it works |
|-------|----------------|--------------|
| **CI workflow** (generated by this skill) | Broken builds, failing tests, lint errors | GitHub Actions runs on every PR — validates that the code compiles and tests pass |
| **Copilot code review** (you enable this) | Convention violations, missing docs/tests, maintenance matrix gaps | Copilot reads `copilot-instructions.md` (generated by this skill) and reviews PRs against your conventions |

Together: PRs are validated for **correctness** (CI) and reviewed for **quality** (Copilot). This skill generates the inputs for both — the CI workflow and the conventions file that Copilot code review reads.

To enable Copilot code review: go to your repo's **Settings → Copilot → Code review** and turn it on. Once enabled, every PR is automatically reviewed against the conventions in `copilot-instructions.md`.

## Tested Against

This skill has been validated against a diverse set of repos — courses, applications, VS Code extensions, monorepos, and more. See [skills/ai-ready/references/training-repos.md](skills/ai-ready/references/training-repos.md) for the full list.

## Contributing

### Quick Start

1. Fork this repo and create a branch
2. Make your changes (skills or docs)
3. Test locally: copy `skills/ai-ready/SKILL.md` to `~/.copilot/skills/ai-ready/SKILL.md`, start `copilot`, then say *"make this repo ai-ready"*
4. Open a PR

See [AGENTS.md](AGENTS.md) for the full contributor guide.

## Credits

This skill was strengthened using:

- **[Sensei](https://github.com/spboyer/sensei)** — AI skill quality scorer that evaluates frontmatter, structure, and discoverability. Used to achieve a "High" rating.
- **[Agent Skills Spec](https://github.com/github/awesome-copilot/blob/main/docs/README.skills.md)** — GitHub's specification for skill authoring. Used to refactor from 964 lines to a 225-line spec-compliant structure with progressive disclosure.

## License

MIT
