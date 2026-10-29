---
name: lead
description: Token-saving way of working on any project task, with an opt-in deep mode. The main session leads: it plans, delegates research and implementation to subagents with capped reports, reviews diffs, and keeps all state in files so the context can be cleared between steps. Use when starting or continuing a task, or when the user says "continue", "next step", or invokes /lead.
---

# Lead

The main session is the **Lead**. It decides, writes task files, briefs agents, reviews their output, and reports to the user. It does **not** bulk-read code or docs and does **not** write production code itself. Everything heavy happens in subagents whose context is thrown away; only short reports come back.

## Environment
This skill decides how work is split; the tools come from the machine. `~/.claude/lead-profile.md`, if present, overrides the **Roles** defaults. Otherwise prefer specialized agents, skills and MCP servers listed in your context over generic ones, and check an agent type can use a tool before briefing it to. Repo and global `CLAUDE.md` win on conventions, commands and tool choice. If a model or agent is missing, use the closest one and say so once.

## Roles
| Role | Default (lean) | Does |
|---|---|---|
| Lead | main session (strongest model) | Scope, decisions, task files, briefs, diff review, final report |
| Researcher | `Explore`, `model: haiku`, `effort: high` (codebase lookups); `general-purpose`, `model: sonnet` (knowledge base via MCP, or synthesis across systems) | Finds facts, returns a capped summary with file paths / page links |
| Builder | `general-purpose`, `model: sonnet`, `isolation: worktree` | Implements one step with tests, commits on its own branch |
| Janitor | `general-purpose`, `model: haiku`, `effort: high` (plus `isolation: worktree` for a TASK.md step) | Docs, config, fixtures, renames, mechanical edits |
| Checker | `general-purpose`, `model: haiku`, `effort: high` | Runs verification commands, reports pass/fail per command, counts and failing names in ≤15 lines |

## Depth: lean unless the user says deep
- **On** only on the user's explicit word: `/lead deep <task>` (whole task), "go deep on S2" (all phases of S2), "deep plan / research / build / review for S2" (that phase). Record it as `Depth:` in TASK.md. Then read `deep.md` in this skill's directory and follow it for those phases.
- **Suggest** at most once per step, in one line, when stakes are concrete: security, data migrations or deletes, money, concurrency, public APIs, an area the profile lists, conflicting sources on a key fact, or a builder that failed twice. Without a yes, stay lean.
- **Off** when the step is done (step-level) or when the user says "lean" (task-level). Janitor work is never deep.

## Tickets vs. tasks
Tracker tickets (Jira or similar) are the interface to the team and the business. `.tasks/` is the Lead's private workspace. Never mix them.
- **Words**: "ticket" only means a tracker ticket, named by its key (`PROJ-123`). The Lead's own units are the *task* (TASK.md) and its *steps* (S1, S2…). Never call a step a ticket.
- **Read-only**: read tickets freely. Create, edit, comment, transition, link or log work only when the user asks for that specific action, and show the exact text first.
- **No leaks**: step IDs, `.tasks/` paths and TASK.md content never appear in tickets, commits, branches or PRs. Those reference the ticket key as the repo's conventions say.
- **The ticket owns the requirement**: copy its acceptance criteria into TASK.md tagged with the key; criteria you add are tagged `(ours)`. If the ticket changes or conflicts with the plan, tell the user; do not quietly re-scope.
- **One task per ticket**: the slug starts with the key (`proj-123-short-name`), so `/lead continue PROJ-123` finds it. TASK.md status is internal and never mirrors or changes the ticket's status.
- Blockers, scope questions and findings the team should know go to the user as a ready-to-post draft; the user decides whether to post it.
- **No tracker**: "ticket" then means a step; "next ticket" / "continue" = the next unfinished step in the Handoff. When none remain, say so and offer final checks.

## Files (the only memory between sessions)
Keep task state out of the repo history unless the team wants it. Location: `.tasks/` in the **main worktree** (`git worktree list --porcelain | sed -n '1s/^worktree //p'`), even when you run in a linked worktree: `git worktree remove` deletes excluded files with the worktree. Exclude it with `echo .tasks/ >> "$(git rev-parse --git-path info/exclude)"` (works from any worktree). If the main worktree is outside your allowed directories, ask the user to restart with `--add-dir <main worktree>`; do not fall back silently.

