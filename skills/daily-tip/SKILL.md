---
name: daily-tip
description: Show a daily Copilot tip. USE THIS SKILL when the user asks for a "daily tip", "tip of the day", "show me a tip", "copilot tip", or when triggered by a sessionStart hook. Shows a fresh tip each day — first checks for a /chronicle-generated tip, then falls back to the built-in list.
---

# Daily Tip

When invoked, display **one** tip. A fresh tip is generated each day via `/chronicle` and cached locally. If no cached tip is available, fall back to the built-in list below.

## How to pick the tip

1. Check if `~/.copilot/tip-of-the-day.txt` exists and `~/.copilot/tip-of-the-day-date.txt` contains today's date (`YYYY-MM-DD`). If both are true, display the cached tip.
2. Otherwise, fall back to the built-in list: get today's day-of-year (1–365) and use `(day_of_year % total_tips)` to select the tip index.
3. Display the selected tip using the format below.

## Display format

```
💡 Daily Copilot Tip

TIP_TEXT
```

## Tips

1. **Use `@` to mention files in your prompt** — Type `@` followed by a relative path to include a file's contents as context. Copilot reads it so you don't have to paste code into your prompt.

2. **Use plan mode before coding** — Press `Shift+Tab` to enter plan mode. Copilot will outline an approach before writing code, so you can steer the direction early.

3. **Use `/compact` to free up context** — Running low on context window? The `/compact` command summarizes your conversation history so you can keep working without starting over.

4. **Mention issues and PRs with `#`** — Type `#` followed by a number to pull in issue or PR context. Copilot reads the title, body, and comments so it understands what you're working on.

5. **Run shell commands with `!`** — Prefix any command with `!` to run it directly without a model call. Great for quick `git status`, `ls`, or `npm test` checks mid-conversation.

6. **Use `/diff` to review your changes** — Before committing, run `/diff` to see everything Copilot changed. It's a quick sanity check that catches surprises.

7. **Add custom instructions in `.github/copilot-instructions.md`** — Teach Copilot your project's conventions once and every prompt benefits. Naming patterns, error handling, preferred libraries — write it down so Copilot stops guessing.

8. **Use `/ask` for side questions** — Need a quick answer without polluting your conversation history? `/ask` lets you ask a one-off question that doesn't add to the context window.

9. **Delegate to the cloud with `/delegate`** — Hand off your current session to GitHub's Copilot coding agent. It creates a PR from your conversation, so you can move on to other work.

10. **Create skills for repeatable workflows** — If you find yourself giving Copilot the same multi-step instructions repeatedly, turn them into a skill in `~/.copilot/skills/` or `.github/skills/`. Invoke with `/skill-name`.

11. **Use `/resume` to pick up where you left off** — Closed a session by accident? `/resume` lets you select and continue a previous session with all its context intact.

12. **Add `AGENTS.md` to your repo** — This file tells Copilot your tech stack, build commands, test commands, and repo structure. Without it, Copilot has to guess — and it guesses wrong more than you'd like.

13. **Press `Ctrl+T` to toggle reasoning** — See how Copilot thinks through your problem. Useful for understanding why it chose a particular approach, or for debugging when it goes sideways.

14. **Use autopilot mode for multi-step tasks** — Press `Shift+Tab` to cycle to autopilot mode. Copilot will keep working through a task without stopping to ask at every step.

15. **Try `/research` for deep investigations** — Need to understand a complex topic across repos and the web? `/research` runs a thorough search and produces a detailed report with citations.

16. **Connect MCP servers for external tools** — Run `/mcp add` to connect Copilot to databases, APIs, and services. The GitHub MCP server is built in, but you can add your own.

17. **Use `Ctrl+S` to preserve your input** — Submit a command but keep the prompt text in the input box. Handy when you're iterating on a prompt and want to tweak and resubmit.

18. **Scope instructions with `applyTo` globs** — For monorepos, create `.github/instructions/**/*.instructions.md` files with `applyTo` patterns so Copilot gets the right conventions for each package or app.

19. **Use `/review` for AI code review** — Get a focused code review of your staged or unstaged changes. It only flags genuine issues — bugs, security problems, logic errors — not style nits.

20. **Undo mistakes with `/undo`** — Made a wrong turn? `/undo` rewinds the last step and reverts file changes. No need to manually fix what Copilot broke.
