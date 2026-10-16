---
name: verify
description: Shared verification rules used by the dev, tester, audit and identity skills — run the project's configured analyze/test/format/extra commands, treat red as not done, never skip/delete/weaken tests, and record exact numbers as evidence. Load it whenever one of those skills says so; not meant to be run on its own.
user-invocable: false
---

# verify — evidence that the code works

The commands come from `docs/PROJECT_CONFIG.md` → `verify`. This skill never hard-codes a tool.

## 1. Commands

| Key | Meaning | When |
|---|---|---|
| `verify.analyze` | static analysis of the whole project | after every task; final check |
| `verify.test` | the whole test suite | after every task; final check |
| `verify.test_path` | tests of one part; replace `{path}` with the changed package/folder | while iterating on a task |
| `verify.format` | formatting check that fails when files would change | after every task |
| `verify.extra[]` | `{ name, command, when }` — project-specific checks (translations, boundaries, builds...) | when its `when` applies |

- An **empty** command means "not configured": write `<name> — غير مُعدّ` in the evidence line. Never claim it passed.
- A command that cannot run because its tool is not installed yet: write `<name> — غير متاح (<reason>)`. Never claim it passed.
- A stack MCP tool (listed in `stack_skills`) may be used to run the same check, but the configured command defines what "green" means. When in doubt, run the configured command.
- Run from the repository root unless the command says otherwise. Keep the output: the evidence needs the real numbers (`X passed / Y failed`).

## 2. Rules

| Rule | Detail |
|---|---|
| **Red = not done** | A failing analyze/test/format means the task is not finished. Fix and re-run. Never change a status to `review` or `done` while red. |
| **No cheating** | Never delete a test, mark it skipped, loosen an assertion, or exclude a file to get green. |
| Fixing a test | Only when you can prove from the task description that the test itself is wrong (`tester`). Record why in the evidence or the lessons. |
| Regression tests | Tests that proved a defect (`tester`, `audit`) stay in the repo permanently. |
| Every checklist item | needs evidence: file exists, a test passes, or command output. |
| Flaky result | Re-run once in isolation; if it passes alone, record it as a known issue in memory §6, never as green-by-assumption. |
| Generated files | If a command rewrites tracked files as a side effect, restore them before committing and note it in memory §9. |

## 3. Final verification (end of session)

Run `verify.analyze` and `verify.test` once more on the whole project (plus any `extra` whose `when` is `final`). Put the numbers in the report's "الفحص النهائي" table (`report-format` skill).
