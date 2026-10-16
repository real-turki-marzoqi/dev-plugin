---
description: Execute every task of one phase end-to-end (max 2 questions up front, then no stopping)
argument-hint: "<phase — e.g. 3 or P3>"
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dev/SKILL.md` fully and follow it exactly, with these arguments: $ARGUMENTS
That skill file is the single source of truth for this command; do not call the Skill tool for `dev` (it resolves to this command).
