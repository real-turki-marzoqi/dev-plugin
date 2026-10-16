---
name: dev
description: Execute every task of one project phase end-to-end — at most 2 questions up front, then run without stopping until the final report. Moves tasks from not-started to in-progress to review (never to done). Use for "dev P3", "/dev 3", "نفذ P3", or "كمّل" on a phase. Requires docs/PROJECT_CONFIG.md (created by start-init).
argument-hint: <phase — e.g. 3 or P3>
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
---

# dev — Execute a full phase

You are the implementing developer of this project. Your job: execute the tasks of phase **`$ARGUMENTS`**.

## 0. Setup and governing rules

**Load the shared skills first** — read each one fully: `memory-protocol` · `notati-sync` · `verify` · `report-format`, plus `codex-adapter` when running in Codex.
Path: `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md` (in Codex, where that variable is not expanded: the sibling folder `../<name>/SKILL.md` of this skill).

| Rule | Detail |
|---|---|
| Config | Read `docs/PROJECT_CONFIG.md`. Missing → stop: "⚠️ شغّل `start-init` أول". |
| Phase parsing | `3`, `P3`, `فيز 3`, `المرحلة 3` all mean `<phases.prefix>3`. Valid phases are the ones listed in `phases.index` (`00-index.md`) — any number. Empty or unknown → ask (counts toward the 2-question limit). |
| Single stop point | **The questions step (section 3) is the ONLY place you may stop.** After it you must NOT stop, ask, request confirmation, or end your turn until every executable task is done and the final report is written. Exception: the Notati project guard (`notati-sync` §1). |
| Question limit | **Maximum 2 questions**, in one question call. No real blocking question → do not ask; start immediately. |
| Evidence, not assumptions | Never write "works", "done" or "correct" without proof from this session: output of a command you ran, or a `file:line` you read. Unverified → say "not verified". |
| End state | A finished task moves to the status with role `review`, **never** `done`. Closing is `tester`'s job. |
| Authority | `AGENTS.md` is the base protocol; this skill details it. On conflict: this skill governs workflow (stopping, questions); `AGENTS.md` section 5 (`code_rules`) governs code. |
| Language | Per `language` in the config (code/commits vs docs vs report). |

### External skills — `stack_skills.dev`
**Read `stack_skills.dev` from `docs/PROJECT_CONFIG.md` and use every listed skill in its stage:**

| Stage | When | Typical skills |
|---|---|---|
| `تخطيط` | before implementing each task: shape the approach, module boundaries, patterns | design patterns, system design |
| `تنفيذ` | while implementing: follow the skill's discipline (e.g. failing test first, then code) | test-driven development |
| `شاشات` | any task that builds or changes a screen | design / UX / platform guideline skills |

A listed skill that is not installed: continue without it and add one line to the report: "سكل `<skill>` غير مثبتة، شغّل `/start-init`". Never stop for this.

Also:

| Trigger | Do |
|---|---|
| Any task that builds or changes a screen, and `ux_gate.enabled` | Before coding, read `ux_gate.design_reference` (visual reference only — never copy its code or logic) and `ux_gate.ux_rules`. |
| An internal page, and `ux_gate.page_template` is set | Read it first and follow it, including any "describe the page before coding" rule it defines. |
| Visual identity | If `identity.output` exists and is marked approved (`✅`), use only its tokens. Otherwise follow section 4.9. |

---

## 1. Mandatory reading

Follow `memory-protocol` §2 (session-start order) for this phase. From the phase file read the header, summary table, and every task whose status role is not `done` or `review`. Then the `notati-map.md` section of this phase.

## 2. Read Notati and check readiness

### 2.1 Notati
- Project guard first (`notati-sync` §1).
- `get_context`, then `get_project` for the configured project, then `get_task` for each task in the phase.
- Unavailable → note it for the report and continue with files only. **Do not stop for this.**
- **The phase file is the source of truth for content and status.** Notati is the tracking board. Record any status mismatch and fix Notati at the end of each task.

### 2.2 Build the execution list
Internal table of every task in the phase:

| Column | Source |
|---|---|
| File ID | `<prefix><N>-xxx` |
| Notati ID | `notati-map.md` |
| Status in file | the `**الحالة:**` line (map to a role via `statuses`) |
| Status in Notati | `get_task` |
| Depends on | description, dependency line, summary-table notes |
| Type | coding / non-coding (`phases.non_coding_label`) / decision gate (`phases.decision_gates`) / has open decision ⚠️ |

- **Order:** by number, unless a dependency note says otherwise.
- **Skip:** tasks whose role is `done` or `review`, and any ID in `notati.ignored_tasks`.
- **Resume:** tasks whose role is `in_progress` from their first unchecked `- [ ]` item.

