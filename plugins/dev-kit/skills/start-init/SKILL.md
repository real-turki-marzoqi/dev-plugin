---
name: start-init
description: Set up (or complete) the project-memory structure, the project config, the Notati link and the project's external skills in any repository — scans the stack, proposes verify commands and project types, asks a few grouped questions, creates only missing files from templates, never overwrites, offers catalog skills for explicit approval, enables them for this project only, and reports what exists / was created / needs review. Use for "start-init" (add "--dry-run" to only show the skills plan), or when another workflow says "شغّل start-init أول".
argument-hint: "[--dry-run] [optional: project name]"
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
---

# start-init — Prepare a project for the dev/tester/audit workflows

Your job: give this repository the memory structure, `docs/PROJECT_CONFIG.md`, a verified Notati link, and the external skills that fit its project type, so `dev`, `tester`, `audit` and `identity` can run.

## 0. Setup and governing rules

**Load the shared skills first** — read each one fully: `memory-protocol` · `notati-sync` · `report-format`, plus `codex-adapter` when running in Codex.
Path: `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md` (in Codex, where that variable is not expanded: the sibling folder `../<name>/SKILL.md` of this skill).

| Rule | Detail |
|---|---|
| **Idempotent** | Never delete, overwrite or reformat an existing file. Only create what is missing, fill **empty** config fields, append missing `.gitignore` lines, and merge missing keys into settings files. Everything else that differs from the templates is reported as "يحتاج مراجعة". |
| Templates | `templates/` next to this `SKILL.md` (`${CLAUDE_SKILL_DIR}/templates/` when loaded as a skill). Copy their structure exactly; fill every `{{PLACEHOLDER}}`. No `{{` may remain in a created file. |
| Catalog | `catalog/skills-catalog.json` at the plugin root (`${CLAUDE_PLUGIN_ROOT}/catalog/skills-catalog.json`; in Codex `../../catalog/skills-catalog.json` from this file). Parse it as JSON. Never copy an external skill into the plugin or invent one that is not in the catalog. |
| External code | Catalog skills are third-party code. **Nothing is installed without the user's explicit approval** in section 6. |
| `--dry-run` | Do sections 1–5 and 7–8 normally; in section 6 build and **show** the full plan (list, install commands, activation settings) but install nothing and write no settings file. Record in memory §9 that the skills plan is pending. |
| Secrets | Follow `notati-sync` §0. The Notati URL never enters a file, command output you show, report, or message. |
| Confirmation before writing | Show the proposed values and get answers **before** creating files. Notati values are written only after the explicit "هذا هو المشروع؟" yes. |
| Language | Templates are Arabic. If `language.docs` is not Arabic, translate the prose of created files while keeping structure, tables and status literals as configured. Messages to the user in `language.report` (default Arabic). |
| Plugin name | Read `name` from `.claude-plugin/plugin.json` two levels above this file (`${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`) and use it for `{{PLUGIN_NAME}}`. Never hard-code it. |
| Not a code task | Do not run phases, write product code, or commit anything except as stated in section 8. |

## 1. Scan the repository

Record, without changing anything:

| Item | How |
|---|---|
| Root | the git root if any, else the current folder |
| Git | `git rev-parse --is-inside-work-tree`; current branch |
| Stack | manifest files present (see 1.1) |
| Project types | proposal from 1.2 |
| Existing protocol files | each target in section 4: exists or not |
| Existing config | `docs/PROJECT_CONFIG.md`: parse it; list empty fields, unfilled `{{` placeholders, `project_types`, `skills_catalog.reviewed` |
| Existing phases | `docs/phases/*.md`: IDs, titles, whether each has a `**notati:**` header line |
| Name / description hints | `README.md` first heading/paragraph, manifest `name`/`description`, folder name |
| `.gitignore` | which of the required lines (section 7) are missing |

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

### 1.2 Suggested project types — examples only

Values: `web` · `mobile` · `desktop` · `backend` · `odoo`. Propose every type the evidence supports:

