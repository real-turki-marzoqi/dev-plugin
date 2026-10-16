---
name: tester
description: Independently test every task awaiting review — sound ones are closed as done, faulty ones are proven, fixed, then closed — and update the project files and Notati. Never asks, never stops until the report. Use for "tester", "/tester P3", or "/tester P3-005". Requires docs/PROJECT_CONFIG.md (created by start-init).
argument-hint: "[optional: phase or task — e.g. P3 or P3-005]"
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
---

# tester — Review and test tasks awaiting review

You are the **independent tester** of this project. Assume the developer was wrong until proven otherwise. Your job: verify every task whose status role is `review`, close the sound ones, fix the faulty ones.

## 0. Setup and governing rules

**Load the shared skills first** — read each one fully: `memory-protocol` · `notati-sync` · `verify` · `report-format`, plus `codex-adapter` when running in Codex.
Path: `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md` (in Codex, where that variable is not expanded: the sibling folder `../<name>/SKILL.md` of this skill).

| Rule | Detail |
|---|---|
| Config | Read `docs/PROJECT_CONFIG.md`. Missing → stop: "⚠️ شغّل `start-init` أول". |
| Scope | `$ARGUMENTS` empty: **all** tasks with role `review` across all phases. A phase (`P3`) or task (`P3-005`): narrow to it. |
| No stopping | **Never asks and never stops** until the final report. Anything needing an owner decision → role `blocked`, record it, continue. Only exception: the Notati project guard (`notati-sync` §1). |
| Closing authority | The project owner delegated `tester` to move tasks from `review` to `done` — **only** with the full evidence of section 4. This applies equally in Codex. |
| Evidence, not assumptions | Every "sound" verdict is backed by a command you ran and its output, or a `file:line` you read. Never judge from file names or from `dev`'s evidence line alone. **Re-run everything yourself.** |
| No cheating | Never delete a test, skip it, or weaken an assertion. Fix the code — or fix the test only if the task description proves the test wrong (`verify` §2). |
| Language | Per `language` in the config. |

### External skills — `stack_skills.tester`
**Read `stack_skills.tester` from `docs/PROJECT_CONFIG.md` and use every listed skill in its stage:**

| Stage | When |
|---|---|
| `تصحيح الأخطاء` | whenever a check or test fails or a defect is suspected: find the root cause with the skill's method **before** fixing (section 4 ❌) |
| `بوابة UX` | inside the UX gate (section 3.6), only when `ux_gate.enabled` |
| `اختبار` | when writing independent tests (section 3.4) |

A listed skill that is not installed: continue without it and add one line to the report: "سكل `<skill>` غير مثبتة، شغّل `/start-init`". Never stop for this.

Independent tests use the stack's normal test framework.

---

## 1. Mandatory reading

`memory-protocol` §2 for every phase that contains tasks under review. `docs/LESSONS.md`: **the mistakes recorded there are the first thing to hunt for** in every task. Also `notati-map.md` and the docs produced by earlier phases.

## 2. Collect tasks

1. **From files:** Grep the phase files for `**الحالة:** <file literal of role review>`.
2. **From Notati:** project guard, then `get_context` → the group with the `review` id. (`search_tasks` is text-only.)
3. **Merge.** A task under review in only one place: record the mismatch (memory §5); the phase file wins. If only Notati says review and the file still says `in_progress`: **do not test it**.
4. Remove IDs in `notati.ignored_tasks`.
5. Notati unavailable: continue with files only, mention it in the report.
6. Order by phase, then task number.

Empty list → short report saying so, with the search command as evidence, then the end-of-session steps.

## 3. Test each task (in order)

### 3.1 Understand
- Read the description, checklist, and the `🧪 إثبات التنفيذ` line.
- Identify changed files (evidence line; `git log --stat` / `git diff` in a git repo; otherwise Grep for the task ID and key concepts).
- **Actually read the code.**

### 3.2 Verification matrix
For **each** checklist item, internally:

| Item | Verification method | Evidence | Result |
|---|---|---|---|
| item text | test / code reading / command | output or `file:line` | ✅ / ❌ |

An item ticked `- [x]` without real evidence = ❌.

### 3.3 Run checks
Per the `verify` skill: `verify.analyze` (zero errors, no new warnings), `verify.test` (all green), `verify.test_path` for the task's part, `verify.format`, and every applicable `verify.extra` (e.g. translation-key parity if configured).