### 2.3 Readiness check (`AGENTS.md` section 3)
- **Previous phase complete?** Memory section 2. If not, and parallel work is not allowed by `00-index.md` → **mandatory question** in section 3.
- **Open decision ⚠️ blocking a task?** Product/business → candidate question. Technical decision delegated to the task → do NOT ask; measure, write an ADR with status "مقترح" (`AGENTS.md` section 6).
- **Non-coding tasks:** do not execute; prepare supporting material only.
- **Decision gates:** prepare the report only; never decide.

## 3. Questions (the ONLY stop point)

1. Collect only things that **actually** block execution or affect data, money, security or a `code_rules` rule, and cannot be resolved from the files.
2. Rank by impact; pick **at most two**; merge related items.
3. Ask in **one** question call, in `language.report`, each with clear options and your recommendation first, marked `(Recommended)`.
4. **Never ask about:** small technical details you can decide, anything already written in the files, or permission to start.
5. Any open decision that did **not** make the two questions: task → role `blocked`, record it in memory §4 with options and recommendation, continue with the rest.
6. Record the answers immediately in memory §3 with the date and source `صاحب المشروع`.

**From this point on, stopping is forbidden.**

## 4. Execution loop (for each task, in order)

### 4.1 Start
- Phase file `**الحالة:**` + summary table → role `in_progress`.
- Notati → its `in_progress` id.

### 4.2 Implement
- Read the full description and checklist, and the existing code you will touch **before** editing.
- Implement item by item. **Write tests together with the code** for all logic.
- Obey every rule in `code_rules` (`AGENTS.md` section 5). Before marking items done, self-check the changed code against each of those rules, one by one.

### 4.3 Verify (mandatory before changing status)
Per the `verify` skill: run `verify.analyze`, `verify.test`, `verify.test_path` for the changed part, `verify.format`, and applicable `verify.extra`. **Red = the task is not done.** Every checklist item needs evidence.

### 4.4 Record in the phase file
- Tick finished items `- [x]`; unfinished stay `- [ ]` with `> ⛔ <reason>`.
- Under `**الحالة:**` add the `🧪 إثبات التنفيذ (dev — YYYY-MM-DD)` line exactly as in `memory-protocol` §5.2.
- `**الحالة:**` + summary table → role `review`.

### 4.5 Notati
- `update_checklist_item` for each finished item.
- `update_task` → the `review` id.
- Blocked task → the `blocked` id, with the reason via `append_description`.

### 4.6 Commit
- Git repo: one commit per task, Conventional Commits in `language.commits`, scoped by task ID: `feat(<task ID>): <summary>`.
- Not a git repo: do NOT run `git init`; mention it in the report.

### 4.7 Checkpoint (after EVERY task)
Replace memory §1 (last task, exact next step) per `memory-protocol` §7 so that "كمّل" resumes correctly.

### 4.8 Problems during execution
| Situation | Action |
|---|---|
| Unspecified technical detail | Decide, log in memory §3, ADR if significant. **Do not ask.** |
| Product/business decision surfaces late | Task → `blocked`, memory §4, move to the next task |
| Conflict between two files | Priority order (`AGENTS.md` section 6), log in memory §5 |
| Missing work that must be done | New task per `notati-sync` §4 (phase file + `create_task` + `notati-map.md` + memory §5) |
| A mistake you made and caught | Fix it and log it in `docs/LESSONS.md` (`memory-protocol` §4) |

### 4.9 Visual identity not approved yet
Colors and fonts are a **product decision**.
- If `identity.output` is missing or not approved, and the phase needs UI: build the token **structure** with placeholder values from `identity.design_reference` (if any), all in **one** token file (`ux_gate.token_file` / `identity.token_file`), so swapping is a one-file change. Never hard-code a color or font outside that file.
- Record in memory §4 that identity is pending; in the final report recommend running `identity`.
- If identity blocks a task beyond placeholders, it is a valid candidate for one of the 2 questions.

## 5. Mistakes log
Every mistake of yours this session → a row in `docs/LESSONS.md`, source `dev`, promotion after two occurrences (`memory-protocol` §4).

## 6. End of session (mandatory, even if not all tasks finished)
Follow `memory-protocol` §7. Specifically for `dev`: memory §2 "المكتمل" counts only role `done` (the phase stays `in_progress`); memory §7 lists any new file; memory §8 bullet `— dev <phase>`; memory §9 notes. Then the `notati-sync` final sync and the `verify` final check.

## 7. Final report
`report-format` → template **dev**. **Never claim anything without evidence.**
