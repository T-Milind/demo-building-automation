# Demo Building Automation

An end-to-end pipeline of four specialized AI agents that takes a local business (a name plus links — Google Maps profile, current website, socials, review platforms) and produces 3-4 distinct, fully-built demo websites behind a single shareable link with a "try another style" switcher, ready to deploy and pitch to the client.

## The pipeline

1. **`business-logic-agent`** — researches the business across every link given, plus its local competitors, and writes a structured `business-logic.md`: the business, its target audience/personas, its current conversion path, what competitors do well, and concrete conversion-optimization opportunities.
2. **`planner-agent`** — reads that file and writes 3-4 distinct, fully self-contained build prompts (one per design direction), following Anthropic's prompt-engineering best practices and pulling real design intelligence (palettes, font pairings, style direction) rather than generic AI defaults.
3. **`coding-agent`** — builds all 3-4 demos as independent Next.js projects in one repo, plus a unified preview shell: one page, one link, with a floating "Try another style · Style X of N" button that instantly swaps between them. Also applies fixes reported by the testing agent and handles GitHub/Vercel deployment.
4. **`testing-agent`** — writes a test plan and thoroughly tests every demo (UI rendering, functionality, the switcher itself, accessibility basics, security hygiene) both locally and on the live deployment, producing a prioritized, actionable report.

The orchestrator (`commands/demo-building-automation.md`) runs all four in sequence, looping the build ↔ test cycle (capped, with escalation to you if it gets stuck) until everything passes, then deploys and re-verifies the live URL.

Agents hand off work through files on disk (`business-logic.md` → `demo-prompts.md` → the repo → test reports) rather than pasting full content back and forth, to keep token usage down across a long pipeline.

## Installing for Claude Code

Copy the agent files and the command file into your global Claude Code config:

```bash
cp agents/*.md ~/.claude/agents/
cp commands/*.md ~/.claude/commands/
```

(On Windows: `~/.claude` is `%USERPROFILE%\.claude`, e.g. `C:\Users\<you>\.claude`.)

Once copied, `business-logic-agent`, `planner-agent`, `coding-agent`, and `testing-agent` become available as subagent types, and `/demo-building-automation` becomes available as a slash command. Run it with the client's name and links:

```
/demo-building-automation Timmy's Mobile Mechanic — https://maps.google.com/..., https://timmysmechanic.com, ...
```

## Using this with other AI tools

The frontmatter block at the top of each file (`name`, `description`, `tools`, `model`) is Claude Code's specific mechanism for registering a subagent and its permitted tools — other tools won't understand that block directly. Everything below the frontmatter, though, is a plain, portable system-prompt/persona definition: paste the body of any agent file as a system prompt (or the relevant section of a custom instructions file) in Cursor, Windsurf, a raw Claude/GPT API call, or any other agentic coding tool, and it works the same way. The orchestrator file (`commands/demo-building-automation.md`) describes the phase-by-phase pipeline logic in plain language for the same reason — adapt the "launch subagent X with input Y" steps to whatever orchestration mechanism your tool provides (separate chat sessions, separate API calls, etc.).

## Requirements

- `gh` (GitHub CLI), authenticated
- `vercel` (Vercel CLI), authenticated
- Node.js/npm
- For visual QA in `testing-agent`: a way to run a throwaway headless browser (e.g. Playwright) — installed to a scratch/temp directory, never as a project dependency

## Status

Built and reviewed, not yet run end-to-end on a real client. Worth a dry run on a test business before relying on it for an actual pitch.
