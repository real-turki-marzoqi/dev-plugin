---
name: start-init
description: Set up (or complete) the project-memory structure, the project config, and the Notati link in any repository — scans the stack, proposes verify commands, asks a few grouped questions, creates only missing files from templates, never overwrites, and reports what exists / was created / needs review. Use for "start-init", "/start-init", or when another workflow says "شغّل start-init أول".
argument-hint: "[optional: project name]"
user-invocable: false
---

# start-init — Prepare a project for the dev/tester/audit workflows

Your job: give this repository the memory structure, `docs/PROJECT_CONFIG.md`, and a verified Notati link, so `dev`, `tester`, `audit` and `identity` can run.

## 0. Setup and governing rules

**Load the shared skills first** (sibling folders; read each `SKILL.md` fully):
`../memory-protocol/` · `../notati-sync/` · `../report-format/` — and `../codex-adapter/` when running in Codex.

| Rule | Detail |
|---|---|
| **Idempotent** | Never delete, overwrite or reformat an existing file. Only create what is missing, fill **empty** config fields, and append missing `.gitignore` lines. Everything else that differs from the templates is reported as "يحتاج مراجعة". |
| Templates | `templates/` next to this `SKILL.md` (`${CLAUDE_SKILL_DIR}/templates/` when loaded as a skill). Copy their structure exactly; fill every `{{PLACEHOLDER}}`. No `{{` may remain in a created file. |
| Secrets | Follow `notati-sync` §0. The Notati URL never enters a file, command output you show, report, or message. |
| Confirmation before writing | Show the proposed values and get answers **before** creating files. Notati values are written only after the explicit "هذا هو المشروع؟" yes. |
| Language | Templates are Arabic. If `language.docs` is not Arabic, translate the prose of created files while keeping structure, tables and status literals as configured. Messages to the user in `language.report` (default Arabic). |
| Plugin name | Read `name` from the plugin manifest (`.claude-plugin/plugin.json` two levels above this file, i.e. `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`; in Codex `.codex-plugin/plugin.json`) and use it for `{{PLUGIN_NAME}}`. Never hard-code it. |
| Not a code task | Do not run phases, write product code, or commit anything except as stated in section 7. |

## 1. Scan the repository

Record, without changing anything:

| Item | How |
|---|---|
| Root | the git root if any, else the current folder |
| Git | `git rev-parse --is-inside-work-tree`; current branch |
| Stack | manifest files present (see 1.1) |
| Existing protocol files | each target in section 4: exists or not |
| Existing config | `docs/PROJECT_CONFIG.md`: parse it; list empty fields and unfilled `{{` placeholders |
| Existing phases | `docs/phases/*.md`: IDs, titles, whether each has a `**notati:**` header line |
| Name / description hints | `README.md` first heading/paragraph, manifest `name`/`description`, folder name |
| `.gitignore` | which of the required lines (section 6) are missing |

### 1.1 Suggested verify commands — examples only

Suggest from what is **actually present** (scripts defined in the manifest win over these defaults). These are illustrative examples, not plugin rules:

