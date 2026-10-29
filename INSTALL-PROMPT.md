# Install prompt

Paste everything below the line into Claude Code on the machine where you want the `lead` skill.
Claude does the install and asks you only for the machine profile details.

---

Install the "lead workflow" skill kit from https://github.com/hawt-cid/lead-workflow for my user-level Claude Code setup. Do the steps below in order. Look before you change anything, and don't overwrite or delete existing files without asking me.

1. **Get the repo.** If `~/lead-workflow` already exists and is a clone of that repo, run `git -C ~/lead-workflow pull`. If it exists but is something else, stop and ask me. Otherwise run `git clone https://github.com/hawt-cid/lead-workflow.git ~/lead-workflow`.

2. **Link the skill.** Run `mkdir -p ~/.claude/skills`. If `~/.claude/skills/lead` doesn't exist, create a symlink to `~/lead-workflow/skills/lead`. If it is already a symlink to that path, leave it. If it is anything else, show me what it is and ask before replacing it. On Windows, where symlinks may need admin rights, use a directory junction (`mklink /J`) and fall back to a copy only if that fails; tell me that a copy won't update with `git pull`.

3. **Import the global instructions.** Make sure `~/.claude/CLAUDE.md` contains the line `@~/lead-workflow/global-CLAUDE.md`. Create the file if it's missing and append the line if it's absent. Don't touch any other content. If my existing global instructions contradict `global-CLAUDE.md`, list the conflicts for me. Mine take precedence, so don't edit either file to resolve them.

4. **Create the machine profile.** If `~/.claude/lead-profile.md` already exists, leave it alone. Otherwise copy `~/lead-workflow/lead-profile.example.md` there. Then fill it in with me:
   - Check which agent types, skills and MCP servers this session has, and propose values for each line from them.
   - Ask me in one batch for whatever you can't infer: base branch, tracker and project key, knowledge-base spaces, the reviewer for deep review, and areas that deserve a deep suggestion.
   - Delete lines that don't apply rather than leaving `<placeholders>`. This file stays on this machine, so work-specific names belong here and never in the repo.

5. **Verify and report.** Confirm that the symlink resolves, that `~/.claude/skills/lead/SKILL.md` is readable, and that the import line is present. Then tell me in a few lines what you did, what you skipped, and that I need to restart Claude Code. After the restart the skill appears as `/lead`. Updates later are `git -C ~/lead-workflow pull`.

Don't commit or push anything, and don't change any project repo. This install touches only `~/lead-workflow` and `~/.claude/`.
