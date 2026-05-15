---
name: ai-ready
description: "**ANALYSIS SKILL** — Analyze any repository and generate AI-ready configuration — AGENTS.md, copilot-instructions.md, skills, CI workflows, issue templates. WHEN: \"make this repo ai-ready\", \"set up AI config\", \"add copilot instructions\", \"prepare this repo for AI contributions\", \"generate AGENTS.md\". INVOKES: glob, grep, view, create, edit for repo analysis and file generation. FOR SINGLE OPERATIONS: use create/edit directly for individual config files."
---

# AI-Ready Repo Skill

## Persona

When executing this skill, adopt the perspective of an experienced open source and enterprise repo maintainer — someone who has managed high-traffic repos, reviewed thousands of PRs, and felt the pain of repeating the same feedback on every contribution.

Prioritize what **reduces review burden and contributor friction**. Don't just generate config files — generate config that solves real problems:

- A maintenance matrix that catches the files contributors always forget to update
- Conventions mined from PR reviews that stop the same mistakes from recurring
- An AGENTS.md that answers the questions maintainers are tired of re-explaining
- CI that catches broken PRs before a human has to look at them

Every decision you make should pass this test: **"Would an experienced maintainer want this in their repo?"** If the answer is no, leave it out.

*Why?*: Generic boilerplate creates noise. Maintainers already have too much config to manage. Every file you generate should earn its place.

---

When invoked, follow these steps in order to analyze the current repository and generate all missing AI-ready configuration assets.

**First run vs. re-run:** The skill runs the same analysis every time. On the first run, most assets will be missing — the skill creates them. On re-runs, it **audits** existing assets against the current codebase, checking for drift, stale content, and new conventions from recent PR reviews. Drifted assets land in "Could Be Better" with specific suggestions. Only truly missing assets are created automatically.

The skill **never overwrites existing files without user approval**.

*Why?*: Overwriting existing files destroys customizations the maintainer already made. Suggest changes — don't force them.

**Skipping assets:** If the user's prompt mentions skipping specific assets (e.g., "make this repo ai-ready but skip CI and issue templates"), respect those exclusions. Still run the full analysis, but skip generation for the excluded assets and note them as "⏭️ Skipped (user requested)" in the report. The analysis is always complete — only generation is skipped.

**Report-only mode:** If the user asks for a report without generating files (e.g., "how ai-ready is this repo?", "score this repo", "just show me the report", "check my repo's ai-readiness"), run the full analysis (Steps 0 and 1) and display the AI-Readiness Report (Step 11) — but **skip all generation steps** (Steps 2–10). The "What I'd Like To Do" section should still show what *would* be created, but nothing is written to disk. The "Ready?" section at the end should offer to run the full skill if they want to proceed.

*Why?*: Sometimes you just want to know where you stand before committing to changes. Report-only mode lets you assess without modifying anything.

### The 12 tracked assets

*Why?*: One score, at a glance. Count how many assets are nailed — that's it. No formulas, no weights. The progress bar and category breakdown show you where to focus.

Assets are grouped into three categories:

**🤖 AI Context** — what AI agents read to understand your repo

| # | Asset | Generated in |
|---|-------|-------------|
| 1 | `AGENTS.md` | Step 2 |
| 2 | `.github/copilot-instructions.md` | Step 3 |
| 3 | Maintenance matrix (in `copilot-instructions.md`) | Step 8 |
| 4 | `.vscode/mcp.json` | Step 4b |
| 5 | `.github/workflows/copilot-setup-steps.yml` | Step 4 |

**🔧 Dev Workflow** — what keeps PRs clean and contributors on track

| # | Asset | Generated in |
|---|-------|-------------|
| 6 | CI workflow (`.github/workflows/ci.yml`) | Step 5 |
| 7 | Issue templates (`.github/ISSUE_TEMPLATE/`) | Step 6 |
| 8 | PR template (`.github/PULL_REQUEST_TEMPLATE.md`) | Step 6 |
| 9 | `.github/dependabot.yml` | (checked, not generated) |

**📖 Onboarding** — what helps new contributors get started

| # | Asset | Generated in |
|---|-------|-------------|
| 10 | README Contributing section | Step 7 |
| 11 | Changelog (`CHANGELOG.md`) | Step 9 |
| 12 | Documentation (or explicit "not needed" note) | Step 10 |

**Scoring:** Count assets with **Nailed It** status. That's your score.
- **Nailed It** 🟩 = asset exists and is well-customized
- **Could Be Better** 🟨 = asset exists but has gaps (not counted toward score)
- **Missing** ⬜ = asset does not exist (not counted toward score)

### Maturity levels

*Why?*: A number alone doesn't tell you where you stand. Medals give you a clear target — most teams should aim for 🥇 Solid.

| Medal | Name | Nailed It count | What it means |
|-------|------|----------------|---------------|
| 🥉 | **Getting Started** | 1–4 of 12 | Basics are in place but AI agents are mostly guessing |
| 🥈 | **On Track** | 5–7 of 12 | AI agents can help but they'll miss your conventions |
| 🥇 | **Solid** | 8–10 of 12 | AI agents follow your patterns and catch most expectations |
| 🏆 | **AI-Ready** | 11–12 of 12 | AI agents contribute like your best team members |

---

## Step 0 — Detect GitHub context automatically

**This step requires zero user input.** The skill is GitHub-native — it discovers everything about the repo from GitHub's own tools and services.

*Why?*: The user shouldn't have to explain their own repo. GitHub already knows the language, the CI setup, the community health, and the PR patterns. Use that.

### 0a. Identify the repo

Run `git remote -v` or `git config --get remote.origin.url` to extract the GitHub `owner/repo`. This is your identity — everything else flows from it. If the remote is not GitHub, fall back to local-only analysis in Step 1.

### 0b. Fetch repo metadata from GitHub

Use the GitHub MCP tools (if available) or `gh` CLI to pull rich context the user should never have to explain:

| What to fetch | Tool / Command | What you learn |
|---------------|---------------|----------------|
| Repo description, topics, visibility, default branch | `github-mcp-server-get_file_contents` on `/` or `gh repo view --json description,topics,isPrivate,primaryLanguage,defaultBranchRef --jq '.' | cat` | What this project is about, how it's categorized, default branch name |
| Language breakdown | `gh api repos/{owner}/{repo}/languages | cat` (bash) | Accurate language percentages (better than guessing from files) |
| Community health | `gh api repos/{owner}/{repo}/community/profile | cat` (bash) | Which community files exist (CONTRIBUTING, CODE_OF_CONDUCT, license, issue templates) — GitHub already knows this |
| Contributors | `gh api repos/{owner}/{repo}/contributors --jq '.[].login' | cat` (bash) | Team size, contribution patterns |
| Open issues | `github-mcp-server-list_issues` or `gh issue list | cat` | Active problems, what the project cares about |
| Recent merged PRs | `gh pr list --state merged --limit 10 --json title,body,files | cat` (bash) | Contribution patterns — what files get touched together, what a typical PR looks like |
| PR review comments | `github-mcp-server-pull_request_read` on recent PRs | **Repeated review feedback = conventions that should be in copilot-instructions.md** |
| Releases | `gh release list --limit 5 | cat` (bash) | Release cadence, versioning scheme |
| GitHub Actions workflows | `github-mcp-server-actions_list` or read `.github/workflows/` | CI/CD setup, what runs on PRs |
| Branch protection | `github-mcp-server-list_branches` | Default branch, protection rules |
| Push permissions | `gh api repos/{owner}/{repo} --jq '.permissions.push' | cat` (bash) | Whether the user can push directly or needs to fork |

