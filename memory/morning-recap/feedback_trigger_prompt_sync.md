---
name: Remote trigger prompt syncing
description: The morning-recap remote trigger fetches the skill from GitHub at runtime — morning-recap.md is the single source of truth
type: feedback
---

The scheduled remote trigger (`trig_014H7ghXu29fSWgMEe4S3yiA`, "Morning Recap") runs in Anthropic's cloud with no access to local files. Its prompt is 3 lines:

> Fetch https://raw.githubusercontent.com/kieranvelasquez-lang/project-a-claude-commands/main/commands/morning-recap.md using WebFetch and follow the instructions exactly. Run fully unattended - no user is present. Skip steps 8 and 9.

**Why:** The trigger is pure infrastructure (when to run, which tools/MCP to attach). All logic lives in `morning-recap.md`. Pushing a change to GitHub is all that's needed — no trigger update required.

**How to apply:** Never put skill logic in the trigger prompt. If the trigger needs updating, it's only for infrastructure changes (schedule, tools, environment). The skill file is always the source of truth.

**Steps 8–9** are routing correction steps (interactive, require user response). They are skipped in the unattended trigger run but available when running `/morning-recap` manually.