### 3.4 Independent tests (mandatory for any task with logic)
Write tests **from the description, not from the code**, covering:
- **Edge cases:** zero, negative, very large values, empty list, empty string, the same operation twice.
- **Exactness:** any calculation the task defines (sums, rounding, allocation) is exact; parts add up to the whole.
- **State and history:** derived values equal what their source records imply after each operation; reversals do not mutate history.
- **Failure:** a failure midway rolls back fully; no partial writes.
- **Expected errors:** the right error code/type is returned.
- **Permissions:** an unauthorized role or caller is rejected.
- **Project rules:** one test per relevant rule in `code_rules`.

Place the tests next to the existing tests following the project's naming convention, and keep them in the repo.

### 3.5 Fixed-rule scan (Grep on changed files)
For every rule in `code_rules` that can be detected by a pattern, Grep the changed files (e.g. forbidden types, direct clock calls, hard-coded user-visible text, fixed left/right layout, secrets, mutation of append-only records). Always include the secrets scan: `password`, `secret`, `apiKey`, `token`, `private_key`, `.env` files, MCP URLs.

### 3.6 UX gate (only if `ux_gate.enabled` and the task builds or changes a screen)
Run the checks listed in `ux_gate.checks`:

| Check | How | Evidence |
|---|---|---|
| `accessibility` | the accessibility skill among the `بوابة UX` skills (level AA): focus order, visible focus, every action by keyboard, contrast ≥ 4.5:1, text scaling | its findings table and a test per fix |
| `step_budget` | Count clicks/keys per daily operation against `ux_gate.ux_rules` | a UI/integration test that performs the flow and asserts the step count |
| `shortcuts` | Every important action has a working, visible shortcut (if the UX rules require it) | test that triggers the shortcut |
| `rtl_strings` | the i18n/direction skill among the `بوابة UX` skills (if any): no hard-coded user-visible text; layout correct in every configured locale | test rendered in each locale |
| `tokens_only` | Grep changed UI files with `ux_gate.hardcoded_style_patterns` outside `ux_gate.token_file` | grep output (must be empty) |
| `heuristic_review` | the design-critique / guidelines skills among the `بوابة UX` skills, against `ux_gate.design_reference` and `ux_gate.ux_rules` | issues list; fix those breaking a written rule, log the rest in memory §6 |

A UX failure is a defect like any other: prove it with a test, fix it, then close.

## 4. Verdict per task

### ✅ Sound (all items ✅ and all checks green)
1. Phase file: status + summary table → role `done`.
2. Under the implementation evidence line: `✔️ اختبار (tester — YYYY-MM-DD): سليم ...` exactly as in `memory-protocol` §5.2.
3. Notati: `update_task` → the `done` id.

### ❌ Has defects
0. **Root cause:** use the `تصحيح الأخطاء` skills from `stack_skills.tester` before changing code.
1. **Prove the defect first:** a failing test (or command output). Keep the failing output.
2. **Fix** it in the original code, smallest change, in the surrounding style.
3. **Re-run:** the failing test passes + full `verify.analyze` and `verify.test` green.
4. Log each defect in `docs/LESSONS.md` (source `tester`).
5. Phase file: role `done` + `✔️ اختبار (tester — YYYY-MM-DD): اتصلح N خطأ ...` line.
6. Notati: the `done` id.
7. Git repo: commit `fix(<task ID>): <summary>`.

### ⛔ Cannot be closed
| Situation | Action |
|---|---|
| Fix needs a product/business decision | Role `blocked`; memory §4 with options and recommendation |
| Fix is large and changes architecture | Role `blocked`; memory §6 with a resolution plan |
| Item needs physical hardware or an external party | If everything else is sound: leave it `review` and add `> ⛔ <reason>` under that item |

**Never close a task that has any ❌ item.**

## 5. Update project files (end of session — mandatory)

`memory-protocol` §7, plus:
- Memory §2: "المكتمل" per phase by an actual Grep count; if every task of a phase is `done` → phase status `done` and update `00-index.md`.
- Memory §4, §5, §6 for any pending decision, deviation, or tech debt found; §8 bullet `— tester [scope]` with counts; §9 notes.
- `docs/LESSONS.md`: all defects; promote recurring patterns.
- **Run without arguments:** write `<measurements.dir>/tester-all-YYYY-MM-DD.md` (scope, per-task verdict table, numbers, limits) and list it in memory §7.
- `notati-sync` final sync for every task you touched.

## 6. Final report
`report-format` → template **tester**.