| Found | `analyze` | `test` | `format` |
|---|---|---|---|
| `package.json` | its `lint` / `typecheck` script | its `test` script | its `format:check` script, else `npx prettier --check .` |
| `pyproject.toml` / `requirements.txt` | `ruff check .` | `pytest` | `ruff format --check .` |
| `__manifest__.py` (Odoo module) | `ruff check .` | the project's Odoo test command (`--test-enable ... --stop-after-init`) | `ruff format --check .` |
| `pubspec.yaml` | the Dart/SDK `analyze` command (or the workspace tool's script if a workspace config exists) | the SDK `test` command | `dart format --output=none --set-exit-if-changed .` |
| `go.mod` | `go vet ./...` | `go test ./...` | `gofmt -l .` (must print nothing) |
| `Cargo.toml` | `cargo clippy -- -D warnings` | `cargo test` | `cargo fmt --check` |
| `*.csproj` / `*.sln` | `dotnet build -warnaserror` | `dotnet test` | `dotnet format --verify-no-changes` |

`verify.test_path`: the same test command with `{path}`. Unknown stack → leave empty and say so.

## 2. Ask (grouped, recommendation first)

Skip any question whose answer is already in an existing config. Use as few question calls as possible (max 4 questions per call):

**Call 1**
1. **Name and description** — option 1 `(Recommended)`: the detected name + one-line description; "Other" for free text.
2. **Initial phases** — `P0 فقط (Recommended)` / "عندي قائمة مراحل" (free text: one `<ID> — <title>` per line).
3. **Verify commands** — `المقترح من الفحص (Recommended)` with a preview of the 3–4 commands / "Other" to edit.
4. **Languages** — `الوثائق والتقرير عربي، الكود والـ commits إنجليزي (Recommended)` / other.

**Call 2**
1. **Notati** — per `notati-sync` §6: "عندي رابط MCP للمشروع" / "دوّر في حسابي" / "أنشئ مشروع جديد" / "بدون Notati الحين".
2. **Workflow options** (multi-select) — "فيه شاشات (بوابة تجربة الاستخدام)" / "أستخدم audit" (default on) / "أحتاج identity".

## 3. Notati link

Skip if the user chose "بدون Notati الحين" (`notati.enabled: false`, `{{NOTATI_*}}` → empty / `—`).

1. Follow `notati-sync` §6 (find) and §7 (register):
   - **Claude Code:** registration is `claude mcp add --transport http --scope local <server> "<URL>"` — the user runs it (`!` prefix) or you run it if they already pasted the URL. Never `--scope project`.
   - **Codex:** `codex-adapter` §3.2 manual steps (project `.codex/config.toml`, git-ignored), shown with a placeholder only.
2. If the newly registered tools are not loaded in this session, try the private HTTP fallback (`codex-adapter` §3.1) to read `get_context`. If that is not possible: create every file with Notati fields empty, add to memory §9 "ربط Notati بانتظار التحقق — أعد تشغيل `start-init` بعد إعادة فتح الجلسة", and finish. A re-run completes the link.
3. **Confirmation (mandatory):** show project name, key, task count, phase count → "هذا هو المشروع؟". Nothing Notati-related is written before yes.
4. **Statuses:** from `get_context` read **every** workflow state. Propose a mapping (by category: todo → `لم يبدأ`/`not_started`, in_progress → `قيد العمل`/`in_progress`, review → `قيد المراجعة`/`review`, done → `مكتمل`/`done`, blocked → `متوقف`/`blocked`; any extra state → its own label/`other`). Ask once to confirm (recommendation first). No state may be dropped.
5. **Phases:**
   - New project: `create_phase` for every initial phase, named `<ID> — <title>`; store ids in `notati.phase_ids`.
   - Existing project: match existing Notati phases to phase IDs by the `<ID> — ` prefix; list unmatched ones as "يحتاج مراجعة" (never rename or create silently).
   - Existing project with tasks: fill `notati-map.md` rows only for a **new** map file, by the `<ID> — ` prefix of task titles; anything ambiguous → "يحتاج مراجعة".

## 4. Create the missing files

For each target: exists → **"موجود"** (do not touch) · missing → create from the template → **"أُنشئ"** · exists but clearly differs from the protocol (e.g. memory sections missing, mixed table/bullet sections, `CLAUDE.md` not pointing to `AGENTS.md`, phase file without `**notati:**` line) → **"يحتاج مراجعة"** with one line why.

| Target | Template | Notes |
|---|---|---|
| `CLAUDE.md` | `templates/CLAUDE.md` | |
| `AGENTS.md` | `templates/AGENTS.md` | sections 4 and 5 stay as guidance for the owner to fill |
| `docs/PROJECT_CONFIG.md` | `templates/docs/PROJECT_CONFIG.md` | existing → fill only empty fields / leftover placeholders, after confirmation → "حُدّث (حقول ناقصة)" |
| `docs/PROJECT_MEMORY.md` | `templates/docs/PROJECT_MEMORY.md` | §2 one row per phase; §1 next step = `dev <first phase>` |
| `docs/LESSONS.md` | `templates/docs/LESSONS.md` | |
| `docs/AUDIT_LOG.md` | `templates/docs/AUDIT_LOG.md` | always created, even if audit is off |
| `docs/DECISIONS_ARCHIVE.md` | `templates/docs/DECISIONS_ARCHIVE.md` | |
| `docs/phases/00-index.md` | `templates/docs/phases/00-index.md` | one row per phase; `{{STATUS_LITERALS}}` = the configured literals joined by ` · ` |
| `docs/phases/notati-map.md` | `templates/docs/phases/notati-map.md` | `## <phase>` + empty table per phase |
| `docs/phases/<ID>-<slug>.md` | `templates/docs/phases/PHASE-TEMPLATE.md` | one per **new** phase; `<slug>` = short kebab-case of the title; header `**notati:** phase_id = …` mandatory |
| `docs/adr/0000-template.md` | `templates/docs/adr/0000-template.md` | |
| `docs/adr/README.md` | `templates/docs/adr/README.md` | |

Do **not** create `.agents/skills/` or any agent-specific copy of the workflows: the plugin covers Claude Code and Codex.

After writing, Grep the created files for `{{` — must be empty.

## 5. First memory entries

- **New memory file:** §1 and §8 come from the template (filled).
- **Existing memory file:** do not touch §1; add one §8 bullet on top: `- **YYYY-MM-DD — start-init (إكمال):** <what was completed>` and trim §8 to the limit; add §9 notes for anything "يحتاج مراجعة".
- Add a §7 row for each created doc that is not already listed.

## 6. `.gitignore`

Append only the missing lines (create the file if absent), under a comment `# local agent config — may contain secrets`:

```
.claude/settings.local.json
.codex/config.toml
.env
.env.*
```

If `.mcp.json` exists and is tracked by git, do not ignore it; instead check it contains no Notati entry and warn if it does. If it exists and is untracked, add `.mcp.json` as well.

## 7. Report and commit

- Report with `report-format` → template **start-init** (files table: موجود / أُنشئ / حُدّث / يحتاج مراجعة; config summary; Notati summary; what the user must do; next step `dev <first phase>`).
- Do not commit unless the user asks. If they do: one commit `chore: initialize project memory with start-init`, after checking `git diff --cached` contains no URL or token.
