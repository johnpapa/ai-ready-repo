# How It Works

The AI-Ready skill prepares your repository for effective collaboration with GitHub Copilot by generating a set of structured assets. These assets work together through three complementary mechanisms — each serving a different purpose, read at a different time, and targeting a different aspect of how AI understands your project.

---

## The Three Mechanisms

### AGENTS.md — Project Context for the Coding Agent

**What it is:** A markdown file placed at the root of your repository that is automatically read by the Copilot coding agent (cloud agent).

**When it's read:** Every time the cloud agent starts working on a pull request, issue, or task in your repository. The agent reads this file before it writes any code — it's the first thing it sees.

**What it contains:**

- **Repository structure** — directory layout, key files, and how the codebase is organized
- **Build commands** — how to install dependencies, compile, and run the project
- **Test commands** — how to run the test suite, what frameworks are used, and any special flags
- **Release process** — versioning strategy, deployment steps, CI/CD triggers
- **Architectural patterns** — how components are structured, naming conventions, data flow
- **Feature creation guides** — step-by-step instructions for adding new functionality

**Think of it as:** The "new hire onboarding doc" for the AI. Just as you'd give a new developer a document explaining how the project works, what to build first, and how to ship code — AGENTS.md does the same for the coding agent.

**Why it matters:** Without AGENTS.md, the coding agent has to infer project structure from file names and code alone. With it, the agent knows exactly how to build, test, and contribute to your project from the start.

---

### .github/copilot-instructions.md — Coding Conventions for All Copilot

**What it is:** A repository-level instructions file that is automatically injected into every Copilot interaction within the repository.

**When it's read:** Every time anyone uses Copilot in this repo — Chat, code completions, pull request reviews, CLI, or any other Copilot surface. It applies to all contributors, not just the coding agent.

**What it contains:**

- **Language conventions** — preferred idioms, import styles, module patterns
- **Framework patterns** — how to use the project's frameworks correctly (e.g., component structure, state management)
- **Test conventions** — naming patterns, assertion style, mocking approach, what to test
- **Code style** — formatting preferences, naming conventions, comment expectations
- **Maintenance matrix** — a cross-reference of what to update when specific parts of the codebase change

**Think of it as:** The "style guide" that is enforced automatically. Instead of hoping every developer reads the style guide, Copilot reads it every time and follows it in every suggestion.

**Why it matters:** This ensures consistency across all AI-assisted code — whether it's an inline completion, a Chat response, or a full PR from the coding agent. Every Copilot interaction respects the same conventions.

---

### .github/skills/ — On-Demand Task Recipes

**What it is:** A directory of markdown files, each containing a step-by-step procedure that can be invoked by name during a Copilot interaction.

**When it's read:** Only when a user explicitly invokes a skill (e.g., "use the add-feature skill" or "follow the scaffolding skill"). Skills are not read automatically — they are pulled in on demand.

**What it contains:**

- **Procedural recipes** for common tasks — adding a new feature, scaffolding a module, setting up a new service
- **Step-by-step instructions** with specific file paths, commands, and patterns to follow
- **Project-specific workflows** tailored to your repository's conventions and structure

**Think of it as:** A "runbook" the AI follows. Rather than explaining a multi-step process every time, you codify it once as a skill and invoke it whenever needed.

**Why it matters:** Complex, multi-step tasks (like adding a new game scene, creating an API endpoint, or scaffolding a new package) require following a specific sequence. Skills encode that sequence so the AI executes it correctly every time.

---

## How the Three Mechanisms Work Together

| Mechanism | Scope | Trigger | Audience |
|---|---|---|---|
| `AGENTS.md` | Whole project context | Automatic (every coding agent task) | Coding agent only |
| `copilot-instructions.md` | Coding conventions | Automatic (every Copilot interaction) | All Copilot users |
| `.github/skills/` | Task-specific procedures | Manual (user invokes by name) | Whoever invokes the skill |

Together, they form a layered system:

1. **AGENTS.md** gives the AI the big picture — what the project is and how it works.
2. **copilot-instructions.md** defines the rules — how code should be written here.
3. **Skills** provide the playbooks — how to execute specific tasks step by step.

---

## The 12 Steps

The skill performs up to 12 steps, each serving a specific purpose:

### 0. GitHub Auto-Discovery

Before looking at local files, the skill pulls context directly from GitHub — repo metadata, community health score, recent merged PRs, PR review comments, CI workflows, and releases. PR review mining discovers repeated feedback patterns that become automated conventions.

### 1. Codebase Analysis

Scans the local repository for languages, frameworks, test setup, CI configuration, existing AI config, changelog, documentation, directory structure, and monorepo detection. Combined with Step 0, this produces a complete picture of the repo's current state.

### 2. AGENTS.md

The project context file for the coding agent. Contains repository structure, build/test/release commands, architectural patterns, and contribution guides. Placed at the repo root.

### 3. .github/copilot-instructions.md

The coding conventions file for all Copilot interactions. Includes language idioms, framework patterns, test conventions, code style rules, and the maintenance matrix. Lives in `.github/`.

### 4. .github/workflows/copilot-setup-steps.yml

Configuration for the Copilot coding agent's environment. Defines the setup steps the agent runs before working on your repo — installing dependencies, building the project, running any required bootstrapping. This ensures the agent's environment matches what a human developer would set up.

### 4b. .mcp.json

MCP server configuration connecting AI agents to your project's databases, APIs, and tools. Generated at the repo root (`.mcp.json`). Uses environment variable placeholders for secrets so the config is safe to commit.

### 5. CI Workflow (.github/workflows/ci.yml)

A GitHub Actions workflow for continuous integration. Runs your test suite and linters on pull requests and pushes. If your repo already has CI, the skill respects the existing configuration and may suggest improvements rather than replacing it.

### 6. Issue Templates (.github/ISSUE_TEMPLATE/)

Structured issue templates that guide contributors (both human and AI) to provide the right information when filing bugs, requesting features, or proposing changes. Well-structured issues lead to better coding agent output.

### 7. README Contributing Section

A contributing guide section for your README (or a standalone CONTRIBUTING.md) that explains how to set up the development environment, run tests, and submit changes. This helps both human contributors and provides additional context for AI tools.

### 8. Maintenance Matrix

