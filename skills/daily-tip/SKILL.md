---
name: daily-tip
description: Show a daily tip about making repositories AI-ready. USE THIS SKILL when the user asks for a "daily tip", "tip of the day", "show me a tip", "ai-ready tip", or when triggered by a sessionStart hook. Picks one tip per day based on the current date so users see a fresh tip each day.
---

# Daily Tip

When invoked, display **one** tip from the list below. Pick the tip using today's date so that users see a different tip each day but the same tip if they ask again on the same day.

## How to pick the tip

1. Get today's day-of-year (1–365).
2. Use `(day_of_year % total_tips)` to select the tip index.
3. Display the selected tip using the format below.

## Display format

```
💡 Daily AI-Ready Tip (#N of TOTAL)

TIP_TEXT

---
Type "make this repo ai-ready" to get started.
```

Replace `#N` with the 1-based tip number and `TOTAL` with the total number of tips.

## Tips

1. **Add `AGENTS.md` to your repo root** — This is the single most impactful file for AI agents. It tells them your tech stack, build commands, test commands, and repo structure so they don't have to guess.

2. **Put build and test commands in `AGENTS.md`** — AI agents can't contribute if they can't verify their work. Include exact commands like `npm test` or `cargo build` so agents can run them without asking.

3. **Add `.github/copilot-instructions.md`** — This file teaches Copilot your coding conventions: naming patterns, error handling style, preferred libraries. Without it, Copilot generates generic code that doesn't match your project.

4. **Mine your PR reviews for repeated feedback** — If you keep leaving the same review comments ("add tests", "update the changelog", "use our error wrapper"), turn those into conventions in `copilot-instructions.md`. That's the whole point — stop repeating yourself.

5. **Create a maintenance matrix** — Document which files must be updated together. When `schema.prisma` changes, do migrations need updating? When `routes.ts` changes, do tests need updating? AI agents follow these rules — humans forget them.

6. **Add issue templates** — Bug reports and feature requests with structured fields give AI agents enough context to start working immediately. A blank issue is a blank canvas — and AI agents paint poorly on blank canvases.

7. **Add a PR template with a checklist** — A PR template that lists "did you add tests?", "did you update docs?", "did you run lint?" catches mistakes before review. AI agents follow checklists literally — which is exactly what you want.

8. **Set up CI that runs on PRs** — If there's no CI, there's no automated safety net. AI agents generate code faster than humans, but without CI, bad code lands faster too. Even a basic lint + test workflow helps.

9. **Don't skip `dependabot.yml`** — Automated dependency updates keep your repo secure and reduce maintenance burden. It's one YAML file that saves hours of manual version bumping.

10. **Use path-specific instructions for monorepos** — If your repo has multiple packages or apps, use `.github/instructions/**/*.instructions.md` with `applyTo` glob patterns so Copilot gets the right conventions for each area.

11. **Keep your `AGENTS.md` current** — A stale `AGENTS.md` is worse than none. If your build command changed from `npm run build` to `turbo build`, agents will fail silently with the old command. Re-run the ai-ready skill periodically to catch drift.

12. **Add a `copilot-setup-steps.yml` for cloud agents** — GitHub's Copilot coding agent runs in a container. Without setup steps, it can't install dependencies or run your build. This file is the cloud agent's bootstrap script.

13. **Write conventions, not rules** — Instead of "always use semicolons", write "we use semicolons because our linter enforces them via `@company/eslint-config`". The _why_ helps AI agents make good judgment calls in edge cases.

14. **Score your repo** — Run `how ai-ready is this repo?` to get a readiness report without changing any files. It shows you exactly what's missing and what it would take to reach 🏆 AI-Ready status.

15. **Existing config counts** — Already have `AGENTS.md` or `copilot-instructions.md`? The ai-ready skill won't overwrite them. It audits what you have against your current codebase and suggests improvements where things have drifted.

16. **Community workflows aren't CI** — If your repo only has `welcome.yml` and `stale.yml`, that's community automation, not build CI. The ai-ready skill recognizes the difference and won't give you false credit for CI coverage.

17. **Every file should earn its place** — Don't generate boilerplate for the sake of completeness. A `CONTRIBUTING.md` that says "please follow our conventions" without listing them is noise. The ai-ready skill generates files with real, repo-specific content.

18. **Use the self-consistency rule** — Generated files should follow the same conventions they define. If your `copilot-instructions.md` says "use single quotes", the generated code should use single quotes. Copilot code review will catch violations.

19. **Re-run after major changes** — Added a new package? Switched from Jest to Vitest? Migrated to a monorepo? Re-run the ai-ready skill to update your AI config. Drift is the enemy of good AI contributions.

20. **Start with a report, then generate** — Not sure if you need this? Run `score this repo` first. If you're already at 🥇 Solid, maybe you just need to fill a gap or two. If you're at 🥉 Getting Started, the full skill run will transform your repo.
