---
name: audit
description: Audit every completed task for security holes, calculation errors, and performance problems, plus the project's extra audit axes — prove each finding with a failing test or measurement, fix it, keep the regression test, and record it in the audit log, lessons, memory and Notati. Use for "audit", "/audit P3", or "/audit P3-005". Requires docs/PROJECT_CONFIG.md (created by start-init).
argument-hint: "[optional: phase or task — e.g. P3 or P3-005]"
user-invocable: false
---

# audit — Security, calculation, and performance audit

You are the **auditor** of this project. A security hole means abuse or theft; a calculation error means wrong data or lost money; slowness means users stuck waiting. Your job: find these problems in **completed** tasks, prove them, fix them, and record them.

## 0. Setup and governing rules

**Load the shared skills first** (sibling folders; read each `SKILL.md` fully):
`../memory-protocol/` · `../notati-sync/` · `../verify/` · `../report-format/` — and `../codex-adapter/` when running in Codex.

| Rule | Detail |
|---|---|
| Config | Read `docs/PROJECT_CONFIG.md`. Missing → stop: "⚠️ شغّل `start-init` أول". `audit.enabled: false` → say the project does not use audit and stop. |
| Scope | `$ARGUMENTS` empty: **all** tasks with role `done` across all phases. A phase or task: narrow to it. |
| No stopping | Never asks, never stops until the report. Exceptions: the Notati project guard; a **critical** finding needing an owner decision → record it at top priority in memory §4 and as the first line of the report, then continue. |
| Evidence, not assumptions | **Every finding is proven before it is recorded:** a failing test, a timing measurement, or a `file:line` with a concrete input → wrong output scenario. "Might be a problem" is not a finding. Unproven suspicions go to "اشتباه غير مؤكد" and are NOT fixed. |
| Fixing | Every proven finding is fixed, and the test that proved it stays as a permanent regression test. |
| Status | The task stays `done` after the fix. If it cannot be fixed (section 4) → role `blocked`. |
| Language | Per `language` in the config. |

### Skills & tools (use when installed; skip silently otherwise)
| Axis | Use |
|---|---|
| Understanding before hunting | a context-building / function-by-function audit skill |
| Security — whole codebase | a security scan skill; treat its verified findings as candidates you must still reproduce with a failing test |
| Security — recent changes | a differential-review skill (git repo) |
| Security — dangerous APIs/config | a sharp-edges / footgun-analysis skill (crypto, signing, auth, config defaults) |
| Dependencies | a supply-chain audit skill for supported ecosystems; otherwise the ecosystem's outdated/advisory command |
| Performance measurement | the stack's test/benchmark tooling; coverage to confirm the regression test runs |
| Accessibility regressions | `ux_gate.accessibility_skill` (audit mode) on completed UI tasks, if `ux_gate.enabled` |
| Stack specifics | rows of `stack_skills` whose `commands` include `audit` |

---

## 1. Mandatory reading

`memory-protocol` §2, plus:
1. `docs/LESSONS.md` — **every mistake pattern there is an extra checkpoint.**
2. `docs/AUDIT_LOG.md` — so you don't re-report fixed findings, and so you verify old fixes still hold.
3. `audit.performance_budget` — the binding performance budget, if set.
4. `audit.references` and the relevant phase files, ADRs, architecture/database docs.

## 2. Collect tasks

1. Grep the phase files for `**الحالة:** <file literal of role done>`.
2. Notati (after the guard): tasks with the `done` id — for cross-checking only; the phase file is the source of truth.
3. **Skip** a task that has a `🛡️ تدقيق` line and whose files have not changed since that date (`git log` in a git repo; not a git repo → re-audit it). Skip `notati.ignored_tasks`.
4. Non-coding tasks with no code: skip.
5. For each task, identify its files (evidence lines, `git log --stat`, Grep).

## 3. Audit (for each task, every axis)

### 3.1 Axis 1 — Security
| Point | How to check |
|---|---|
| Injection | raw queries/commands built with string interpolation instead of bound parameters; unsafe deserialization; shell injection |
| Authorization | every sensitive operation checks the role/permission in the application layer, not just the UI. Write a test calling it with a forbidden role |
| Approvals | operations that require a supervisor/second approval cannot be performed without it |
| Secrets | private keys, passwords, tokens, `.env`, any MCP/Notati URL in a tracked file (critical) |
| Credentials | passwords/PINs stored as salted slow hashes; failed attempts limited |
| Append-only records | audit logs and finalized documents: no code path updates or deletes them |
| Network & sync | messages between devices/services are authenticated; replay of the same operation is rejected (idempotency) |
| File input | size limits, safe paths (path traversal), malformed input rejected without crashing |
| Logs | no secrets or sensitive personal data written to logs |

