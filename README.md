# Lead workflow kit

A portable copy of the Opus-plans / Sonnet-builds workflow from the palovka repo, with nothing project-specific.
It runs lean by default. You can switch on a "deep" mode for the steps where quality matters more than tokens.

## Install (on the work machine)
```bash
git clone <this repo> ~/lead-workflow
mkdir -p ~/.claude/skills
ln -s ~/lead-workflow/skills/lead ~/.claude/skills/lead
echo '@~/lead-workflow/global-CLAUDE.md' >> ~/.claude/CLAUDE.md
cp ~/lead-workflow/lead-profile.example.md ~/.claude/lead-profile.md   # then fill it in
```
Restart Claude Code. The skill shows up as `/lead`. Update later with `git -C ~/lead-workflow pull`.

The symlink and `@` import keep your existing global `CLAUDE.md` intact and make updates a `git pull`.
Your existing global instructions win where they conflict with this kit.

## Machine profile
`~/.claude/lead-profile.md` tells the Lead which models, agents, skills and MCP connectors to use on this machine,
which knowledge-base spaces to search, and which areas deserve a deep suggestion.
It lives outside this repo, so work-specific names never end up on GitHub.
Without a profile, the Lead maps roles from whatever agents, skills and connectors the session has.

## Use
1. In the work repo: `/lead <paste the ticket or describe the task>`.
   Claude creates `.tasks/<slug>/TASK.md`, sends research to agents, and proposes steps.
2. Approve or adjust the steps. Builders run in worktrees; Claude reviews and merges.
3. After each step: `/clear`, then `/lead continue <slug>`.

## Deep mode (opt-in)
Lean is the default and Claude never switches on its own. It may suggest deep once per step when the stakes are concrete, e.g. auth, migrations or public APIs.

| You say | Effect |
|---|---|
| `/lead deep <task>` | Whole task runs deep until you say "lean" |
| "go deep on S2" | All phases of step S2 |
| "deep plan / research / build / review for S2" | Just that phase |

Deep adds a plan critique, wider research with second-source checks, a builder on the strongest model, and an independent reviewer.
After each deep phase Claude says in one line what it caught, so you can judge whether it was worth it.

## Optional per repo
- `/init` drafts a repo `CLAUDE.md` (build/test commands, conventions). Builders rely on it.
- Add the knowledge base as an MCP server (Confluence, Notion, Jira…) so researchers can search it.
  Check with your company which connectors and accounts are approved first.
- Commit `.tasks/` instead of excluding it if the team wants shared task notes.
