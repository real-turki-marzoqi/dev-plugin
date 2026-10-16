---
description: Set up or complete the project memory, config and Notati link (idempotent)
argument-hint: "[optional: project name]"
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/start-init/SKILL.md` fully and follow it exactly, with these arguments: $ARGUMENTS
That skill file is the single source of truth for this command; do not call the Skill tool for `start-init` (it resolves to this command).