| Evidence | Suggest |
|---|---|
| `__manifest__.py` / `__openerp__.py` | `odoo` (+ `backend`) |
| `pubspec.yaml` with `android/` or `ios/` folders | `mobile` |
| `pubspec.yaml` with `windows/`, `macos/` or `linux/` folders | `desktop` |
| `package.json` depending on a browser UI framework, or `index.html` | `web` |
| `package.json` depending on a mobile UI framework (e.g. `react-native`, `expo`) | `mobile` |
| `package.json` depending on a desktop shell (e.g. `electron`, `@tauri-apps/*`) | `desktop` |
| `*.xcodeproj` / `Package.swift` with an app target | `mobile` and/or `desktop` (Apple) |
| server framework dependency, `go.mod`, `Cargo.toml` binary, `requirements.txt` with a web/API framework, `Dockerfile` exposing a port | `backend` |

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
3. **Project types** (multi-select) — ويب · موبايل · ديسكتوب · باك إند/API · أودو, with the section 1.2 proposal marked `(Recommended)`.

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
| `docs/PROJECT_CONFIG.md` | `templates/docs/PROJECT_CONFIG.md` | existing → fill only empty fields / leftover placeholders, after confirmation → "حُدّث (حقول ناقصة)"; `project_types` from Call 2 |
| `docs/PROJECT_MEMORY.md` | `templates/docs/PROJECT_MEMORY.md` | §2 one row per phase; §1 next step = `dev <first phase>` |
| `docs/LESSONS.md` | `templates/docs/LESSONS.md` | |
| `docs/AUDIT_LOG.md` | `templates/docs/AUDIT_LOG.md` | always created, even if audit is off |
| `docs/DECISIONS_ARCHIVE.md` | `templates/docs/DECISIONS_ARCHIVE.md` | |
| `docs/phases/00-index.md` | `templates/docs/phases/00-index.md` | one row per phase; `{{STATUS_LITERALS}}` = the configured literals joined by ` · ` |
| `docs/phases/notati-map.md` | `templates/docs/phases/notati-map.md` | `## <phase>` + empty table per phase |
| `docs/phases/<ID>-<slug>.md` | `templates/docs/phases/PHASE-TEMPLATE.md` | one per **new** phase; `<slug>` = short kebab-case of the title; header `**notati:** phase_id = …` mandatory |
| `docs/adr/0000-template.md` | `templates/docs/adr/0000-template.md` | |
| `docs/adr/README.md` | `templates/docs/adr/README.md` | |

Do **not** create `.agents/skills/` copies of this plugin's workflows: the plugin covers Claude Code and Codex. (External catalog skills installed per project in section 6 may land there; that is expected.)

After writing, Grep the created files for `{{` — must be empty.

## 5. First memory entries

- **New memory file:** §1 and §8 come from the template (filled).
- **Existing memory file:** do not touch §1; add one §8 bullet on top: `- **YYYY-MM-DD — start-init (إكمال):** <what was completed>` and trim §8 to the limit; add §9 notes for anything "يحتاج مراجعة".
- Add a §7 row for each created doc that is not already listed.

## 6. External skills from the catalog

Runs after the config exists.

### 6.1 Select
1. Parse the catalog. Keep entries whose `project_types` contain `all` or any of the config's `project_types`.
2. **Re-run on a configured project** (`skills_catalog.reviewed` is not empty): offer **only** entries whose `id` is not in `skills_catalog.reviewed`, plus entries that became relevant because a project type was added. If nothing is new → one line "لا سكلز جديدة في الكتالوج" and skip to 6.6. Never re-ask about reviewed entries.

### 6.2 Detect what is installed
Run what is available on this machine (skip a tool that is not installed):

| Check | Claude Code | Codex |
|---|---|---|
| Plugins | `claude plugin list --json` → installed ids and their enabled state | `codex plugin list` → lines `<id> (installed, enabled)` |
| Per-project skills (`npx-skills` / `manual`) | `.claude/skills/<skill>/SKILL.md` in the project, or the user-level skills folder | `.agents/skills/<skill>/SKILL.md` in the project, or the user-level skills folder |

Classify each selected entry: **مثبتة** (installed for the current agent; note if the other agent lacks it) / **ناقصة**.

