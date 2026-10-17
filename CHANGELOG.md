# Changelog

## [0.2.1] — 2026-10-17

### Fixed
- Install `test-driven-development` and `systematic-debugging` individually with project-level `npx skills` for Claude Code, matching Codex; remove plugin metadata and the `superpowers:` invocation prefix to avoid the full plugin's SessionStart/bootstrap workflow.
- Add the shared dev-kit precedence rule to `memory-protocol` and reference it from `dev`, `tester`, `audit` and `identity`: external skills provide techniques within `stack_skills` stages and cannot override dev-kit workflows.
- Align `notati-sync` §7 with `codex-adapter` §3.2: default to git-ignored project `.codex/config.toml` in a trusted project; keep global `notati-<key>` with `notati.server_codex` as the alternative.
- Remove the private-repository installation warning from README and document dev-kit precedence in the catalog section.

## [0.2.0] — 2026-10-17

### Added
- External skills catalog `plugins/dev-kit/catalog/skills-catalog.json` (13 verified entries: references and install commands only, nothing copied) and its Arabic README.
- `start-init`: project types, catalog filtering, installed-skill detection, explicit install approval, user-level plugins vs project-level `npx skills`, per-project activation (`.claude/settings.json` `enabledPlugins`, `.codex/config.toml` `[plugins."<id>"]`), `stack_skills` per command, `skills_catalog.reviewed` so re-runs show only new entries, and `--dry-run`.
- `dev`, `tester`, `audit`, `identity` read `stack_skills.<command>` and use each skill in its stage; a missing skill is reported, never blocking.

### Changed
- Skills are the entry points in Claude Code (`/dev-kit:<name>`); `commands/` removed. Main skills grant `Read(${CLAUDE_PLUGIN_ROOT}/**)` via `allowed-tools`.
- Codex reads `.claude-plugin/marketplace.json` and `.claude-plugin/plugin.json` directly; `.codex-plugin/` and `.agents/plugins/` removed.
- Config: `project_types`, `stack_skills.{dev,tester,audit,identity}`, `skills_catalog`; removed `ux_gate.accessibility_skill` / `ux_gate.i18n_skill` (now `بوابة UX` skills).

### Removed
- Vendored `flutter-accessibility`, `flutter-internationalization`, `flutter-ui-package` and `LICENSE-vgv.txt`: the plugin no longer copies external skills.

## [0.1.0] — 2026-10-17

### Added
- `start-init` skill: idempotent setup of `AGENTS.md`, `docs/PROJECT_CONFIG.md`, project memory, lessons, audit log, decisions archive, phase index, Notati map, first phase file and ADR templates; Notati project discovery/creation with mandatory confirmation; status mapping from Notati workflow states; `.gitignore` hardening.
- `dev`, `tester`, `audit`, `identity` skills, generalized from an existing project's commands: every project-specific value moved to `docs/PROJECT_CONFIG.md`.
- Shared skills: `memory-protocol`, `notati-sync`, `verify`, `report-format`, `codex-adapter`.
- Claude Code entry commands (`commands/`) and manifests/marketplaces for Claude Code and Codex. The plugin lives in `plugins/dev-kit/` because Codex 0.132 does not list a plugin whose source is the marketplace root.
- Bundled third-party skills `flutter-accessibility`, `flutter-internationalization`, `flutter-ui-package` (MIT, Very Good Ventures).

### Rules settled in this version
- Memory section 1 is replaced (max 15 lines); section 8 keeps the last 10 sessions as bullets; newest on top in every section; each section is either a table or bullets.
- Decisions section archives struck-out rows to `docs/DECISIONS_ARCHIVE.md` past 40 rows.
- `LESSONS.md` has one table with fixed columns; "قواعد مستخلصة" is bullets only.
- Evidence lines always carry the command name: `(dev — YYYY-MM-DD)`.
- The phase file wins for status and content; Notati is the tracking board.
- Every command checks the connected Notati project against the config before writing.