A cross-reference table embedded in `copilot-instructions.md` that maps "when X changes, update Y." This is one of the most valuable assets — it ensures that when code changes, the related documentation, tests, templates, and CI configuration all stay in sync. See [The Maintenance Matrix](#the-maintenance-matrix) section below for details.

### 9. Changelog Evaluation

The skill checks whether the repo has a changelog and whether it's healthy. It handles non-standard locations — some projects maintain changelogs in their docs site rather than a root `CHANGELOG.md`. The skill follows pointer files, checks freshness against git tags, and flags stale changelogs. If no changelog exists, it creates one in Keep a Changelog format.

### 10. Documentation Evaluation

The skill checks whether the repo has documentation, identifies the framework (Docsify, Docusaurus, MkDocs, VitePress, etc.), verifies the docs deploy pipeline, and checks that the README links to the docs. If docs exist, their location and conventions are documented in AGENTS.md and the maintenance matrix. If docs don't exist, the skill assesses whether they're needed based on the project type.

### 11. AI-Readiness Report

Displays the final readiness report — current score, category breakdown (AI Context, Dev Workflow, Onboarding), what was created or updated, and a projected score. Offers to create a PR with all changes and add the AI-Ready badge to the README.

---

## How the Analysis Works

Before generating any assets, the skill performs a comprehensive analysis of your repository. This is what makes the output customized rather than generic.

The skill instructs Copilot to scan your repository for:

### Languages

Detects the programming languages used by examining manifest files:

- `package.json` → JavaScript/TypeScript
- `Cargo.toml` → Rust
- `go.mod` → Go
- `requirements.txt` / `pyproject.toml` → Python
- `*.csproj` / `*.sln` → C# / .NET
- And others

### Frameworks

Identifies the frameworks and libraries in use:

- **Frontend:** React, Vue, Angular, Svelte, Phaser
- **Backend:** Express, Fastify, Django, Flask, Actix, Gin
- **Mobile:** React Native, Flutter
- **Others** based on dependency declarations

### Test Setup

Determines how testing is configured:

- **Test runner** — Jest, Vitest, Playwright, pytest, go test, cargo test
- **Test directory** — where tests live (`tests/`, `__tests__/`, `src/**/*.test.*`)
- **Test commands** — how to invoke the test suite
- **Coverage configuration** — if and how coverage is measured

### CI/CD

Examines existing continuous integration and deployment:

- **Existing workflows** — what's already in `.github/workflows/`
- **PR triggers** — whether CI runs on pull requests
- **Build steps** — what the current pipeline does
- **Deployment targets** — where and how the project is deployed

### Existing AI Configuration

Checks what AI-readiness assets already exist:

- Does `AGENTS.md` already exist? What does it contain?
- Is there a `.github/copilot-instructions.md`?
- Are there existing skills in `.github/skills/`?
- Is `.github/workflows/copilot-setup-steps.yml` configured?

### Directory Structure

Maps the layout of the repository:

- Source directories and their organization
- Asset directories (images, sounds, static files)
- Configuration files and their locations
- Documentation structure

### What's Missing

Based on all of the above, the analysis identifies gaps — which of the 12 assets are missing, incomplete, or could be improved.

### Customized Output

The AI uses this analysis to generate assets that are specific to your project. For example:

- A **Phaser game** project gets AGENTS.md content about scene lifecycle, physics setup, and sprite management — not generic web app boilerplate.
- A **Go microservice** gets copilot-instructions about error handling patterns, interface conventions, and module structure — not React component guidelines.
- A **monorepo** gets skills that understand the package structure and cross-package dependencies.

This is the key difference from template-based approaches: every generated asset reflects your actual codebase.

---

## The Maintenance Matrix

The maintenance matrix is one of the most impactful assets the skill generates. It solves a common problem: when one part of the codebase changes, other parts need to be updated to stay in sync — but it's easy to forget which parts.

### The Problem

In any non-trivial project, changes ripple:

- Add a new API endpoint → update the OpenAPI spec, add tests, update the README
- Change a database schema → update the model, migration, seed data, and related tests
- Add a new game scene → register it in the game config, add assets, create tests, update the scene list

Without a reference, these secondary updates are often missed — leading to stale docs, missing tests, and broken workflows.

### The Solution

The maintenance matrix is a structured cross-reference table that explicitly maps these relationships:

```
| When this changes...        | Also update...                                    |
|-----------------------------|---------------------------------------------------|
| `src/game/scenes/`          | `src/game/game.ts` (scene registry), tests, AGENTS.md |
| `package.json` dependencies | `.github/workflows/copilot-setup-steps.yml`, CI workflow              |
| API routes                  | OpenAPI spec, API tests, README endpoints list      |
| Database schema             | Migrations, seed data, model tests                  |
```

### Where It Lives

The maintenance matrix is embedded in `.github/copilot-instructions.md`, which means it is automatically included in every Copilot interaction. When Copilot suggests a change to a file in the "when this changes" column, it knows to also suggest or remind about the corresponding updates.

### Why It Works

- **For the coding agent:** When working on a PR, the agent sees the matrix and proactively makes the secondary updates — resulting in more complete PRs that don't miss related changes.
- **For Copilot Chat:** When discussing a change, Copilot can reference the matrix to remind you what else needs updating.
- **For code review:** PR reviewers (human or AI) can check the matrix to verify all related updates were made.

The matrix is generated based on your actual repository structure and the relationships the analysis discovers — not a generic template.
