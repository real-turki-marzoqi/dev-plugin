---
name: notati-sync
description: Shared Notati rules used by the dev, tester, audit, identity and start-init skills — the project guard, tool table, status mapping from the project config, append-only notes, checklist updates, mismatch handling, final sync, offline behaviour, and how to find or create the project and register its MCP server without leaking the secret URL. Load it whenever one of those skills says so; not meant to be run on its own.
user-invocable: false
---

# notati-sync — task-board synchronisation

Notati is the **tracking board** (status, progress, comments). The **phase file** is the source of truth for task content **and** status. When they disagree, the phase file wins: correct Notati and log the mismatch in memory section 5.

All values come from `docs/PROJECT_CONFIG.md` → `notati` and `statuses`. If `notati.enabled` is `false`, skip every Notati step and write `Notati — غير مفعّل` in the report.

## 0. The secret rule (absolute)

The Notati MCP URL contains a **private, project-scoped token**.

- **Never** write it in any repository file, commit, log, script, report, evidence line, or message to the user — not even partially.
- Never echo it back if the user pastes it. Refer to it as "رابط MCP".
- The config stores only `key`, `project_id`, `project_name`, `phase_ids`, and `statuses`.
- `audit` treats a Notati URL in any tracked file as a **critical** secret finding.

## 1. Project guard — first Notati call of every command

1. Call `get_context`.
2. Compare the referenced project with `notati.key` and `notati.project_id` from the config.
3. **Mismatch → stop the whole command before any write** (files or Notati), and make the first line of the reply a warning in `language.report`:
   `⚠️ مشروع Notati المتصل (<key>) لا يطابق الإعداد (<config key>). أوقفت التنفيذ لحماية المشروع الصحيح.`
   This applies even to commands that otherwise "never stop".
4. Config has no `key` / `project_id` yet → run **§6 Find the project** (only `start-init` writes the result; other commands ask the user to run `start-init`).
5. Read the workflow states returned by `get_context`. Any state missing from `statuses` → log it in memory section 5 and tell the user to add it via `start-init`; do not invent a mapping.

## 2. Tools

| Item | Rule |
|---|---|
| Preferred tools | The project-scoped server named `notati.server` (Claude Code: `mcp__<server>__*`). Scope is already the project. |
| Fallback | An account-level Notati connector exposing the same tools. **Always pass the project key** and re-run the guard (§1). |
| Codex | Discover tools by capability; HTTP fallback per the `codex-adapter` skill. |
| Read by status | `get_context` returns tasks grouped by status. `search_tasks` is **text search only** (no status filter). |
| Read one task | `get_task` (returns checklist item ids). |
| Notes | `update_task` with `append_description`. **Never** pass `description` — it replaces the whole text. |
| Checklist | `get_task` → `update_checklist_item` with `item_id` and `done: true`. |
| Status values | Only the `notati` ids listed in `statuses`. Never invent an id (e.g. `in_review` is not a state unless listed). |
| Ignored tasks | Never read-for-update or update any ID in `notati.ignored_tasks`. |
| ID mapping | `docs/phases/notati-map.md`, or `search_tasks` with the file ID. |

## 3. Status mapping

Workflows speak in **roles** (`not_started`, `in_progress`, `review`, `done`, `blocked`, `other`). For each status change:

1. Find the `statuses` row with that role.
2. Write its `file` literal in the phase file (`**الحالة:**` + summary table).
3. Send its `notati` id with `update_task`.

A row with `role: other` (e.g. a "deferred" state) is used only when the phase file already uses its literal or the owner asks for it.

## 4. When to update

| Event | Phase file (role) | Notati |
|---|---|---|
| Task started | `in_progress` | `update_task` → its id |
| Checklist item finished | `- [x]` | `update_checklist_item` (`done: true`) |
| Task finished by `dev`, all checks green | `review` (**never** `done`) | `update_task` → its id |
| Task closed by `tester` | `done` | `update_task` → its id |
| Blocked on a decision or problem | `blocked` | `update_task` → its id + reason via `append_description` |
| Audit note on a task | unchanged | `append_description` with the audit line |
| Non-coding task, material prepared | unchanged | `append_description`: what was prepared |
| New task needed | new ID in the phase file + summary table | `create_task` with title `<file ID> — <title>` in the phase's `phase_id`, same checklist; add the row to `notati-map.md`; log in memory §5 |
| New phase | new phase file with `**notati:** phase_id = …` | `create_phase` named `<phase ID> — <title>`; store the id in `notati.phase_ids` |

## 5. Mismatches, offline, final sync

| Situation | Action |
|---|---|
| Status differs between file and Notati | Phase file wins. Correct Notati, log one row in memory §5. |
| Only Notati says `review` while the file says `in_progress` | Do not treat it as under review. Log the mismatch. |
| Notati tools unavailable / call fails | Say so in the first lines of the session, continue on files only, write `Notati — غير متاح` in the report's final check. Do not stop. |
| Guard mismatch (§1) | Stop. |
| End of session | `get_task` for **every** task touched; confirm status and checklist match the phase file; fix and log any difference. |

## 6. Find the project (used by `start-init`; other commands only when the config lacks it)

1. **Config has `key` + `project_id`:** run the guard (§1). Mismatch → stop and ask.
2. **No config values:** ask **one** question with three options, recommendation first:
   - **"عندي رابط MCP للمشروع"** → register it (§7), then verify through the tools (or the HTTP fallback in `codex-adapter`).
   - **"دوّر في حسابي"** → with the account-level connector, list projects (name + key) and let the user pick one.
   - **"أنشئ مشروع جديد"** → `create_project`, then `create_phase` for each initial phase.
3. **Mandatory confirmation:** show project name, key, task count and phase count, and ask "هذا هو المشروع؟". **Write nothing** (config, phase headers, map) before a yes.
4. Store in the config: `project_name`, `key`, `project_id`, `phase_ids`, and the status table built from the workflow states (§1 step 5 — every state, including extra ones like "deferred", each mapped by the user to a phase-file literal and a role).

## 7. Registering the MCP server without leaking the URL

| Agent | Method | Where it is stored |
|---|---|---|
| Claude Code | `claude mcp add --transport http --scope local <notati.server> "<URL>"` | the user's `~/.claude.json`, under this project's path — **not** in the repo |
| Codex | per the `codex-adapter` skill (user-level config, URL never in the repo) | the user's Codex config |

Rules:
- Prefer that the **user runs the command themselves** (in Claude Code: prefix with `!` in the prompt) with the URL in place of `<URL>`. Show them the command template only, never a filled-in URL.
- If the user already pasted the URL to you, you may run the command yourself; never repeat the URL in any output.
- **Never** use `--scope project` (it writes `.mcp.json` into the repo).
- New MCP servers load in a **new session**. If the tools are not available yet, finish the file work, record Notati as "pending verification" in memory §9, and tell the user to restart and re-run `start-init` (it completes only what is missing).
- Make sure `.gitignore` ignores local files that could contain the URL: `.claude/settings.local.json`, `.mcp.json` (only if it is not already tracked for other servers — otherwise warn), `.codex/config.toml`, `.env`, `.env.*`.