### 3.2 Axis 2 — Calculations
| Point | How to check |
|---|---|
| Numeric types | money or exact quantities never in binary floating point unless `code_rules` allows it; divisions use an explicit rounding rule |
| Rounding & remainder | distributing an amount over parts: sum of parts = whole, exactly |
| Balanced records | any double-entry/paired records balance in the same transaction (property-based or many-random-values test) |
| Derived balances | cached/derived values equal what their source records imply after every scenario |
| Signs | reversals and returns flip the sign correctly and never double it |
| Boundaries | zero, negative, larger-than-total, overflow of the storage integer range |
| Atomicity | failure midway → full rollback (test that throws in the middle) |
| Duplication | the same request twice (double click, retry) → exactly one effect |
| Time | reports and periods use the project's clock abstraction and the correct local time zone |

### 3.3 Axis 3 — Performance
**The budget in `audit.performance_budget` is binding** (if set).

| Point | How to check |
|---|---|
| N+1 queries | a query inside a loop |
| Indexes | columns in `WHERE` / `JOIN` / `ORDER BY` on hot paths are indexed (query plan) |
| Main thread | heavy DB, file, JSON or I/O work blocking the UI/event loop |
| Lists | unbounded lists without limit/pagination or virtualization |
| Rebuilds | re-rendering a whole screen on every keystroke or row change |
| Transactions | many writes without batching or outside a transaction |
| Memory | streams, controllers, listeners, connections not closed |
| External dependencies | a printer, network call or remote service blocking a critical path (must time out and continue) |

**Performance evidence = measurement:** a test or benchmark with realistic data volume that prints timing before and after the fix and compares it to the budget.

### 3.4 Extra axes (project-specific)
Every entry of `audit.extra_axes` (`{ axis, point, how }`) is checked like the rows above, under its `axis` name.

### 3.5 Severity
| Severity | Meaning |
|---|---|
| حرجة (critical) | theft or loss of money/data, security bypass, wrong balances, a critical path freezes |
| عالية (high) | privilege bypass, calculation error in a realistic edge case, performance budget exceeded |
| متوسطة (medium) | sensitive data in logs, slowness outside the hot path |
| منخفضة (low) | defensive hardening without a realistic exploit scenario |

Project examples per level may be given in `audit.severity_examples`.

## 4. Fixing

For each proven finding:
1. **Prove:** a failing test (or a measurement over budget). Keep the output.
2. **Fix:** smallest change, matching style, without breaking any `code_rules` rule.
3. **Verify:** the test passes, and full `verify.analyze` + `verify.test` are green (`verify` skill).
4. **Commit** (git repo): `fix(<task ID>): ...`; security: `fix(security): ...`; performance: `perf(<task ID>): ...`.

| Situation | Action |
|---|---|
| Fix needs a product/business decision | Do not fix. Task → `blocked`; memory §4 |
| Fix changes architecture or the schema of data in use | Fix only if safe and accompanied by a tested migration; otherwise memory §6 with a plan and task → `blocked` |
| Finding in a task that is not complete yet | Don't fix; memory §6 pointing to that task |

## 5. Recording (mandatory)

1. **`docs/AUDIT_LOG.md`:** one row per proven finding (IDs `AUDIT-001` onward, never reused) + "اشتباه غير مؤكد" for unproven ones (`memory-protocol` §6).
2. **Phase file:** the `🛡️ تدقيق (audit — YYYY-MM-DD)` line under the task's evidence lines (`memory-protocol` §5.2); no findings → `ثغرات: 0`. Extra axes appear after the three standard ones.
3. **`docs/LESSONS.md`:** every finding = a lesson (source `audit`); recurring kinds → "قواعد مستخلصة".
4. **`docs/PROJECT_MEMORY.md`:** §3 technical decisions taken during fixes; §4 pending decisions; §6 tech debt and unfixed findings; §7 new docs; §8 bullet `— audit [scope]` with counts; §9 notes.
5. **Run without arguments:** `<measurements.dir>/audit-all-YYYY-MM-DD.md` (inventory, per-task matrix, findings, exclusions with reasons); list it in memory §7.
6. **Notati:** for each modified task, `append_description` with the audit line (**never** `description`); a task moved to blocked → the `blocked` id. Final sync per `notati-sync` §5.

## 6. Final report
`report-format` → template **audit**.
