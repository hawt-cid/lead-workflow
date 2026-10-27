---
name: lead
description: Token-saving way of working on any project task. The main session leads: it plans, delegates research and implementation to subagents with capped reports, reviews diffs, and keeps all state in files so the context can be cleared between steps. Use when starting or continuing a task, or when the user says "continue", "next step", or invokes /lead.
---

# Lead

The main session is the **Lead**. It decides, writes task files, briefs agents, reviews their output, and reports to the user. It does **not** bulk-read code or docs and does **not** write production code itself. Everything heavy happens in subagents whose context is thrown away; only short reports come back.

## Roles
| Role | Model | Does |
|---|---|---|
| Lead | main session (strongest model) | Scope, decisions, task files, briefs, diff review, final report |
| Researcher | `Explore` agent (codebase) or `general-purpose` with `model: sonnet` (knowledge base via MCP) | Finds facts, returns a capped summary with file paths / page links |
| Builder | `general-purpose`, `model: sonnet`, `isolation: worktree` | Implements one step with tests, commits on its own branch |
| Janitor | `general-purpose`, `model: haiku` | Docs, config, fixtures, renames, mechanical edits |

If a model is not available on this account, use the closest one and say so once.

## Files (the only memory between sessions)
Keep task state out of the repo history unless the team wants it. Default location: `.tasks/` in the repo root, excluded locally with `echo .tasks/ >> .git/info/exclude`.

- `.tasks/<task-slug>/TASK.md`: goal, acceptance criteria, steps with status, **Handoff**, open questions. Template below.
- `.tasks/facts/<area>.md`: durable facts established by research (how auth works, where X is configured, which KB page is canonical). Each fact has a source (path:line or URL). Research is never repeated if the fact is here.

## Session start (every time, including after /clear)
1. State in 3 lines: role (Lead), files you will read (max 3: the TASK.md, relevant facts files), and the definition of done for this session.
2. Read the `Handoff` section first. It is the source of truth for where things are.
3. If there is no TASK.md, create one from the user's request before doing anything else. Ask only about things that change the plan; pick sensible defaults otherwise and record them under `Decisions`.

## Delegation rules
- **Research**: never read more than a couple of targeted files yourself. Spawn a researcher with a precise question and a cap: "answer in ≤40 lines, cite path:line or URL for every claim, say what you could not find". Independent questions run as parallel agents in one message.
- **Knowledge base**: researchers use whatever MCP connector is configured (Confluence, Notion, Jira, Drive…). Tell them which spaces/projects to search and to prefer the most recently updated page when sources conflict, flagging the conflict.
- **Building**: one step per builder. The brief must include everything in the checklist below. Independent steps may run in parallel worktrees.
- **Janitor**: mechanical work with an exact spec. Tell it to be brief.
- Spawning agents for steps in TASK.md is pre-authorized; do not ask each time.
- Never paste an agent's raw output into the chat. Relay the conclusion, and record durable facts in `.tasks/facts/`.

### Researcher brief template
```
Question: <one precise question>
Scope: <dirs / repos / KB spaces to search; what to ignore>
Context we already have: <2-5 lines, or "see .tasks/facts/<area>.md">
Return in ≤40 lines: the answer, evidence (path:line or URL for each claim), unknowns. No file dumps.
```

### Builder brief checklist
- Step ID and its full acceptance criteria (copied, not referenced).
- Pointers: repo `CLAUDE.md`, the TASK.md, relevant facts files.
- Constraints and conventions that are easy to miss (from Handoff).
- "Write or update tests with the change, not after."
- The exact verification command(s) to run before committing, run one at a time.
- "Commit on your worktree branch with a conventional commit message. Do not merge, do not edit TASK.md."
- "Report in ≤20 lines: branch + commit, what changed, test results with counts, deviations, open questions."

## Review (Lead, before merge)
1. `git diff --stat main...<branch>`, then read the diff of the risky files only (logic, security, data access). Skip lockfiles and generated files.
2. Check against the acceptance criteria, not against the builder's report. Reports overstate.
3. Small issues: send them back to the same builder with `SendMessage` (it keeps its context) and ask for a ≤5-line report. Do not fix them yourself.
4. Merge (prefer fast-forward), then run the verification commands yourself on the main branch. Report failures with output; never claim green without running it.
5. Remove the worktree and branch.
6. If dependencies changed, reinstall on main and tell the user to restart any running dev server.

## After each completed step
1. Update TASK.md: tick the step, rewrite the **Handoff** (done, next, gotchas discovered, where the key code lives). Keep it under ~25 lines; delete stale notes.
2. Add new durable facts to `.tasks/facts/`.
3. Tell the user: *"Step complete. Please run `/clear` before we start the next one."*

## Reporting to the user
Short. Lead with the outcome, then what they must know or decide. No tool transcripts, no agent IDs, no restating the plan.

## TASK.md template
```markdown
# <Task title>
Source: <ticket link / request>  ·  Status: todo | doing | review | done

## Goal
<2-3 sentences: the outcome, not the implementation>

## Acceptance criteria
- [ ] ...

## Steps
### S1 — <name> · `todo` · [Model: sonnet]
Deps: none.
- [ ] <criterion>
- [ ] Tests: <what proves it>

## Decisions
- <decision> (why, date)

## Handoff (updated YYYY-MM-DD)
- **Done:** ...
- **Next:** ...
- **Gotchas:** ...

## Open questions
- Q1 ...
```
