---
name: GitHub repos for Claude memory and skills
description: Where to push Claude memory files and skill updates at end of each session
type: reference
---

Memory files live in `~/.claude/projects/-Users-kvelasquez/memory/` and must be pushed to GitHub at the end of any session where a learning was captured.

**Primary repo for memory backups:** https://github.com/kieranvelasquez-lang/daily-dealflow-skill
- Push memory files to the `memory/` subdirectory of this repo
- Clone to `/tmp/daily-dealflow-skill`, copy files, commit, push to `main`

**Related repos:**
- https://github.com/kieranvelasquez-lang/project-a-claude-commands — Claude skill/command files (commands/ directory)
- https://github.com/kieranvelasquez-lang/daily-dealflow-skill — daily dealflow skill + memory
- https://github.com/kieranvelasquez-lang/weekly-dealflow-skill — weekly dealflow skill

**When to push:** Automatically at the end of any session where a new memory file was written or an existing one was updated. Do not wait for the user to ask.