### 0c. Mine PR review comments for conventions

This is the **highest-value** GitHub-native insight. Look at the 5-10 most recent merged PRs.

*Why?*: If a maintainer leaves the same review comment on 5 different PRs, that's a convention waiting to be documented. Mining PR reviews turns reviewer fatigue into automated guidance.

1. Use `github-mcp-server-list_pull_requests` (state: closed, sort: updated) to find recent merged PRs
2. For each, use `github-mcp-server-pull_request_read` (method: get_review_comments) to read review threads
3. Look for **repeated patterns** — the same feedback given across multiple PRs becomes a convention:
   - "Please add tests for this" → add to test conventions
   - "Use X pattern instead of Y" → add to coding conventions
   - "Update the docs when you change this" → add to maintenance matrix
   - "Don't forget to update the changelog" → add to maintenance matrix

**If few or no review comments are found** (e.g., PRs are self-merged or auto-merged), expand the search to up to 20 merged PRs. If there are still no review patterns, note this in the findings: _"No PR review patterns found — consider adding conventions as the team grows."_ Never silently skip this section.

These mined conventions go directly into `copilot-instructions.md` — turning repeated human review feedback into automated AI guidance.

### 0d. Check community health gaps

GitHub's community health API tells you exactly what's missing. Map it to the assets this skill generates:

| GitHub says missing | Skill generates |
|-------------------|-----------------|
| No issue templates | `.github/ISSUE_TEMPLATE/` (Step 6) |
| No pull request template | `.github/PULL_REQUEST_TEMPLATE.md` (Step 6) |
| No CONTRIBUTING guide | README Contributing section (Step 7) |
| No CODE_OF_CONDUCT | Can suggest adding one |
| No license | Flag in the report |
| No README | Flag in the report |

---

## Step 1 — Analyze the codebase

GitHub context tells you *what* the repo is. Local analysis tells you *how* it works. You need both.

*Why?*: GitHub metadata covers languages, CI, and community health — but it can't tell you the test runner, the build commands, or the directory layout. Scan the local codebase for those deeper technical details.

Use built-in tools (glob, grep, view) combined with the GitHub context from Step 0.

### 1a. Detect languages and frameworks

Use glob to find manifest files. Read each one with view to extract details:

| Manifest | Language | What to extract |
|----------|----------|-----------------|
| `package.json` | JavaScript/TypeScript | dependencies, devDependencies, scripts (build, test, lint, typecheck), engines.node |
| `Cargo.toml` | Rust | workspace members, dependencies, build/test profile |
| `go.mod` | Go | module name, Go version |
| `pyproject.toml` or `requirements.txt` | Python | dependencies, build system, scripts, python version |
| `*.csproj` or `*.sln` | C# / .NET | target framework, package references, test SDK |
| `Gemfile` | Ruby | dependencies, ruby version |
| `pom.xml` or `build.gradle` | Java | dependencies, plugins, build tasks |

Also check for:
- **Lockfiles** — `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lockb`, `Cargo.lock`, `go.sum`, `poetry.lock`, `Pipfile.lock`
- **Runtime version files** — `.nvmrc`, `.node-version`, `.python-version`, `.tool-versions`, `.ruby-version`, `rust-toolchain.toml`
- **Monorepo markers** — `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json`, Cargo workspace, Go workspace. Also check for **large library monorepos**: Maven aggregator (`pom.xml` with `<modules>`), Python multi-package (`libs/` directory with multiple `pyproject.toml`), or Turborepo + Changesets (`turbo.json` + `.changeset/`).

  *Why?*: Large open-source libraries like LangChain organize code as multi-package monorepos — dozens of independently published packages under one repo. Treating them as a single package misses cross-package dependencies, per-package build commands, and module-specific conventions.
- **Notebooks** — `*.ipynb` files. If found, note the count and locations. Notebooks are common in course repos, data science projects, and tutorials.
- **VS Code extension markers** — check for `contributes` in root `package.json` (commands, themes, snippets, views, menus). If present, this is a **VS Code extension**, not a regular app. Also check for `vsce` or `@vscode/vsce` in devDependencies, and `vscode:prepublish` in scripts. Extensions come in three flavors:
  - **Functional extensions** — TypeScript code with activation events, commands, webpack/esbuild bundling, tests
  - **Theme extensions** — JSON theme files, no runtime code, published via `vsce`
  - **Snippet extensions** — JSON snippet definitions, language-scoped, content-driven not logic-driven

  *Why?*: VS Code extensions look like npm packages but have completely different conventions. The `package.json` IS the product spec — commands, menus, settings, keybindings. Treating them like a web app misses what matters.
- **Multi-app collections** — multiple independent apps in subdirectories (e.g., `angular/`, `react/`, `svelte/`), each with its own `package.json`, but **no workspace config** tying them together. This is different from a monorepo — there's no shared build or dependency graph. Each app builds and runs independently.

  *Why?*: Not every repo with multiple folders is a monorepo. Some are "collections" — the same concept implemented in different frameworks for comparison or learning. Don't invent workspace tooling where none exists.
- **Demo app patterns** — a frontend app + mock backend (`json-server`, `db.json`) + proxy config (`proxy.conf.json`, `vite.config.ts` proxy). Common in demo/tutorial repos. If detected, document the mock API setup in AGENTS.md so agents know to start both frontend and backend.

### 1a-ii. Detect course/tutorial repos

*Why?*: Course repos are fundamentally different from application repos. The "product" is markdown lessons and code samples — not a running application. Generating CI, setup steps, or a build pipeline for a course repo misses the point. Detecting this early shapes every later step.

Check for **multiple signals** — no single check is definitive:

1. **Numbered folders** — glob for top-level directories matching `NN-*` (e.g., `00-intro`, `01-setup`, `05-advanced`), `N-topic` (e.g., `1-Introduction`, `6-Data-Science-In-Wild`), `Chapter N`, `Module N`, or `Unit N`. 3+ matches is a strong signal.
2. **README content** — scan the root README for course/tutorial language: "lesson", "chapter", "module", "unit", "what you'll learn", "prerequisites", "course structure", "hands-on", "assignment", "quiz", "curriculum", "week". Multiple matches strengthen the signal.
3. **Repo description/topics** — check the GitHub description and topics (from Step 0b) for terms like "beginners", "course", "tutorial", "workshop", "learn", "curriculum", "lessons".
4. **Lesson structure** — check if numbered folders each contain a `README.md` (lesson content) and optionally `assignment.md`, `solution/`, `code/`, `quiz/`, or `notebook/` subdirectories.
5. **No primary application** — the repo has no root-level `package.json`, `Cargo.toml`, `go.mod`, or other manifest that would indicate a buildable application (individual lesson folders may have their own manifests for code samples).
6. **Devcontainer** — check for `.devcontainer/` directory. Common in course repos to provide a ready-to-go development environment. If present, credit it as a form of environment setup (similar to copilot-setup-steps.yml).

**A repo is a course if 3+ of these signals are present.** Record it in the findings table as `Repo type: course` with evidence.

