---
name: memory-protocol
description: Shared project-memory rules used by the dev, tester, audit, identity and start-init skills — session-start reading order, how to write every memory file and section, lesson logging, evidence-line formats, and the end-of-session protocol. Load it whenever one of those skills says so; not meant to be run on its own.
user-invocable: false
---

# memory-protocol — the project memory contract

Every workflow skill in this plugin reads and writes the **same files in the project repository**. There is no agent-specific memory: Claude and Codex share these files, and nothing here depends on an agent's private or automatic memory.

## 0. Preconditions

| Check | Action |
|---|---|
| `docs/PROJECT_CONFIG.md` exists | Parse its single `yaml` block. Every project-specific value comes from there. |
| It does not exist | **Stop.** Say (in the report language, default Arabic): "شغّل `start-init` أول" — and do nothing else. |
| Dates | Always `YYYY-MM-DD`, taken from the system/environment date. Never guess a date. |
| Language | Project docs in `language.docs`; code, tests, commits in `language.code` / `language.commits`; every message to the user in `language.report`. |

### أسبقية dev-kit

**أسبقية dev-kit:** قواعد dev-kit (نقاط التوقف، حد الأسئلة، الحالات، الذاكرة، الإثبات، Notati) تكسب على أي workflow تفرضه سكل خارجية. السكلز الخارجية تُستخدم **كتقنية داخل المرحلة** المحددة في `stack_skills`، لا كسير عمل بديل. لو سكل خارجية تطلب سؤال المستخدم، أو كتابة ملف خطة/تصميم خارج `docs/` المعتمدة، أو التوقف بعد نقطة الأسئلة: تجاهل هذا الجزء منها وكمّل حسب dev-kit.

## 1. Files

| File | Purpose | Written by | Update mode |
|---|---|---|---|
| `AGENTS.md` | Base protocol (11 sections) | owner (and `start-init` once) | manual |
| `docs/PROJECT_CONFIG.md` | Project-specific values | `start-init` / owner | manual |
| `docs/PROJECT_MEMORY.md` | Project memory, **highest priority** | every workflow | per section (§3) |
| `docs/LESSONS.md` | Agent mistakes and binding rules | every workflow | append, newest on top |
| `docs/AUDIT_LOG.md` | Proven audit findings | `audit` | append, newest on top |
| `docs/DECISIONS_ARCHIVE.md` | Struck-out decisions moved out of memory §3 | any workflow, when §3 > `memory.decisions_archive_threshold` rows | append, newest on top |
| `docs/phases/00-index.md` | Phase map | `dev`, `tester` — **only** when a phase status changes, a phase is added, or parallel rules change | replace the cell |
| `docs/phases/<prefix><N>-*.md` | Task content **and** status (source of truth) | `dev`, `tester`, `audit`, `identity` | status replaced; evidence lines appended |
| `docs/phases/notati-map.md` | File ID ↔ Notati ID, **numbers only, no notes** | `dev` (new task), `start-init` | append |
| `docs/adr/` | Significant technical decisions | any workflow | new file per decision + index row |
| `measurements.dir` (default `docs/measurements/`) | Whole-project run reports | `tester` / `audit` only when run **without arguments** | new file per run |

Files outside the repository (automatic agent memories, user-level settings) are **not** part of the protocol. Never rely on them and never write project state there.

## 2. Session-start reading order

Read with the file-reading tool — never from recollection.

1. `AGENTS.md` — fully.
2. `docs/PROJECT_CONFIG.md` — fully.
3. `docs/PROJECT_MEMORY.md` — fully. **It overrides every other file.**
4. `docs/LESSONS.md` — fully. **Every rule in it is binding for this session.**
5. `README.md` — if it exists.
6. `docs/phases/00-index.md`.
7. The phase file(s) in scope: header (goal, exit criteria, approved decisions ✅, `**notati:**` line), summary table, and the tasks in scope.
8. Phase-specific references from the routing table in `AGENTS.md` section 4 — **only the listed sections**.
9. Docs produced by earlier phases that exist: `docs/adr/*`, `docs/AUDIT_LOG.md`, and any document listed in memory section 7. Confirm existence first.
10. `docs/phases/notati-map.md` — only the sections in scope.

