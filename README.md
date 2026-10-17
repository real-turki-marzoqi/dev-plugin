# dev-kit

بلوقن واحد يشتغل على **Claude Code** و**Codex**: سير عمل بالمراحل (`dev` · `tester` · `audit` · `identity`) فوق ذاكرة مشروع مشتركة داخل الريبو، مع مزامنة لوحة المهام **Notati**، و`start-init` اللي يجهّز أي مشروع ويقترح له السكلز الخارجية المناسبة لنوعه.

**الفكرة الأساسية:** البلوقن عام، والمشروع إعداد. كل قيمة خاصة بمشروعك (أوامر التحقق، مفتاح Notati، الحالات، البوابات، محاور التدقيق، السكلز الخارجية) تعيش في `docs/PROJECT_CONFIG.md` داخل المشروع.

---

## المحتويات

| السكل | الوظيفة | ينقل الحالة |
|---|---|---|
| `start-init` | يجهّز الذاكرة والإعداد وربط Notati والسكلز الخارجية. **لا يمسح شيئاً**؛ يكمّل الناقص فقط | — |
| `dev <المرحلة>` | ينفذ تاسكات مرحلة. سؤالين كحد أقصى في البداية، ثم لا يتوقف | لم يبدأ ← قيد العمل ← قيد المراجعة |
| `tester [مرحلة/تاسك]` | يختبر التاسكات قيد المراجعة باستقلال، يصلح المعيب، ويقفل السليم | قيد المراجعة ← مكتمل |
| `audit [مرحلة/تاسك]` | يدقق المكتمل: أمان، حسابات، أداء + محاور المشروع؛ يثبت كل ثغرة ويصلحها | يبقى مكتمل (أو متوقف) |
| `identity [تلميحات]` | 3 اتجاهات هوية بصرية مقاسة بالأرقام، وصاحب المشروع يختار | — |

### سكلز مشتركة (تستخدمها السكلز فوق، لا تُستدعى مباشرة)

| السكل | المحتوى |
|---|---|
| `memory-protocol` | ترتيب القراءة، قواعد كل ملف وكل قسم في الذاكرة، الدروس، صيغ أسطر الإثبات، نهاية الجلسة |
| `notati-sync` | فحص المشروع، جدول الأدوات، ربط الحالات، المزامنة، إيجاد المشروع وتسجيله بدون تسريب الرابط |
| `verify` | أوامر التحقق من الإعداد، "الأحمر = غير منتهٍ"، منع تخطي الاختبارات |
| `report-format` | هيكل التقرير النهائي لكل سكل |
| `codex-adapter` | ترجمة أدوات Claude ← Codex، وبديل HTTP لـ Notati |

---

## التثبيت

