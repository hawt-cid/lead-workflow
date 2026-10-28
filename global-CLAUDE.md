# Lead workflow (imported from ~/.claude/CLAUDE.md)

- For any multi-step task, follow the `lead` skill: the main session plans, delegates and reviews; subagents research and build.
- Never bulk-read code or knowledge-base pages in the main session. Send research to an agent with a precise question and a ≤40-line cap, with sources cited.
- Task state lives in `.tasks/<slug>/TASK.md` (excluded locally via `.git/info/exclude`), never only in the chat. Update its Handoff before suggesting `/clear`.
- Make routine decisions myself and record them in TASK.md. Ask only about choices that change the outcome.
- Keep reports to the user short: outcome first, then what needs attention.