- `.tasks/<task-slug>/TASK.md`: goal, acceptance criteria, steps with status and depth, **Handoff**, open questions. Template below.
- `.tasks/facts/<area>.md`: durable facts established by research (how auth works, where X is configured, which KB page is canonical). Each fact has a source (path:line or URL). Research is never repeated if the fact is here.

## Branches and parallel Leads
Several Leads may run at once, one per ticket, each in its own worktree.
- **Base branch** = the team's development branch: the profile's `Base branch`, else `develop` if it exists, else the repo's default branch. Ticket branches start from it and PRs target it.
- **Integration branch** = the branch your worktree is on (the ticket branch). If that is the base branch, create the ticket branch first from the fresh base (`git fetch origin <base>`, then branch from `origin/<base>`). If `git log --oneline origin/<base>..HEAD` shows commits that are not this ticket's, the branch was cut from elsewhere (e.g. `main`): stop and tell the user. Builders branch from the integration branch and you merge them into it. Never merge into or push the base branch (or `main`); they are reached through the team's PR process.
- **Stay in your lane**: TASK.md records `Worktree` and `Branch`. At session start, if yours differ, stop and tell the user where that task lives. Never touch another task's files, worktrees or branches.
- **Shared facts**: re-read a facts file right before editing it and only add or correct lines; another Lead may have just written to it.
- **Shared resources**: ports, databases, containers and caches may be in use by another Lead. If verification fails with "address in use", a lock, or data you did not create, suspect that before debugging, and tell the user. The repo `CLAUDE.md` should say how to run per-worktree (port, DB name).

## Session start (every time, including after /clear)
1. State in 3 lines: role (Lead), files you will read (max 4: the profile if present, the TASK.md, up to 2 facts files), and the definition of done for this session. Mention depth only if it is not lean.
2. If the folder is not a git repo, `git init` and make a first commit (builder worktrees need one). If `.claude/skills/lead` in the project is a copy rather than a symlink to the source, tell the user once.
3. Read the `Handoff` section first. It is the source of truth for where things are.
4. If there is no TASK.md, create one from the user's request before doing anything else. Ask only about things that change the plan; pick sensible defaults otherwise and record them under `Decisions`, including which agents/skills/connectors fill each role.

## Delegation rules
- **Research**: never read more than a couple of targeted files yourself (deep plan excepted). Spawn a researcher with a precise question and a cap: "answer in ≤40 lines, cite path:line or URL for every claim, say what you could not find". Independent questions run as parallel agents in one message.
- **Knowledge base**: researchers use whatever MCP connector is configured (Confluence, Notion, Jira, Drive…). Tell them which spaces/projects to search (the profile may list them) and to prefer the most recently updated page when sources conflict, flagging the conflict.
- **Why line**: when launching a step, give the user one line saying why it exists, citing the source requirement (`path:line`, ticket key). Not a plan restatement.
- **Parallelism**: at most 2 builders at once, 1 when the user says tokens are tight. If an agent fails (rate limit, crash), look at its worktree and commit what is there as WIP. Then resume it with `SendMessage`, or start a new builder *without* `isolation` and tell it to `cd` to that worktree's absolute path (`isolation: worktree` always makes a fresh one). Never discard unfinished work.
- **Building**: one step per builder. The brief must include everything in the checklist below.
- **Model per step**: when planning, tag each step `Builder: haiku` or `Builder: sonnet`. Haiku (the Janitor, with the full Builder brief) for docs, config, renames, fixtures and single-file changes with an exact spec; Sonnet for new logic, tests that need designing, or more than ~3 files. When unsure, Sonnet.
- **Escalate once**: if a Haiku agent fails, or its output needs a rework round, redo that work with Sonnet instead of a second Haiku attempt. Note it in the Handoff so similar steps start on Sonnet.
- **Janitor**: mechanical work with an exact spec. Tell it to be brief.
- Spawning agents for steps in TASK.md is pre-authorized; do not ask each time. This does not pre-authorize deep.
- Never paste an agent's raw output into the chat. Relay the conclusion, and record durable facts in `.tasks/facts/`.