**When a repo is a course, the following steps adapt:**
- **Step 4** (copilot-setup-steps.yml) — skip if a `.devcontainer/` exists (it serves the same purpose for courses). If no devcontainer and no build step, skip entirely.
- **Step 5** (CI workflow) — skip build/test CI. Suggest markdown validation (link checking, spell check) instead if not already present
- **Step 3** (copilot-instructions.md) — include lesson structure conventions: expected folder contents, naming patterns, how to add a new lesson. If lessons have quizzes or assignments, document the expected structure (e.g., each lesson needs `README.md` + `assignment.md` + `solution/`)
- **Step 2** (AGENTS.md) — "Adding a New Lesson" section instead of "Adding a New Feature". Include the lesson template (what files/folders each lesson should contain)
- **Report** — mark skipped assets as "N/A — course repo" instead of "Missing". Credit `.devcontainer/` in the "Nailed It" section if present.

### 1b. Detect test setup

- Identify test runner from dependencies (Jest, Vitest, Playwright, pytest, go test, cargo test, xUnit, NUnit, RSpec, etc.)
- Find test directories: `tests/`, `test/`, `__tests__/`, `spec/`, `e2e/`, `*_test.go`, `**/*.test.*`
- Extract test commands from scripts (npm test, pytest, cargo test, dotnet test, etc.)

### 1c. Detect CI/CD

- Check `.github/workflows/` — read each YAML file to determine triggers (`pull_request`, `push`), jobs, and steps
- Note whether PR checks already exist
- Check for other CI systems: `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/`, `azure-pipelines.yml`
- **Recognize community workflows** — workflows like `welcome-issue`, `stale`, `lock`, `greetings`, or `close-stale-issues` are valid automation, not build CI. Don't flag a repo as "missing CI" just because its only workflows are community management. Note them as "community workflows" in the findings.

*Why?*: Many course repos and open source projects have workflows for welcoming new contributors and managing stale issues. That's CI — just not build CI. Flagging it as "missing" is misleading.

### 1d. Check existing AI configuration

Check whether each of these exists:
- `AGENTS.md`
- `.github/copilot-instructions.md`
- `.github/workflows/copilot-setup-steps.yml`
- `.github/skills/` (any skill files)
- `.github/agents/` (any agent files)
- `.github/extensions/` (any extension files)
- `.devcontainer/` (devcontainer config — common in course repos and Codespaces-enabled projects. Credit as environment setup if present.)

