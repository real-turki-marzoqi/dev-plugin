---
name: identity
description: Explore and lock the project's visual identity (colors, fonts, tokens) — research, 3 measured directions rendered on the project's key screen, one decision from the owner, then write the approved identity into the project. Use for "identity" or "/identity" with optional hints like "green and yellow" or "refine 2". Requires docs/PROJECT_CONFIG.md (created by start-init).
argument-hint: "[optional: direction hints — e.g. \"green and gold\" or \"refine 2\"]"
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
---

# identity — Visual identity: explore → measure → decide → lock

You are the **design lead** of this project. Colors and fonts in `identity.design_reference` (if any) are **placeholders**, not an approved identity. Your job: produce 3 distinct, measured identity directions, show them on the project's key screen, get **one decision** from the owner, then lock it into the project.

## 0. Setup and governing rules

**Load the shared skills first** — read each one fully: `memory-protocol` · `notati-sync` · `verify` · `report-format`, plus `codex-adapter` when running in Codex.
Path: `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md` (in Codex, where that variable is not expanded: the sibling folder `../<name>/SKILL.md` of this skill).

| Rule | Detail |
|---|---|
| Config | Read `docs/PROJECT_CONFIG.md`. Missing → stop: "⚠️ شغّل `start-init` أول". |
| Owner decides | Identity is a **product decision** (`AGENTS.md` section 6). You propose and recommend; you never choose silently. |
| Single stop point | The decision question in section 5 is the **only** stop. At most **2 questions** in one question call (the choice + optional tweak). After the answer, do not stop until the final report. |
| Evidence, not taste | Every claim about a direction is backed by a number or a source: contrast ratios computed by a script, font license read from its source page, glyph/feature support checked. "Looks modern" is not evidence. |
| Hints | If `$ARGUMENTS` has hints, at least one direction follows them. `refine N` means: keep direction N from `identity.options_file` and produce 3 variations of it. |
| Language | Tokens, code, file names in `language.code`. Docs entries in `language.docs`. All messages and the final report in `language.report`. |

---

### External skills — `stack_skills.identity`
**Read `stack_skills.identity` from `docs/PROJECT_CONFIG.md` and use every listed skill in its stage** (`هوية بصرية`: research in section 3 and the token structure in section 4). A listed skill that is not installed: continue without it and add one line to the report: "سكل `<skill>` غير مثبتة، شغّل `/start-init`". Never stop for this.

---

## 1. Mandatory reading

1. `memory-protocol` §2 essentials: `AGENTS.md`, `docs/PROJECT_MEMORY.md` (sections 3 and 4 — any identity decision already taken wins), `docs/LESSONS.md`.
2. `identity.product_context` and `README.md`: product, audience, slogan, languages and scripts the UI must support.
3. `ux_gate.ux_rules` (if set): performance and UX principles that constrain styling.
4. `identity.design_reference` (if set): current placeholder palette, fonts, component set, and the key screen layout. **Visual reference only — do not copy code.**
5. `identity.token_task` in its phase file (if set): token structure, embedded fonts, component list.
6. If they exist: `identity.output`, `identity.options_file`, `identity.token_file`.

## 2. Hard constraints (every direction must pass all of them)

| Constraint | How to verify |
|---|---|
| Text contrast ≥ 4.5:1 (body) and ≥ 3:1 (large text, icons, focus ring) | script (section 4.2) |
| Semantic colors distinct: success / danger / warning / info / primary call-to-action | contrast between each pair ≥ 3:1 **or** paired with an icon; simulate deuteranopia/protanopia |
| Critical states not color-only | spec says icon + text for each critical state |
| Font pair covers every script in `identity.product_context`; license **SIL OFL** or Apache 2.0 (or as constrained) | read the license on the font's official page; record the URL |
| Tabular digits (`tnum`) in the numbers font when the UI shows aligned numbers | check the font's OpenType features |
| Readable at the smallest size used on the target hardware | render test (section 4.3) |
| No resemblance to a competitor or known brand | name the closest known brands and state why it differs |
| Every entry of `identity.constraints` | as stated there |