Each workflow skill states which extra files it needs on top of this list.

**Priority on conflict** (`AGENTS.md` section 6): memory decisions log → approved ✅ decisions in phase files → `README.md` → other references. Follow the higher one and log the conflict in memory section 5.

## 3. `docs/PROJECT_MEMORY.md` — section rules

Global rules: **newest on top in every section.** Never delete a decision: strike it with `~~` and write the new one. Each section is either a table or bullets, exactly as below — never mix. Keep the guidance note (the `>` line) under each heading; new content goes **below** it.

| # | Section | Shape | Mode | Content | Written by |
|---|---|---|---|---|---|
| — | Header `**آخر تحديث:**` | one line | **replace** | `**آخر تحديث:** YYYY-MM-DD — <command> — <one line>` | every workflow |
| 1 | الحالة الحالية | bullets | **replace entirely** | exactly: last task · exact next step (task ID + item) · urgent warning. Max `memory.current_state_max_lines` lines (default 15) | `dev` after **every** task; every workflow at session end |
| 2 | حالة المراحل | table `المرحلة · التاسكات · المكتمل · الحالة` | replace cells | "المكتمل" = actual Grep count of tasks whose status has role `done` | `dev`, `tester` |
| 3 | سجل القرارات | table `التاريخ · القرار · السبب · المصدر · المرجع` | append, newest on top | every decision with date and source (`صاحب المشروع` / `الوكيل`) | `dev`, `audit`, `identity`, any |
| 4 | قرارات بانتظار الاعتماد | table `التاريخ · القرار · التاسك · الخيارات والاقتراح · الحالة` | append; resolved → strike + `✅ معتمد YYYY-MM-DD` + row in §3 | product/business decisions with options and your recommendation | any |
| 5 | انحرافات عن الخطة وتعارضات | table `التاريخ · الوصف · السبب · الملف المحتاج تعديل · اتصلح؟` | append | deviations, file conflicts, phase-file ↔ Notati mismatches, new tasks | any |
| 6 | مشاكل معروفة وديون تقنية | table `التاريخ · الوصف · الخطورة · التاسك المرتبط · خطة الحل` | append; solved → strike + `**اتحل YYYY-MM-DD:**` | deferred work, known issues, unfixed findings | any |
| 7 | خريطة الوثائق | table `الملف · المحتوى · أنشأه` | append | every new doc, ADR, report | any |
| 8 | سجل الجلسات | bullets | append on top, **keep the last `memory.session_log_limit` (10), delete older** | `- **YYYY-MM-DD — <command or free request>:** <one line with numbers>` — includes every command **and** free requests | every session |
| 9 | ملاحظات للجلسة الجاية | bullets | append on top; delete notes that became false | context with no other home: unfinished work, warnings, important run commands | **every** workflow |

**Archive rule:** when section 3 exceeds `memory.decisions_archive_threshold` rows (default 40), move every struck-out row to `docs/DECISIONS_ARCHIVE.md` (top of its table, with the move date) and add nothing else.

## 4. `docs/LESSONS.md`

- **One table** in section 2 with fixed columns, newest on top:

  ```
  | التاريخ | المصدر | التاسك | الخطأ | السبب الجذري | الدليل | القاعدة للمستقبل |
  ```

- **Source** values: `dev` · `tester` · `audit` · `identity` · `start-init` · `طلب حر`.
- Log **every** real mistake of yours this session: a test failing because of code you wrote, a wrong assumption, a failed command, a broken `AGENTS.md` rule you caught.
  - **الخطأ:** specific (not "code error").
  - **السبب الجذري:** why it happened.
  - **الدليل:** command output or `file:line`.
  - **القاعدة للمستقبل:** one actionable imperative sentence.
