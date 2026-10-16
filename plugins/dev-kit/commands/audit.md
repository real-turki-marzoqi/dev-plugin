---
description: Audit completed tasks for security, calculations and performance; prove, fix, record
argument-hint: "[optional: phase or task — e.g. P3 or P3-005]"
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/audit/SKILL.md` fully and follow it exactly, with these arguments: $ARGUMENTS
That skill file is the single source of truth for this command; do not call the Skill tool for `audit` (it resolves to this command).
