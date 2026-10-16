# Changelog

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