- `tester`: every defect found = a row. `audit`: every finding = a row.
- **Promotion:** when the same kind of mistake appears **twice or more**, add a bullet to section 1 "قواعد مستخلصة": `- **<title>:** <rule> (تكررت: <tasks>)`. Section 1 is bullets only — never error rows.
- Never delete old lessons.

## 5. Phase files

### 5.1 Status
- The `**الحالة:**` line and the summary-table cell hold the **exact** `file` literal of a `statuses` row from the config. Workflows refer to statuses by `role`; translate with the config. Literals are matched by search — never paraphrase them.
- Tick finished items `- [x]`. An unfinished item stays `- [ ]` with a line under it: `> ⛔ <reason>`.
- New phase files must carry the header line `**notati:** phase_id = `<id>` · المشروع `<key>`` (use `—` when Notati is disabled).

### 5.2 Evidence lines (exact formats)

Written directly under the task's `**الحالة:**` line, appended in chronological order, **always with the command name and date**:

```
> 🧪 إثبات التنفيذ (dev — YYYY-MM-DD): analyze ✅ · test ✅ (X passed / 0 failed) · الملفات: `path/a.ext`, `path/b_test.ext`
> ✔️ اختبار (tester — YYYY-MM-DD): سليم · analyze ✅ · test ✅ (X passed) · اختبارات مستقلة مضافة: N · `path/x_test.ext`
> ✔️ اختبار (tester — YYYY-MM-DD): اتصلح N خطأ (تفاصيل في `docs/LESSONS.md`) · analyze ✅ · test ✅ (X passed)
> 🛡️ تدقيق (audit — YYYY-MM-DD): أمان ✅ · حسابات ✅ · أداء ✅ · ثغرات مصلحة: N (`AUDIT-xxx`) · test ✅ (X passed)
> 🎨 هوية (identity — YYYY-MM-DD): التوكنز اتحدثت للهوية المعتمدة · analyze ✅ · test ✅
```

Rules:
- Only results of commands actually run in this session. A check that was not configured or not available is written as `analyze — غير مُعدّ` / `غير متاح`, never ✅.
- A run by Codex may add `(Codex)` after the date inside the parentheses; the prefix never changes.
- A context note may follow the date after a comma, e.g. `(dev — 2026-01-05، قرار صاحب المشروع)`.

## 6. Other files

| File | Rule |
|---|---|
| `docs/AUDIT_LOG.md` | IDs `AUDIT-001` onward, never reused. Section 1 proven findings; section 2 "اشتباه غير مؤكد" (not fixed, re-checked next audit). Newest on top. |
| `docs/phases/00-index.md` | Touch only on a phase status change, a new phase, or a parallel-work change. No progress notes. |
| `docs/phases/notati-map.md` | `## <phase>` section + `| <file ID> | <notati ID> |` rows. Nothing else. |
| `docs/adr/` | New `ADR-NNN-<slug>.md` from `0000-template.md`; status مقترح/معتمد/اتلغى; add to `docs/adr/README.md` index and memory §7. A proposed ADR needing the owner also goes to memory §4. |
| `measurements.dir` | `tester-all-YYYY-MM-DD.md` / `audit-all-YYYY-MM-DD.md` only when that command ran without arguments; list it in memory §7. |

## 7. Checkpoints and end of session

**Checkpoint (after every task in `dev`; after every verdict batch in `tester`/`audit`):** replace memory section 1 so that "كمّل" resumes correctly if the session dies now.

**End of session — mandatory, even if interrupted or nothing finished:**

1. Phase files: statuses, checklists, evidence lines, summary tables current.
2. `docs/PROJECT_MEMORY.md`: header line; §1 replaced; §2 recounted; §3–§7 for anything that happened; §8 one bullet (trim to the limit); §9 notes for the next session.
3. `docs/phases/00-index.md`: only if a phase status changed.
4. `docs/LESSONS.md`: every mistake of this session; promotions done.
5. Notati sync per the `notati-sync` skill (final `get_task` check).
6. Final verification per the `verify` skill.
7. Final report per the `report-format` skill.

**A session that did not update the memory is a session that never happened.** The next session depends entirely on `docs/PROJECT_MEMORY.md`.