> الريبو: [real-turki-marzoqi/dev-plugin](https://github.com/real-turki-marzoqi/dev-plugin).

### Claude Code

```bash
claude plugin marketplace add real-turki-marzoqi/dev-plugin
claude plugin install dev-kit@dev-kit-marketplace
```

أو داخل الجلسة: `/plugin marketplace add real-turki-marzoqi/dev-plugin` ثم `/plugin install dev-kit@dev-kit-marketplace`.

### Codex

Codex يقرأ نفس ملف `.claude-plugin/marketplace.json`، فما يحتاج ملفات خاصة:

```bash
codex plugin marketplace add real-turki-marzoqi/dev-plugin
codex plugin add dev-kit@dev-kit-marketplace
```

**ابدأ جلسة Codex جديدة** بعد التثبيت. على ويندوز، لو ظهر `Filename too long`: `git config --global core.longpaths true`.

---

## الاستخدام

| الخطوة | Claude Code | Codex |
|---|---|---|
| تجهيز المشروع | `/dev-kit:start-init` | `$dev-kit:start-init` |
| عرض خطة السكلز فقط | `/dev-kit:start-init --dry-run` | `$dev-kit:start-init --dry-run` |
| تنفيذ مرحلة | `/dev-kit:dev P0` | `$dev-kit:dev P0` |
| الاختبار | `/dev-kit:tester` | `$dev-kit:tester` |
| التدقيق | `/dev-kit:audit P0` | `$dev-kit:audit P0` |
| الهوية البصرية | `/dev-kit:identity` | `$dev-kit:identity` |

- أي سكل ما تلقى `docs/PROJECT_CONFIG.md` توقف وتطلب تشغيل `start-init` أول.
- أي سكل تبدأ بفحص أن مشروع Notati المتصل يطابق الإعداد. عدم التطابق = توقف قبل أي كتابة.
- سكل خارجية مذكورة في الإعداد وغير مثبتة: الكوماند يكمّل بدونها ويكتب في التقرير "سكل X غير مثبتة، شغّل `/start-init`".

### الملفات اللي ينشئها `start-init` في مشروعك

```
CLAUDE.md                     ← "اقرأ AGENTS.md"
AGENTS.md                     ← البروتوكول (11 قسم)
docs/PROJECT_CONFIG.md        ← إعداد المشروع (كتلة yaml واحدة)
docs/PROJECT_MEMORY.md        ← الذاكرة (9 أقسام)
docs/LESSONS.md               ← الدروس (جدول واحد + قواعد مستخلصة)
docs/AUDIT_LOG.md             ← ثغرات audit
docs/DECISIONS_ARCHIVE.md     ← القرارات المشطوبة القديمة
docs/phases/00-index.md       ← خريطة المراحل
docs/phases/notati-map.md     ← تحويل الأرقام فقط
docs/phases/P0-<slug>.md      ← ملف المرحلة الأولى
docs/adr/0000-template.md · docs/adr/README.md
.claude/settings.json         ← تفعيل بلوقنز الكتالوج المختارة لهذا المشروع (دمج، لا استبدال)
.codex/config.toml            ← نفس التفعيل لـ Codex (محلي، يتجاهله git)
```

---

## الكتالوج

**الخلاصة:** `plugins/dev-kit/catalog/skills-catalog.json` قائمة مراجع لسكلز خارجية موثّقة مع أوامر تثبيتها. **البلوقن لا ينسخ أي سكل خارجية داخله.** التفاصيل الكاملة في [`catalog/README.md`](plugins/dev-kit/catalog/README.md).

**أسبقية dev-kit:** قواعده تكسب على أي workflow خارجي؛ السكلز الخارجية تقنية داخل مرحلة `stack_skills` فقط، وفق [قاعدة الأسبقية](plugins/dev-kit/skills/memory-protocol/SKILL.md#أسبقية-dev-kit).

### كيف يشتغل

1. `start-init` يقترح نوع المشروع من الفحص وتأكده أنت (اختيار متعدد): ويب · موبايل · ديسكتوب · باك إند/API · أودو.
2. يصفّي الكتالوج بالأنواع المختارة + `all`، ويفحص المثبّت عندك.
3. يعرض الناقص في جدول ويسأل: **تثبيت الكل / اختيار / تخطي**. لا شيء يتثبت بدون موافقتك.
4. البلوقنز تتثبت على مستوى المستخدم مرة وحدة، وسكلز `npx skills` على مستوى المشروع فقط.
5. يفعّل المختار **لهذا المشروع فقط** ويطفّي بلوقنز الكتالوج الثانية فيه، عشان سكلز الموبايل ما تشتغل في مشروع ويب.
6. يكتب `project_types` و`stack_skills.<الكوماند>` في `docs/PROJECT_CONFIG.md`، والكوماندز تستخدم كل سكل في مرحلتها.
7. إعادة التشغيل تعرض **الجديد فقط**.

### السكلز بالأنواع

| السكل | الغرض | المصدر | الأنواع | تستخدمها (المرحلة) | الرخصة |
|---|---|---|---|---|---|
| `tdd` | TDD: اختبار فاشل أولاً | `obra/superpowers` | الكل | `dev` (تنفيذ) · `tester` (اختبار) | MIT |
| `systematic-debugging` | السبب الجذري قبل الإصلاح | `obra/superpowers` | الكل | `dev` (تنفيذ) · `tester` (تصحيح الأخطاء) | MIT |
| `design-patterns` | باترنز التصميم | `itzcull/software-skills` | الكل | `dev` (تخطيط) · `audit` (روائح الكود) | MIT |
| `system-design` | تصميم الأنظمة | `itzcull/software-skills` | الكل | `dev` (تخطيط) · `audit` (روائح الكود) | MIT |
| `code-smells` | روائح الكود وإعادة الهيكلة | `itzcull/software-skills` | الكل | `audit` (روائح الكود) | MIT |
| `owasp-security` | OWASP Top 10 وASVS | `agamm/claude-code-owasp` | الكل | `audit` (تدقيق أمني) · `tester` (اختبار) | MIT |
| `differential-review` | مراجعة أمنية للتغييرات | `trailofbits/skills` | الكل | `audit` (تدقيق أمني) | CC-BY-SA-4.0 |
| `post-patch-validation` | التحقق من إصلاح الثغرة | `trailofbits/skills` | الكل | `audit` (تحقق من الإصلاح) | CC-BY-SA-4.0 |
| `ui-ux-pro-max` | قرارات تصميم الواجهات | `nextlevelbuilder/ui-ux-pro-max-skill` | ويب · موبايل · ديسكتوب | `dev` (شاشات) · `identity` | MIT |
| `impeccable` | نقد وفحص تصميم الواجهة | `pbakaus/impeccable` | ويب · موبايل · ديسكتوب | `tester` (بوابة UX) | Apache-2.0 |
| `web-design-guidelines` | إرشادات Vercel لواجهات الويب | `vercel-labs/agent-skills` | ويب | `tester` (بوابة UX) | MIT (في README) |
| `apple-hig` | إرشادات Apple (iPhone وMac) | `ehmo/platform-design-skills` | موبايل · ديسكتوب | `dev` (شاشات) · `tester` (بوابة UX) | MIT |
| `odoo-18` | مرجع تطوير أودو 18 | `unclecatvn/agent-skills` | أودو | `dev` (تنفيذ) · `tester` (اختبار) | MIT |

### إضافة سكل جديدة

1. تحقق من الريبو والرخصة (بدون رخصة = لا تُضاف) وأمر التثبيت من README أو المانيفست.
2. أضف كائناً في `skills` داخل `plugins/dev-kit/catalog/skills-catalog.json` بكل الحقول، وحدّث `updated`.
3. تأكد إن الـ JSON صالح.
4. ارفع `version` في `plugins/dev-kit/.claude-plugin/plugin.json`، وسجّل في `CHANGELOG.md`، وارفع على GitHub.

---

## ربط Notati

رابط MCP لكل مشروع فيه **توكن سري ومحصور بالمشروع**. البلوقن لا يكتبه أبداً في أي ملف أو commit أو تقرير. الإعداد يحفظ فقط: `key` · `project_id` · `project_name` · `phase_ids` · `statuses`.

### Claude Code (نطاق المشروع المحلي، خارج الريبو)

شغّل الأمر بنفسك داخل المشروع (في الجلسة اسبقه بـ `!`)، والصق رابطك مكان `<URL>`:

```bash
claude mcp add --transport http --scope local notati "<URL>"
```

يُحفظ في `~/.claude.json` تحت مسار المشروع، لا في الريبو. **لا تستخدم** `--scope project` (يكتب `.mcp.json` داخل الريبو). أعد فتح الجلسة بعد الإضافة.

### Codex (يدوي)

Codex لا يدعم تسجيل سيرفر لمشروع واحد من الأمر، ولا يقرأ متغيرات البيئة داخل `url`. الطريقة الآمنة لكل مشروع:

1. تأكد أن `.gitignore` في المشروع فيه `.codex/config.toml` (`start-init` يضيفه).
2. افتح `<المشروع>/.codex/config.toml` وأضف:
   ```toml
   [mcp_servers.notati]
   url = "<الصق رابط MCP هنا>"
   ```
3. المشروع لازم يكون **موثوقاً** في Codex (يسألك أول مرة).
4. افتح جلسة Codex جديدة.

**بديل عام:** `codex mcp add notati-<key> --url "<URL>"` يسجّله لكل المشاريع في `~/.codex/config.toml`. بعدها اكتب في إعداد المشروع `notati.server_codex: "notati-<key>"`.

---

## التحديث

| الخطوة | Claude Code | Codex |
|---|---|---|
| 1. تحديث الماركت بليس | `claude plugin marketplace update dev-kit-marketplace` | `codex plugin marketplace upgrade dev-kit-marketplace` |
| 2. تحديث البلوقن | `claude plugin update dev-kit@dev-kit-marketplace` | جلسة جديدة |
| 3. في كل مشروع | `/dev-kit:start-init` | `$dev-kit:start-init` |

الخطوة 3 تعرض فقط السكلز الجديدة في الكتالوج، أو اللي صارت مناسبة لأن نوع مشروع انضاف.

**للمطوّر:** ارفع `version` في `plugins/dev-kit/.claude-plugin/plugin.json` مع كل إصدار، وسجّله في `CHANGELOG.md`، واعمل tag. بدون رفع الرقم، مستخدمو Claude Code ما يستلمون التحديث.

### تغيير اسم البلوقن

الاسم `dev-kit` مكتوب في ملفين فقط، ومعه اسم المجلد `plugins/dev-kit/`: `.claude-plugin/marketplace.json` · `plugins/dev-kit/.claude-plugin/plugin.json`. السكلز والتمبلتس تقرأ الاسم من المانيفست وقت التشغيل.

---

## البنية

```
.claude-plugin/marketplace.json          ← ماركت بليس واحد يقرأه Claude Code وCodex
plugins/dev-kit/
├── .claude-plugin/plugin.json           ← المانيفست (Claude Code وCodex)
├── catalog/skills-catalog.json          ← مراجع السكلز الخارجية
├── catalog/README.md
└── skills/<name>/SKILL.md               ← المصدر الوحيد لكل سير عمل (تظهر كأوامر /dev-kit:<name>)
    └── start-init/templates/            ← قوالب ملفات المشروع
README.md · CHANGELOG.md · .gitignore
```

- **لماذا البلوقن في مجلد فرعي؟** Codex (`0.132`) لا يعرض بلوقن مصدره جذر الماركت بليس نفسه (`./`).
- **بدون `commands/`:** في Claude Code كل سكل تظهر كأمر `/dev-kit:<name>` لحالها. السكلز الخمس الرئيسية فيها `allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)` عشان تقرأ السكلز المشتركة والتمبلتس والكتالوج بدون طلب إذن.