## 3. Research (gather inputs before inventing)

1. **Domain data:** query the `هوية بصرية` skills from `stack_skills.identity` for the product domain (color, typography, design system). Treat output as **input, not answer**.
2. **Fonts:** shortlist 4–6 families that cover the required scripts. For each: license, weights, companion family, `tnum` support, readability at small sizes.
3. **Brand vs. UX:** frame what the product should feel like from both perspectives explicitly (a listed design skill can help).
4. **System structure:** token naming primitive → semantic → component, mapping 1:1 to `identity.token_file`.

## 4. Build 3 directions

### 4.1 Define
Three **genuinely different** directions (different hue family or mood, not three shades of one). If a placeholder palette exists, one direction is that palette **cleaned up**, as the baseline. For each:

| Field | Content |
|---|---|
| Name | short name + one-line idea |
| Palette | primitives + semantic tokens: `primary`, `onPrimary`, `cta`, `success`, `danger`, `warning`, `info`, `surface`, `surfaceAlt`, `border`, `text`, `textMuted`, `focus` |
| Fonts | UI font and numbers font, with license URLs |
| Shape & spacing | radius scale, spacing scale (multiples of 4) |
| Rationale | why it fits the audience — 3 bullets max |
| Risks | what could go wrong |

### 4.2 Measure (mandatory evidence)
Write a small script in a scratch/temp folder (any available runtime) that, for every direction:
- Computes WCAG contrast for every text/background pair used on the key screen, and every semantic color against `surface`.
- Simulates deuteranopia and protanopia (standard matrices) and reports the minimum ΔE between `success`, `danger`, `warning`, `cta`.
- Prints a pass/fail table.

A direction that fails a hard constraint is fixed and re-measured — never presented failing.

### 4.3 Render on the key screen
Build **one HTML page** with a tab or column per direction, each rendering `identity.key_screen` in the project's default language and direction at the target resolution, plus a token swatch strip and the contrast table.
- In Claude Code, if an artifact-publishing tool is available, publish it so the owner can open it anywhere; otherwise (and in Codex) save it to `identity.preview_file` and give the path.
- Preview fonts may load from a web font service; the app itself embeds them.

### 4.4 Record options
Write `identity.options_file`: the 3 directions with tokens, fonts + license URLs, the measurement table, your recommendation and why. Link the preview.

## 5. Decision (the only stop)

Ask in **one** question call, in `language.report`:
1. Which direction (options = the 3 directions, recommendation first with `(Recommended)`, each with a short preview of its key tokens).
2. Optional, only if useful: one tweak question (e.g. light only vs. light + dark).

If the owner picks "Other" with free-text changes, apply them, re-measure (4.2), and continue — do not ask again.

## 6. Lock the identity

1. `identity.output`: header line `**الحالة:** ✅ معتمد — YYYY-MM-DD`, final tokens (hex), fonts with license URLs and weights to embed, radius/spacing scales, rules (critical states = icon + text, `tnum` for numbers), and the measurement table as evidence.
2. `docs/PROJECT_MEMORY.md`: §3 decision row (source `صاحب المشروع`); strike the identity row in §4 as `✅ معتمد`; §7 add `identity.output` and `identity.options_file`; §5 note that placeholder colors are superseded; §8 bullet; §9 notes.
3. **If `identity.token_file` already exists in code:** update it to the approved tokens, embed the chosen fonts + license files, run the `verify` commands, and Grep with `ux_gate.hardcoded_style_patterns` for colors/fonts outside the token file. Move affected UI tasks back to role `review` (Notati: the `review` id) with the evidence line:
   ```
   > 🎨 هوية (identity — YYYY-MM-DD): التوكنز اتحدثت للهوية المعتمدة · analyze ✅ · test ✅
   ```
4. **If it does not exist yet:** under `identity.token_task` in its phase file add: `> 🎨 الهوية المعتمدة في <identity.output> — استخدمها بدل الألوان المؤقتة.`
5. `docs/LESSONS.md`: log any mistake made during the session (source `identity`).

## 7. Final report
`report-format` → template **identity**.