**If `.github/agents/` or `.github/skills/` contain files, enumerate them.** List each agent/skill by name and note its description (from the file's frontmatter or first comment). These represent significant AI configuration that should be credited in the report. Include a count in the findings table (e.g., "6 custom Copilot skills, 2 custom agents").

**If multiple instruction files exist** (e.g., `AGENTS.md`, `.github/copilot-instructions.md`, `CLAUDE.md`, `.github/instructions/*.instructions.md`), read all of them and check for consistency:

1. **Duplicate content** — flag sections that appear in multiple files.

   *Why?*: Copy-pasted instructions drift apart over time. One authoritative source with references keeps everything in sync.

2. **Contradictory conventions** — flag conflicting guidance with the specific files and lines. Tabs vs. spaces, different test commands, mismatched naming rules — call out every conflict.

   *Why?*: An agent that reads "use tabs" in one file and "use spaces" in another will pick one at random. You want deterministic behavior.

3. **Stale references** — flag file paths, commands, or patterns that no longer exist in the repo. Cross-reference against the directory scan from Step 1h.

   *Why?*: An instruction file that references `src/utils/helpers.ts` when that file was deleted three months ago sends agents on a wild goose chase.

4. **Scope clarity** — every instruction file should declare what it covers. No `applyTo` pattern and no explicit scope statement? Flag it.

   *Why?*: Without clear scoping, agents don't know which file governs which part of the codebase. They guess — and they guess wrong.

Record consistency findings in the findings table with evidence showing which files conflict.

### 1e. Check repo configuration

Check whether each of these exists:
- `.github/CODEOWNERS` or `CODEOWNERS`
- `.github/dependabot.yml`
- `.github/ISSUE_TEMPLATE/` (any templates)
- `.github/PULL_REQUEST_TEMPLATE.md`
- `LICENSE` or `LICENSE.md`
- `README.md` with a `## Contributing` section (grep for it)

### 1f. Evaluate changelog

Check for a changelog and assess its health:

1. **Find the changelog** — look for `CHANGELOG.md` at the repo root. If it exists, read it. If it's a pointer file (e.g., "See the changelog in our docs"), follow the pointer and read the actual changelog content.
2. **Check alternative locations** — if no root changelog, check `docs/changelog/`, `docs/CHANGELOG.md`, GitHub Releases (via the GitHub MCP tools if available), or a docs site changelog page.
3. **Assess format** — is it Keep a Changelog format, a flat list, GitHub Releases only, or a docs-site page? Note the format.
4. **Assess freshness** — compare the most recent changelog entry against the latest git tag (`git tag --sort=-v:refname | head -1`). If the changelog is significantly behind the latest release, flag it as stale.
5. **Note non-standard locations** — if the changelog lives somewhere other than `CHANGELOG.md` at the root, this is acceptable but should be noted. The root file should at minimum link to the real location.

### 1g. Evaluate documentation

Check if the repo has documentation and assess its health:

1. **Find documentation** — check for `docs/`, `documentation/`, a docs config file (`docusaurus.config.js`, `mkdocs.yml`, `docs/index.html`, `.vitepress/`, `_config.yml`), or a wiki link.
2. **Detect framework** — if docs exist, identify the framework: Docsify, Docusaurus, MkDocs, VitePress, Jekyll, Hugo, or plain markdown.
3. **Check navigation** — does the docs site have a sidebar, table of contents, or nav config? (e.g., `_sidebar.md`, `sidebars.js`, `mkdocs.yml nav:`, `_config.yml`)
4. **Check deploy pipeline** — is there a GitHub Actions workflow that builds/deploys docs? (look in `.github/workflows/` for docs-related workflows)
5. **Check README linkage** — does the README link to the docs site or docs directory?
6. **Assess whether docs are needed** — if no docs exist, consider the project type. Libraries with public APIs, frameworks, and tools with complex configuration generally need documentation. Simple utilities or single-file projects may not.

### 1h. Scan directory structure

List the top-level directories and their immediate children (skip `node_modules`, `.git`, `dist`, `build`, `target`, `vendor`, `__pycache__`, `.venv`).

### 1i. Compile findings

Before proceeding, produce a structured summary combining GitHub context (Step 0) and codebase analysis (Step 1). Include file-path evidence for each finding:

| Category | Finding | Evidence (source) |
|----------|---------|-------------------|
| Repo | e.g., johnpapa/ai-ready | `git remote -v` |
| Description | e.g., "Copilot CLI skill..." | GitHub API / repo metadata |
| Topics | e.g., copilot, skills, ai-ready | GitHub API |
| Language | e.g., TypeScript (65%), Rust (30%) | GitHub API language breakdown |
| Multi-language | yes/no — if no single language exceeds 50%, flag as multi-language | GitHub API |
| Repo type | app / course / docs-only / VS Code extension / npm package / collection | Step 1a-ii detection |
| VS Code extension type | functional / theme / snippets (if applicable) | `package.json` contributes field |
| Notebooks | e.g., 12 `.ipynb` files in `lessons/` | glob for `*.ipynb` |
| Mock backend | e.g., json-server on port 3000 | `db.json`, proxy config |
| Framework | e.g., React, Phaser | `package.json` dependencies |
| Test runner | e.g., Vitest | `package.json` devDependencies |
| Test command | e.g., `npm test` | `package.json` scripts.test |
| Build command | e.g., `npm run build` | `package.json` scripts.build |
| Runtime version | e.g., Node 22 | `.nvmrc` or `package.json` engines |
| Package manager | e.g., pnpm | `pnpm-lock.yaml` exists |
| Contributors | e.g., 3 contributors | GitHub API |
| Team size | e.g., solo / small / large | Contributor count |
| PR CI exists | yes/no | `.github/workflows/` or GitHub Actions API |
| Community health | e.g., 71% | GitHub API community/profile |
| PR review patterns | e.g., "maintainer often asks for tests" | Mined from recent PR review comments |
| Release cadence | e.g., monthly, tagged releases | GitHub Releases API |
| AGENTS.md | exists / missing | repo root |
| copilot-instructions.md | exists / missing | `.github/` |
| Changelog | exists / pointer / missing | `CHANGELOG.md`, Releases |
| Changelog freshness | current / stale | latest entry vs latest git tag |
| Docs exist | yes / no | `docs/`, config file |
| Docs framework | Docsify / Docusaurus / etc. | config file path |
| Docs deploy pipeline | yes / no | workflow file path |
| README links to docs | yes / no | README.md link |
| Default branch | e.g., `main`, `dev`, `master` | `gh repo view --json defaultBranchRef` |
| Push access | yes / no | `gh api repos/{owner}/{repo} --jq '.permissions.push'` |
| Custom agents | e.g., 2 agents: migration guide, orchestrator | `.github/agents/` |
| Custom skills | e.g., 6 skills: bunit-test, component-dev, ... | `.github/skills/` |
| Devcontainer | yes/no | `.devcontainer/` |
| Monorepo | yes/no | workspace config file |
| Areas | e.g., frontend (React), backend (Express), shared (TypeScript) | workspace config paths |

**List which of the 12 assets are missing and need to be created.** Do NOT overwrite existing files — only create assets that don't exist yet.

**For existing AI-ready assets**, read their current contents and compare against your analysis. Flag drift in any of these dimensions:

| Asset | What to compare |
|-------|----------------|
| `AGENTS.md` | Repo structure still accurate? Build/test commands still correct? Tech stack changed? |
| `copilot-instructions.md` | New conventions from recent PR reviews? Maintenance matrix still covers current file relationships? |
| `copilot-setup-steps.yml` | Runtime versions match? Install/build commands still correct? New dependencies? |
| CI workflow | Build/test/lint commands still match the project? New tools added? |
| Issue templates | Still relevant to the project type? |
| README Contributing | Links still valid? Commands still correct? |

For each existing asset where you find drift, classify it as **"Could Be Better"** in the report with a specific suggestion (e.g., "AGENTS.md lists Node 18 but `.nvmrc` now says Node 22"). Do not silently skip existing files — always evaluate them.

### 1j. Detect monorepo areas

If a workspace config was found in Step 1a, read it to find package/project paths (e.g., `packages/*`, `apps/*`, `libs/*`). List each area — name, path glob, and primary stack — and note which areas have conventions that differ from root.

**For large library monorepos** (Maven aggregator, Python `libs/`, pnpm workspace with many packages):
- List each published package/module separately with its purpose (e.g., `langchain4j-core`, `langchain4j-open-ai`, `langchain4j-ollama`)
- Note the module taxonomy if one exists (core vs providers vs integrations vs experimental)
- Identify **cross-package dependencies** — which packages depend on which. Changes to core packages ripple to all dependents.
- Detect **release tooling** — Changesets (`.changeset/`), semantic-release, Maven release plugin, or manual versioning. Document in the maintenance matrix.
- Detect **conditional modules** — JDK-specific modules (`jdk21`), platform-specific builds, or optional integrations that only build under certain conditions.

*Why?*: A fix in `langchain4j-core` affects 30+ downstream modules. Without mapping cross-package dependencies, agents make changes to one package and miss the ripple effects.

---

## Step 2 — Generate AGENTS.md

If `AGENTS.md` does not already exist at the repository root, create it. All content must be derived from the analysis in Step 1 and from inspecting the actual repo.

If `AGENTS.md` already exists, read it and compare against the current analysis. Flag drift (e.g., outdated repo structure, stale build commands, missing sections) as a "Could Be Better" suggestion. **Do not overwrite** — suggest specific updates and let the user decide.

Sections to include (or verify):

- **Project Overview** — derived from README, package.json, pyproject.toml, Cargo.toml, or similar manifest files. **Never hardcode the project/package version** — reference the manifest file (e.g., "See `package.json` for the current version") so the file doesn't go stale. Runtime/tool versions may be included when derived from the repo (e.g., `.nvmrc`, `engines`, `.python-version`, CI/workflow files, or other manifests/config).

  *Why?*: Hardcoded versions go stale the moment someone bumps a number. A reference to the manifest is always current.
- **Repository Structure** — a directory tree showing the key folders and what they contain.
- **Tech Stack** — languages, frameworks, runtimes, and major dependencies.
- **Build & Run** — exact commands to install dependencies, build, and run the project locally.
- **Testing** — test runner, how to run tests, any special setup required (e.g., browser drivers, database fixtures).
- **Key Patterns and Conventions** — architectural patterns, naming conventions, module structure, and any patterns inferred from the codebase (e.g., "all API routes live in `src/routes/`", "scenes extend BaseScene").
- **CI/CD** — summary of existing CI workflows, what triggers them, and what they do.
- **Adding a New [Feature/Module]** — a step-by-step guide customized to the project type (e.g., "Adding a New Game" for a game project, "Adding a New API Endpoint" for a web API, "Adding a New Command" for a CLI tool). **Trace the full registration chain** — don't just list the obvious files. Follow imports to find enum definitions, type registries, index re-exports, and config declarations that must also be updated.

  *Why?*: The obvious files are easy. It's the hidden registration steps — the enum that must match, the index that must re-export, the config that must register — that trip up contributors. Trace the full chain.

  For example, if commands are registered in `extension.ts` but their IDs come from an enum in `models/enums.ts`, include both files.
- **Screen Size / Responsive Rules** — include this section only if the project is a frontend or UI project.
- **Common Pitfalls** — things that frequently trip up contributors (e.g., "run `npm run build:frontend` before tests", "version must be updated in three files"). **If the analysis found any pointer files or non-standard locations** (e.g., a root `CHANGELOG.md` that redirects to `docs/changelog/`), call this out explicitly as a pitfall so agents edit the real file, not the pointer.

---

## Step 3 — Generate .github/copilot-instructions.md

If `.github/copilot-instructions.md` does not already exist, create it.

If it already exists, read it and compare against the current analysis — especially new PR review patterns from Step 0c not yet captured as conventions, and maintenance matrix entries that no longer reflect the current file structure. Flag drift as "Could Be Better" suggestions.

Content to include (or verify):

- **Language-Specific Conventions** — coding style, idioms, and patterns for the project's primary language(s) (TypeScript, Python, Go, Rust, etc.), derived from the analysis. **If the repo is multi-language** (no single language exceeds 50%), create separate convention subsections for each language instead of one combined block. Each subsection should cover that language's idioms, style, and patterns independently.

  *Why?*: A repo with Python, TypeScript, and Java code needs three sets of conventions — not a blended soup. Agents working in the Python folder should see Python rules, not Java rules.
- **Notebook Conventions** — include only if `.ipynb` files were detected. Cover: clear cell outputs before committing, pin kernel/runtime version, keep cells focused (one concept per cell), use markdown cells for explanations.
- **Course/Lesson Conventions** — include only if the repo was detected as a course in Step 1a-ii. Cover: expected folder structure per lesson (e.g., `README.md` + `assignment.md` + `solution/` + `quiz/`), naming patterns for lesson folders (`NN-topic-name`), how to add a new lesson, what content each lesson README must include. If the course has quizzes, document the quiz format and where quiz files live. If the course has a progressive project (code built across lessons), document the dependency chain between lessons.
- **Framework Patterns** — how the project uses its framework(s) (React component patterns, Express middleware conventions, Django app structure, etc.).
- **Conventions Mined from PR Reviews** — if Step 0c found repeated review feedback, include those as explicit conventions. For example, if the maintainer frequently asks "add tests for new features," make that a rule. This is the highest-value section — it turns human review fatigue into automated AI guidance.
- **Test Conventions** — which test runner to use, naming patterns for test files, what to test (unit, integration, e2e), how to run tests.
- **Code Style Notes** — inferred from linter/formatter configs if present (ESLint, Prettier, Black, Ruff, rustfmt, etc.). Reference the config files rather than duplicating rules.
- **Asset and Content Rules** — include only if the project has static assets (images, sounds, fonts, etc.). Cover naming conventions, file formats, and where assets live.
- **Maintenance Matrix** — this is critical.

  *Why?*: This is the single most valuable section in copilot-instructions.md. Without it, every AI agent (and human contributor) has to rediscover which files need updating when something changes. That's exactly the knowledge that lives in a maintainer's head and never gets written down — until now.

  Define what must be updated when different parts of the codebase change:

  | Change Made | Files to Update |
  |---|---|
  | New feature added | List the specific files, configs, registries, docs, and tests that must be created or updated |
  | Existing feature modified | List what else must change (tests, docs, related modules) |
  | Shared/common code changed | List all consumers and dependents to check |
  | Build or tooling changed | List CI configs, Dockerfiles, setup scripts to update |
  | Project structure changed | List AGENTS.md, README, import paths, CI paths to update |

  Populate the matrix with **real file paths and real patterns** from the repo. **Trace import chains and registration patterns** — don't stop at the obvious top-level files. Follow imports to find enum definitions, type interfaces, index re-exports, config declarations, and other files in the dependency chain.

  For example, if a new command requires updating both `commands.ts` and an enum in `models/enums.ts`, include both. If a feature has a registration step in an index file, include that too.

  **If the analysis found pointer files or non-standard locations** (e.g., a changelog that lives in `docs/changelog/` instead of the root), use the real path in the matrix — never reference the pointer file.

### Monorepo: Area-scoped instructions

If the repo is a monorepo with areas that have different stacks or conventions (detected in Step 1j), create `.github/instructions/{area-name}.instructions.md` for each distinct area:

```yaml
---
applyTo: "{area-path}/**"
---
```

Include only what differs from root conventions — framework patterns, test setup, or build commands specific to that area. Skip areas that share the same stack as the root.

---

## Step 4 — Generate .github/workflows/copilot-setup-steps.yml

If `.github/workflows/copilot-setup-steps.yml` does not already exist, create it.

If it already exists, verify that runtime versions, install commands, and build steps still match the current project. Flag mismatches as "Could Be Better" suggestions.

Steps to include (or verify):

- Check out the repository
- Set up the language runtime (Node.js, Python, Go, Rust, .NET, etc.) at the correct version
- Install project dependencies (npm install, pip install, go mod download, cargo fetch, etc.)
- Install test dependencies if separate (e.g., Playwright browsers, test fixtures)
- Build the project (if a build step is required before tests can run)

Base every step on the analysis results from Step 1. Use the exact commands and versions the project actually uses.

**Derive from existing CI when possible.** If the repo already has a CI workflow (`.github/workflows/`), use it as the source of truth for SDK versions, restore commands, and build steps. Mirror CI — don't invent new commands.

*Why?*: The CI workflow already has the correct SDK versions, restore commands, and build steps. Don't reinvent them — mirror them.

**Multi-target frameworks (.NET):** If `.csproj` files use `<TargetFrameworks>` (plural) with multiple targets (e.g., `net8.0;net9.0;net10.0`), the setup steps must install **all** required SDK versions. Check every `.csproj` for `TargetFramework` and `TargetFrameworks` properties and collect the full set of versions needed.

---

## Step 4b — Generate .vscode/mcp.json

*Why?*: MCP servers give AI agents access to your project's tools and data — databases, APIs, file systems. Without this config, the agent can read your code but can't talk to the services your code depends on.

If `.vscode/mcp.json` does not already exist, generate it based on the dependencies and tools detected in Step 1.

If it already exists, read it and verify the servers still match the project's current dependencies. Flag mismatches as "Could Be Better" suggestions.

### What to include

Analyze the repo's tech stack from Step 1 — languages, frameworks, databases, APIs, cloud services, and tools — and recommend **any MCP server that would give AI agents useful context** for this project.

*Why?*: The MCP ecosystem is growing fast. Don't limit recommendations to a fixed list. If the project uses Stripe, recommend a Stripe MCP server. If it uses Kubernetes, recommend a K8s MCP server. Match the repo's actual stack to what's available.

Common patterns to look for:
- **Databases** — PostgreSQL, MySQL, SQLite, MongoDB, Redis, DynamoDB, etc.
- **APIs and services** — GitHub, Stripe, Slack, Twilio, SendGrid, etc.
- **Cloud platforms** — AWS, Azure, GCP SDKs and CLIs
- **Browser automation** — Puppeteer, Playwright, Selenium
- **File and search** — filesystem access, Elasticsearch, Algolia
- **DevOps tools** — Docker, Kubernetes, Terraform

For each detected dependency, search for a matching MCP server package (typically `@modelcontextprotocol/server-*` or community packages). If no MCP server exists for a dependency, skip it — don't invent one.

Only include servers the project actually needs. Do not speculatively add servers.

*Why?*: Speculative MCP servers add noise and may prompt for credentials the user doesn't have. Only connect what the project actually needs.

### Output format

Generate `.vscode/mcp.json` with the `servers` object:

```json
{
  "servers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

Use environment variable references (`${VAR}`) for connection strings and secrets. Never hardcode values.

*Why?*: Hardcoded connection strings are a security risk and break across environments. Environment variable references let each developer use their own credentials.

Note which servers were added and why in the "What I Did" report section. If no relevant dependencies are detected, skip this step and mark the asset as "N/A — no MCP-compatible dependencies detected" in the report.

---

## Step 5 — Generate CI workflow

Check `.github/workflows/` for any existing workflow that triggers on `pull_request`. If none exists, create `.github/workflows/ci.yml` with:

- **Triggers:** `pull_request` (all branches) and `push` to the **default branch** (detected in Step 0b — do not hardcode `main`), with **path filters** so CI only runs when code actually changes.

  *Why?*: Without path filters, every docs typo fix triggers a full build. Path filters keep CI fast and focused on real code changes.

  Use `paths-ignore` to skip documentation-only and config-only changes:
  ```yaml
  on:
    pull_request:
      paths-ignore:
        - '**.md'
        - 'docs/**'
        - '.github/ISSUE_TEMPLATE/**'
        - 'images/**'
        - 'LICENSE'
        - '.gitignore'
    push:
      branches: [main]
      paths-ignore:
        - '**.md'
        - 'docs/**'
        - '.github/ISSUE_TEMPLATE/**'
        - 'images/**'
        - 'LICENSE'
        - '.gitignore'
  ```
  Customize the `paths-ignore` list based on the repo's actual structure. If the project has other non-code directories (e.g., `assets/`, `design/`, `samples/`), add those too. If the project has code in markdown (e.g., a docs site with executable code blocks), **do not** ignore markdown files.
- **Jobs:** A build-and-test job that:
  - Checks out the code
  - Sets up the correct language runtime
  - Installs dependencies
  - Runs linting (if a lint command exists)
  - Builds the project
  - Runs the test suite
- Match the project's actual build/test toolchain from the analysis

**Never modify or replace existing workflow files.**

---

## Step 6 — Generate issue templates

If `.github/ISSUE_TEMPLATE/` does not already exist, create:

- **Bug Report** (`bug-report.yml`) — a structured issue form with fields relevant to the project type (steps to reproduce, expected vs. actual behavior, environment details, screenshots if UI project).
- **Feature Request** (`feature-request.yml`) — a structured form for proposing new features (description, motivation, alternatives considered).
- **Project-Specific Templates** — if the project type warrants it, add a third template (e.g., "New Game Proposal" for a game project, "New Integration" for a platform with plugins, "API Change" for an API-heavy project).

Use the **YAML issue form format** (not the older markdown template format).

*Why?*: YAML forms give contributors structured fields instead of a blank text box. Structured reports are easier to triage and less likely to be missing critical info.

**If old-format markdown templates exist** (`.md` files in `.github/ISSUE_TEMPLATE/` that are not `config.yml`), note them in the report as a "Could Be Better" suggestion: _"Consider converting `{filename}` from the old markdown template format to the newer YAML form format for consistency and better user experience."_ Do not delete or overwrite the old templates.

### PR template

If `.github/PULL_REQUEST_TEMPLATE.md` does not already exist, create it with:

- **Description** — a prompt asking what the PR does (one or two sentences)
- **Changes** — a bulleted list of files changed and why
- **How to Test** — steps a reviewer can follow to verify the changes work, customized to the project's test commands from the analysis
- **Checklist** — items relevant to the project (e.g., "Tests pass", "Docs updated", "Lint clean"). Derive checklist items from the maintenance matrix — if the matrix says "update X when Y changes", add a checklist item for it.

Keep it short and useful — a PR template that's too long gets ignored.

---

## Step 7 — Update README Contributing section

If `README.md` exists at the repo root but does not contain a "Contributing" section (search for `## Contributing` or `# Contributing`):

- **If a standalone `CONTRIBUTING.md` exists**, add a short `## Contributing` section in the README that summarizes the key points and links to it:
  ```
  ## Contributing
  See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on how to contribute, including setup, testing, and PR conventions.
  ```
  Do not duplicate the content — link to it.

  *Why?*: Duplicated content drifts apart. One source of truth with a link from README keeps things in sync.
- **If no `CONTRIBUTING.md` exists**, add a `## Contributing` section near the end of the README with:
  - A suggestion to use Copilot CLI for contributions, with an example prompt in a code block:
    ```
    Add a new [feature/command/endpoint] called X that does Y
    ```
  - How to fork, branch, and submit a PR
  - How to run tests locally (exact commands)
  - A link to `AGENTS.md` for the full contributor guide
- **Never rewrite or restructure the rest of the README** — only append the Contributing section.

If a Contributing section already exists, skip this step.

---

## Step 8 — Verify maintenance matrix

The maintenance matrix should already be part of `copilot-instructions.md` (generated in Step 3). Verify that it covers:

- What files reference each other (e.g., a game registry that must be updated when a new scene is added)
- What must be updated when different parts of the codebase change
- Cross-cutting concerns (e.g., version numbers in multiple files, route registrations, module re-exports)

**Trace actual dependency graphs**, not just top-level files.

*Why?*: The obvious files are easy. It's the hidden dependencies — the enum that must match, the index that must re-export, the config that must register — that trip up contributors. Trace the full chain.

Language-specific techniques:

| Language | How to trace |
|----------|-------------|
| .NET | Grep for `<ProjectReference>` in `.csproj` files; check `ServiceCollectionExtensions` or `Program.cs` for DI registration |
| Node.js/TS | Follow `import`/`require` chains; check index re-export files (`index.ts`) |
| Go | Follow `import` statements; check `main.go` and wire/DI setup |
| Python | Follow `import` statements; check `__init__.py` re-exports and entry points |
| Rust | Follow `mod` and `use` declarations; check `lib.rs` and `main.rs` |

If the matrix is missing or incomplete, suggest specific additions in the report. If `copilot-instructions.md` was just created in Step 3, add the matrix directly. If it already existed before this run, report the gaps as "Could Be Better" and let the user decide.

---

## Step 9 — Evaluate and improve changelog

Based on the changelog analysis from Step 1f:

### If no changelog exists at all:
- Create `CHANGELOG.md` at the repo root with a Keep a Changelog format header and an initial `[Unreleased]` section
- If the project has git tags or GitHub Releases, populate it with entries from the most recent releases

### If a changelog exists but is a pointer file:
- This is acceptable — some projects maintain their changelog in a docs site. **Do not overwrite the pointer file.**
- Verify the pointer target actually exists and contains real changelog content
- If the pointer target is missing or empty, flag this in the summary as needing attention
- If the root `CHANGELOG.md` doesn't clearly link to the real location, suggest adding a direct link

### If a changelog exists but is stale:
- Flag this in the summary with the date of the last entry vs. the latest release/tag
- Suggest the maintainer update it, but **do not auto-generate changelog entries** — the maintainer knows what changed

### Changelog in AGENTS.md:
- If the project maintains a changelog in a non-standard location, document this in `AGENTS.md` so AI agents know where to find and update it

---

## Step 10 — Evaluate and improve documentation

Based on the documentation analysis from Step 1g:

### If documentation exists:
- **Include in AGENTS.md** — add a "Documentation" section noting the framework, location, and how to build/preview docs locally
- **Include in copilot-instructions.md** — add docs conventions (where docs live, naming, how to update them when code changes)
- **Add to maintenance matrix** — ensure the matrix includes rules like "when a feature is added, update the docs"
- **Check for docs CI** — if docs have a deploy pipeline but no validation (link checking, build verification), suggest adding one to CI

### If no documentation exists:
- **Assess whether docs are needed** based on the project type:
  - Libraries, frameworks, CLIs, and plugins → docs recommended
  - Simple utilities, scripts, or single-purpose tools → README may be sufficient
- If docs are recommended, note this in the summary as a suggestion — **do not generate a docs site**. Docs frameworks are a preference decision for the maintainer.

### Documentation in AGENTS.md:
- Always include a "Documentation" section in `AGENTS.md` that describes:
  - Where docs live (if they exist)
  - What framework is used (if any)
  - How to preview docs locally
  - Or explicitly state "This project does not have a docs site — documentation is in README.md"

---

## Step 11 — Display the AI-Readiness Report

After completing all steps, you MUST display the AI-Readiness Report using the **exact format** below. Fill in the placeholders from your analysis. Do not skip, abbreviate, or restructure this report.

Calculate the score by counting how many assets have **Nailed It** status. Determine the maturity level from the count. Build the progress bar using 🟩 for nailed, 🟨 for could-be-better, and ⬜ for missing — always 12 squares.

Display this report:

```
🎯 **AI-Readiness Report**

Your repo is about to get a whole lot easier to contribute to — and
a whole lot faster to review. AI agents will know your conventions,
follow your patterns, and deliver PRs that are ready to merge.

**{repo-name}**

---

📊 **Your Repo Today** · {medal} **{level-name}** · {progress-bar} · {nailed} of 12 nailed
{languages} · {frameworks} · {test-runner} ({test-count}) · `{build-command}`

🤖 **Existing AI Config (detected)**

_Include this section only if the repo already has AI configuration (copilot-instructions.md, custom agents, custom skills). Omit it entirely if there is no pre-existing AI config._

| Asset | Detail |
|-------|--------|
| {asset-name} | {detail — e.g., "542 lines — components, testing, shims, docs"} |
| {.github/agents/} | {count} agents: {names} |
| {.github/skills/} | {count} skills: {names} |

⚠️ **Instruction Consistency**

_Show this section when consistency issues are found — skip it when everything lines up._

| Issue | Files | Detail |
|-------|-------|--------|
| {issue-type} | {file1} ↔ {file2} | {specific contradiction or duplication} |

✅ **Nailed It ({count})**

| Asset | Detail |
|-------|--------|
| {asset-name} | {one-line detail} |
| ... | ... |

💡 **Could Be Better ({count})**

| Asset | Suggestion |
|-------|-----------|
| {asset-name} | {suggestion} |
| ... | ... |

_Why these matter:_ {brief explanation of why the could-be-better items are worth improving}

⭕ **Missing ({count})**

| Asset | Why it matters |
|-------|---------------|
| {asset-name} | {why it matters} |
| ... | ... |

_Why these matter:_ {brief explanation of what the missing items cost the repo}

---

🛠️ **What I'd Like To Do** — proposed changes to close the gaps:

| Action | Detail |
|--------|--------|
| ➕ Create | `{filename}` — {what it will contain} |
| 🔍 Audit | `{filename}` — {what drifted and suggested fix} |
| ⏭️ Skip | `{filename}` — skipped (user requested) |
| 💬 Suggest | {suggestion} |
| ✅ Skip | {count} files already in great shape |

_For monorepos: list each `.github/instructions/{area}.instructions.md` file created as a separate ➕ Create row._

---

🏆 **If You Accept** · {after-progress-bar} · {after-nailed} of 12 nailed → {after-medal} **{after-level}**

🤖 AI Context        {5 status indicators}
🔧 Dev Workflow      {4 status indicators}
📖 Onboarding        {3 status indicators}

---

🚀 **What's Next?**

👉 **Create the PR now** — just say:
```
create a branch and open a PR with these changes
```

👉 **Tweak first** — tell me what to change:
```
update the AGENTS.md to include more detail about the command registration pattern
```

👉 **Share the report** — want a visual version for your team?
```
generate an HTML report I can share
```

👉 **Skip for now** — no worries, the analysis is done. Come back anytime and say `make this repo ai-ready` to pick up where you left off.
```

### HTML report (optional)

*Why?*: Terminal reports are great for the developer running the skill. But when you need to share results with a manager, post to a wiki, or attach to an email — you need something visual.

If the user asks for an HTML report (e.g., "generate a report I can share", "make an HTML report"), generate a self-contained `ai-ready-report.html` in the repo root.

The HTML report mirrors the terminal summary — same sections, same data, same structure:

1. **Header** — repo name, maturity level with emoji medal (🥉🥈🥇🏆), weighted score percentage, progress bar, generation date
2. **Tech profile** — languages, frameworks, test runner, build command
3. **Existing AI config** — if detected (copilot-instructions.md, custom agents/skills)
4. **Instruction consistency** — if issues found
5. **Asset status** — three groups: ✅ Nailed It, 💡 Could Be Better, ⭕ Missing — with one-line details per asset
6. **What was generated** — action table (➕ Create, 🔍 Audit, ⏭️ Skip, 💬 Suggest)
7. **Updated score** — before/after with maturity level change
8. **What to do next** — remaining recommendations

The file must be self-contained (inline CSS, no external dependencies) and shareable — one file you can open in any browser or drop into an email. Use green/amber/gray status colors, system fonts, and a responsive layout. Keep it simple — this is a summary, not a dashboard.

Generate the HTML report only when the user asks for it. The terminal output is always the default.

After displaying the report, handle the badge and PR in this order:

### 11a. Add AI-Ready badge

Check if the README already contains an `AI--Ready` badge. If it does not, **automatically** insert this badge at the top of the README, after any existing title or badge row — do not ask, just add it:

```markdown
[![AI Ready](https://img.shields.io/badge/AI--Ready-yes-brightgreen?style=flat)](https://github.com/johnpapa/ai-ready)
```

The badge is a static Shields.io image with zero dependencies. It links back to the ai-ready repo so others can discover it. Include this in the "What I Did" section of the report as a `➕ Create` action.

### 11b. Offer to create the PR

After displaying the report and handling the badge, **ask the user** if they want to create a branch and open a PR. Do not tell them to type a command — ask them directly:

_"Would you like me to create a branch and open a PR with these changes?"_

If the user agrees:

1. **Check push permissions** from Step 0b.
2. **If the user has push access**: create a feature branch (e.g., `feat/ai-ready-config`), commit all new/modified files (including the badge), push, and open a PR targeting the **default branch** (detected in Step 0b — never assume `main`).
3. **If the user does NOT have push access**: use a fork-based flow automatically — fork the repo (`gh repo fork --clone=false`), add the fork as a remote, push the branch to the fork, then open a cross-fork PR (`gh pr create --head {user}:feat/ai-ready-config`). Handle it end-to-end — never ask the user to figure out the fork workflow.

Include a summary of what was added and the before/after score in the PR body. If the user declines, end the session gracefully.

**Always add a report comment to the PR.** After creating the PR, post a comment with a condensed version of the AI-Readiness Report:

```
## 🎯 AI-Readiness Report

**{repo-name}**

**Before:** {before-medal} **{before-level}** · {before-nailed} of 12 nailed
**After this PR:** {after-medal} **{after-level}** · {after-nailed} of 12 nailed

🤖 AI Context        {status indicators}
🔧 Dev Workflow      {status indicators}
📖 Onboarding        {status indicators}

| Action | File |
|--------|------|
| ➕ Create | `{filename}` |
| ... | ... |

Generated by [ai-ready](https://github.com/johnpapa/ai-ready)
```

*Why?*: The PR body is written once, but the report comment is what reviewers see first. A consistent, scannable summary makes it easy to understand the impact at a glance.

Rules for filling in the template:

- **Nailed It** = asset exists and is well-customized to the repo
- **Could Be Better** = asset exists but has gaps or could be enhanced
- **Missing** = asset does not exist and should be created
- If a section has 0 items (e.g., nothing missing), omit that section entirely
- The tech profile table should only include rows that apply (e.g., skip "Frameworks" if none detected)
- Keep each detail to one short line — no multi-line descriptions
- The "What I Did" section should list every file that was created, suggested, or skipped
- **Show an updated progress bar** after the "What I Did" section — recount nailed assets (counting all created files as now "Nailed It"), determine the new medal, and show the category breakdown. This shows the user the improvement visually (e.g., going from 🥈 On Track · 🟩🟩🟩🟩🟩🟨⬜⬜⬜⬜⬜⬜ · 5 of 12 → 🏆 AI-Ready · 🟩🟩🟩🟩🟩🟩🟩🟩🟩🟩🟩🟩 · 12 of 12)
- The "What To Do Next" section should include only the bullet points that are relevant — e.g., if no files were created, skip "review generated files" and instead say something like "Your repo is already AI-ready — nice work!"

---

## Important Rules

- **NEVER open a pager** — append `| cat` to every `gh api`, `gh pr list`, `gh release list`, `gh issue list`, `git log`, `git diff`, and any other `gh` or `git` command that might paginate. Use `git --no-pager` for git commands. A pager will hang the session.

- **NEVER overwrite existing files** — only create assets that are missing.
- **ALWAYS customize to the repo's actual language, framework, and patterns** — never produce generic boilerplate.
- **Self-consistency — every generated file must follow the conventions you establish.** The skill creates `copilot-instructions.md` with conventions, then creates other files (CI workflows, issue templates, AGENTS.md, setup steps) in the same PR. Copilot code review should find **zero issues** in that PR.

  Before finalizing files, cross-check: Does the CI workflow follow the YAML style you documented? Does AGENTS.md follow the markdown conventions you set? Do issue templates match the patterns you prescribed? If any generated file contradicts the conventions in `copilot-instructions.md`, fix it before creating it.

  *Why?*: The skill's own PR is the first test of its output. If Copilot code review finds issues in the PR that created the conventions, the conventions aren't worth much.
- **GitHub-native by default** — you are almost certainly in a GitHub repo. Use GitHub MCP tools and `gh` CLI to auto-discover repo metadata, community health, PR patterns, and contribution history. Never ask the user to explain what their repo is or what tools they use — discover it automatically.

  If GitHub tools are unavailable, fall back to local git + filesystem analysis.
- **Mine PR reviews for conventions** — the maintainer's repeated review feedback is the most valuable source of conventions. Turn it into `copilot-instructions.md` rules so AI stops making the same mistakes.
- **Be specific and actionable** — include real file paths, real commands, and real patterns from the repo. Never produce generic advice.
- **Respect existing AI config** — if the repo already has configuration, treat it as authoritative and fill in the gaps.
- **Generate asset/content rules only if the project has assets** (images, sounds, fonts, models, etc.).
- **Use the `create` tool to write files** — never use `edit` to create a new file from scratch.
- **Run the full analysis first (Steps 0 and 1)** — never guess at the repo's structure or toolchain. Every generated asset must be based on evidence from the analysis.
- **ALWAYS display the AI-Readiness Report at the end** — use the exact format from the summary step. This is the user-facing output. Never skip, abbreviate, or restructure it.
- **NEVER use markdown headings (`#`, `##`, `###`) in your output to the user** — headings render in red/colored text in most terminals. Use **bold text** with emojis instead (e.g., `✅ **Nailed It (9)**`). This applies to the AI-Readiness Report and all other user-facing output during the skill execution.

---

## Training Repos

This skill's heuristics — especially course detection, notebook handling, and multi-language support — were trained and validated against these repos. Use them for regression testing when making changes to the skill.

**Course/Tutorial repos:**
- `github/copilot-cli-for-beginners` — Copilot CLI course (markdown + Python)
- `microsoft/ai-agents-for-beginners` — AI agents course (markdown + notebooks + Python/C#)
- `microsoft/generative-ai-for-beginners` — GenAI course (markdown + notebooks + Python/JS/TS)
- `microsoft/mcp-for-beginners` — MCP tutorial (markdown + TS/Python/Java/C#)
- `microsoft/langchainjs-for-beginners` — LangChain.js course (markdown + TypeScript)
- `microsoft/langchain-for-beginners` — LangChain course (markdown + Python)
- `microsoft/langchain4j-for-beginners` — LangChain4j course (markdown + Java)
- `microsoft/ML-For-Beginners` — Machine Learning course (markdown + notebooks + Python)
- `microsoft/Web-Dev-For-Beginners` — Web development course (markdown + JS/HTML/CSS)
- `microsoft/AI-For-Beginners` — AI course (markdown + notebooks + Python)
- `microsoft/Data-Science-For-Beginners` — Data science course (markdown + notebooks + Python)
- `microsoft/IoT-For-Beginners` — IoT course (markdown + hardware samples)
- `microsoft/Generative-AI-for-beginners-dotnet` — GenAI .NET course (markdown + C#)
- `microsoft/generative-ai-for-beginners-java` — GenAI Java course (markdown + Java)
- `microsoft/AZD-for-beginners` — Azure Developer CLI tutorial (markdown + CLI examples)
- `microsoft/edgeai-for-beginners` — Edge AI course (markdown + sample apps)
- `microsoft/xr-development-for-beginners` — XR/Unity course (markdown + Unity/C#)

**Application repos:**
- `johnpapa/vscode-peacock` — VS Code functional extension (TypeScript, Mocha tests)
- `johnpapa/shopathome` — Multi-framework shopping app (Angular 21, React 19, Svelte 5, Vue 3.5, Fastify 5, Azure Functions v4)
- `johnpapa/angular-styleguide` — Documentation-only style guide (markdown)
- `johnpapa/heroes-angular` — Standard Angular SPA with json-server backend, Cypress, proxy config
- `johnpapa/heroes-vue` — Vue SPA with separate API package, not a monorepo
- `johnpapa/heroes-react` — React SPA (CRA-era) with json-server, proxy, Docker, env files

**npm packages:**
- `johnpapa/lite-server` — Small CLI package (JS, Mocha/Istanbul tests, bin/ entry point)

**Multi-app collections:**
- `johnpapa/hello-worlds` — Angular/React/Svelte/Vue demos, independent apps, no workspace
- `johnpapa/http-interceptors` — Same concept in Angular + Svelte, comparison monorepo

**VS Code extension variants:**
- `johnpapa/vscode-cloak` — Functional extension (TypeScript, webpack, commands + settings)
- `johnpapa/vscode-winteriscoming` — Theme extension (JSON theme files, no runtime code)
- `johnpapa/vscode-angular-snippets` — Snippets extension (JSON snippets, language-scoped, devcontainer)

**Large open-source library monorepos:**
- `langchain-ai/langchain` — Python multi-package monorepo (`libs/*`), pyproject.toml per package, AGENTS.md + CLAUDE.md
- `langchain-ai/langchainjs` — TypeScript monorepo (pnpm + Turborepo + Changesets), workspace packages under `libs/`
- `langchain4j/langchain4j` — Java Maven aggregator with 30+ modules, JDK-conditional builds, Spotless formatting

**Real-world field tests:**
- `FritzAndFriends/BlazorWebFormsComponents` — .NET multi-target library (Blazor, C#)
