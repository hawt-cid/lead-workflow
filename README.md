# Lead workflow kit

A portable copy of the Opus-plans / Sonnet-builds workflow from the palovka repo, with nothing project-specific.

## Install (on the work machine)
```bash
mkdir -p ~/.claude/skills
cp -r skills/lead ~/.claude/skills/
cat global-CLAUDE.md >> ~/.claude/CLAUDE.md   # then delete the heading line if you like
```
Restart Claude Code. The skill shows up as `/lead`.

## Use
1. In the work repo: `/lead <paste the ticket or describe the task>`.
   Claude creates `.tasks/<slug>/TASK.md`, sends research to agents, and proposes steps.
2. Approve or adjust the steps. Builders run in worktrees; Claude reviews and merges.
3. After each step: `/clear`, then `/lead continue <slug>`.

## Optional per repo
- `/init` drafts a repo `CLAUDE.md` (build/test commands, conventions). Builders rely on it.
- Add the knowledge base as an MCP server (Confluence, Notion, Jira…) so researchers can search it.
  Check with your company which connectors and accounts are approved first.
- Commit `.tasks/` instead of excluding it if the team wants shared task notes.