### 6.3 Show and ask (explicit approval)
Table of the missing entries: `الاسم · الغرض · المصدر · الرخصة · نوع التثبيت`. One question: **تثبيت الكل / اختيار / تخطي**. "اختيار" → a multi-select of the entries. Then, if any `npx-skills`/`manual` entry was chosen, one more question: **"أضيف مجلدات السكلز المثبتة على مستوى المشروع إلى `.gitignore`؟"** (yes / no).

No answer or "تخطي" → install nothing. `--dry-run` → show the table and the commands of 6.4 and 6.5, install nothing, write no settings.

### 6.4 Install (approved entries only)
Use the entry's `claude_install` / `codex_install` exactly as written in the catalog, for each agent installed on this machine.

| `install_type` | Scope |
|---|---|
| `plugin` | **user level**, once per machine (Claude Code: `claude plugin install <plugin_id> --scope user -y` after adding the marketplace if missing; Codex: `codex plugin marketplace add <repo>` then `codex plugin add <plugin_id>`) |
| `npx-skills` / `manual` | **project level only** (never global). If the user said yes in 6.3, append the installed folders (e.g. `.claude/skills/<skill>/`, `.agents/skills/<skill>/`) to `.gitignore` |

A failed install: record it (command + error first line) for the report and memory §6, continue with the next entry. Never stop.

### 6.5 Enable for this project only
So skills of other project types never activate here:

| Agent | Where | What |
|---|---|---|
| Claude Code | `.claude/settings.json` (merge; create if missing) | `enabledPlugins`: `"<plugin_id>": true` for every selected `plugin` entry that is installed; `"<plugin_id>": false` for every **other** catalog `plugin` entry that is installed and enabled at user level. `extraKnownMarketplaces`: `"<marketplace>": { "source": { "source": "github", "repo": "<marketplace_repo>" } }` for each selected marketplace that is not built in. |
| Codex | `.codex/config.toml` (git-ignored, trusted project; merge) | `[plugins."<plugin_id>"]` `enabled = true` for selected, `enabled = false` for other installed catalog plugins |

- Never change a key that already exists in those files; report a conflicting value as "يحتاج مراجعة".
- When a plugin was installed at user level **by this run** and is not wanted elsewhere, you may ask once whether to disable it globally (`claude plugin disable <id> --scope user`); never disable at user level a plugin that was already enabled before this run.
- Per-project skills (`npx-skills` / `manual`) are already project-only.

### 6.6 Write the config
In `docs/PROJECT_CONFIG.md` (merge; keep existing entries):

```yaml
project_types: [web, backend]
stack_skills:
  dev:      [ { id: <catalog id>, skill: <invocation name>, stage: <stage> } ]
  tester:   [ ... ]
  audit:    [ ... ]
  identity: [ ... ]
skills_catalog:
  catalog_updated: <catalog "updated" value>
  reviewed: [<every catalog id shown to the user, installed or not>]
  declined: [<ids the user skipped>]
```

- One `stack_skills` row per approved entry **per command** in its `used_by`, per skill in its `skills[]`: `{ id: <entry id>, skill: <skills[].invoke>, stage: <stages[<command>]> }`.
- Declined or failed entries are not added to `stack_skills`; they are listed in `declined` (failed ones also in memory §6).
- `--dry-run`: write `project_types` only; add the planned `stack_skills` as a note in memory §9.

## 7. `.gitignore`

Append only the missing lines (create the file if absent), under a comment `# local agent config — may contain secrets`:

```
.claude/settings.local.json
.codex/config.toml
.env
.env.*
```

If `.mcp.json` exists and is tracked by git, do not ignore it; instead check it contains no Notati entry and warn if it does. If it exists and is untracked, add `.mcp.json` as well. `.claude/settings.json` stays tracked (it carries the project's plugin activation).

## 8. Report and commit

- Report with `report-format` → template **start-init** (files table: موجود / أُنشئ / حُدّث / يحتاج مراجعة; config summary; Notati summary; skills table: مثبتة / ثُبّتت / فشلت / تخطّى / مخطط (dry run); activation per agent; what the user must do; next step `dev <first phase>`).
- Do not commit unless the user asks. If they do: one commit `chore: initialize project memory with start-init`, after checking `git diff --cached` contains no URL or token.