### Researcher brief template
```
Question: <one precise question>
Scope: <dirs / repos / KB spaces to search; what to ignore>
Tools: <MCP server or skill to use, if any>
Context we already have: <2-5 lines, or "see .tasks/facts/<area>.md">
Return in ≤40 lines: the answer, evidence (path:line or URL for each claim), unknowns. No file dumps.
```

### Builder brief checklist
- Step ID and its full acceptance criteria (copied, not referenced).
- "Base: <integration branch> @ <sha>. Before anything else, check HEAD is that commit; if not, `git reset --hard <sha>` (your worktree is fresh). Install dependencies if the worktree has none, and run the build once before e2e tests."
- Pointers: repo `CLAUDE.md`, the TASK.md, relevant facts files, as **absolute paths**. `.tasks/` is git-excluded, so it does not exist inside a fresh worktree.
- "Commit WIP early and often so a cut-off loses nothing; squash it into one final commit before reporting."
- Skills or MCP tools it should use (test runner, codegen, ticket lookup), from the profile or your context.
- Constraints and conventions that are easy to miss (from Handoff).
- "Write or update tests with the change, not after."
- The exact verification command(s) to run before committing, run one at a time.
- "Commit on your worktree branch with a conventional commit message that references <ticket key> as the repo's conventions say. Never mention step IDs or `.tasks/`. Do not merge, do not edit TASK.md, do not touch the ticket."
- "Report in ≤20 lines: branch + commit, what changed, test results with counts, deviations, open questions."

## Review (Lead, before merge)
1. `git diff --stat <integration>...<branch>`, then read the diff of the risky files only (logic, security, data access). Skip lockfiles and generated files.
2. Check against the acceptance criteria, not against the builder's report. Reports overstate.
3. Small issues: send them back to the same builder with `SendMessage` (it keeps its context) and ask for a ≤5-line report. Do not fix them yourself.
4. Merge into the integration branch (prefer fast-forward), then have a Checker run the verification commands there (run them yourself only if their output is a few lines). Report failures with the failing output; never claim green without a run.
5. Remove the builder's worktree and branch.
6. If dependencies changed, reinstall in your worktree and tell the user to restart this worktree's dev server, if any.

## After each completed step
1. Update TASK.md: tick the step, rewrite the **Handoff** (done, next, gotchas discovered, where the key code lives). Keep it under ~25 lines; delete stale notes. Reset a step-level depth.
2. Add new durable facts to `.tasks/facts/`.
3. Tell the user: *"Step complete. Please run `/clear` before we start the next one."*

## Reporting to the user
Quiet by default: at most one line between tool calls, and only when needed. Do not explain screenshots or diffs while reviewing. One short final report per step: the outcome first, then what the user must know or decide. Root-cause analysis and review notes go in one or two lines under the step in TASK.md, not the chat. No tool transcripts, no agent IDs, no restating the plan.

## TASK.md template
```markdown
# <Task title>
Source: <PROJ-123 + link, or the request>  ·  Status (internal): todo | doing | review | done  ·  Depth: lean
Worktree: <absolute path>  ·  Branch: <ticket branch>

## Goal
<2-3 sentences: the outcome, not the implementation>

## Acceptance criteria
- [ ] (PROJ-123) <copied from the ticket>
- [ ] (ours) <added by us>

## Steps
### S1 — <name> · `todo` · Builder: haiku | sonnet · Depth: lean
Deps: none.
- [ ] <criterion>
- [ ] Tests: <what proves it>

## Decisions
- Tooling: researcher=<…>, builder=<…>, janitor=<…>, checker=<…>, reviewer=<…>, KB=<…>
- <decision> (why, date)

## Handoff (updated YYYY-MM-DD)
- **Done:** ...
- **Next:** ...
- **Gotchas:** ...

## Open questions
- Q1 ...
```
