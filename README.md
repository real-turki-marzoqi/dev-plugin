# dev-kit

بلوقن واحد يشتغل على **Claude Code** و**Codex**: سير عمل بالمراحل (`dev` · `tester` · `audit` · `identity`) فوق ذاكرة مشروع مشتركة داخل الريبو، مع مزامنة لوحة المهام **Notati**، و`start-init` اللي يجهّز أي مشروع جديد.

**الفكرة الأساسية:** البلوقن عام، والمشروع إعداد. كل قيمة خاصة بمشروعك (أوامر التحقق، مفتاح Notati، الحالات، البوابات، محاور التدقيق) تعيش في `docs/PROJECT_CONFIG.md` داخل المشروع، والسكلز تقرأ منه.

---

## المحتويات

| الكوماند | الوظيفة | ينقل الحالة |
|---|---|---|
| `start-init` | يجهّز الذاكرة والإعداد وربط Notati. **لا يمسح شيئاً**؛ يكمّل الناقص فقط | — |
| `dev <المرحلة>` | ينفذ تاسكات مرحلة. سؤالين كحد أقصى في البداية، ثم لا يتوقف | لم يبدأ ← قيد العمل ← قيد المراجعة |
| `tester [مرحلة/تاسك]` | يختبر التاسكات قيد المراجعة باستقلال، يصلح المعيب، ويقفل السليم | قيد المراجعة ← مكتمل |
| `audit [مرحلة/تاسك]` | يدقق المكتمل: أمان، حسابات، أداء + محاور المشروع؛ يثبت كل ثغرة ويصلحها | يبقى مكتمل (أو متوقف) |
| `identity [تلميحات]` | 3 اتجاهات هوية بصرية مقاسة بالأرقام، وصاحب المشروع يختار | — |

### سكلز مشتركة (لا تُستدعى مباشرة)

| السكل | المحتوى |
|---|---|
| `memory-protocol` | ترتيب القراءة، قواعد كل ملف وكل قسم في الذاكرة، الدروس، صيغ أسطر الإثبات، نهاية الجلسة |
| `notati-sync` | فحص المشروع، جدول الأدوات، ربط الحالات، المزامنة، إيجاد المشروع وتسجيله بدون تسريب الرابط |
| `verify` | أوامر التحقق من الإعداد، "الأحمر = غير منتهٍ"، منع تخطي الاختبارات |
| `report-format` | هيكل التقرير النهائي لكل كوماند |
| `codex-adapter` | ترجمة أدوات Claude ← Codex، وبديل HTTP لـ Notati |

---

## التثبيت

> استبدل `<owner>/<repo>` باسم ريبو GitHub بعد الرفع.

### Claude Code

داخل الجلسة:

```
/plugin marketplace add <owner>/<repo>
/plugin install dev-kit@dev-kit-marketplace
```

أو من الطرفية:

```bash
claude plugin marketplace add <owner>/<repo>
claude plugin install dev-kit@dev-kit-marketplace
```

من نسخة محلية (للتطوير):

```bash
claude plugin marketplace add "/path/to/dev plugin"   # مجلد الريبو، لا plugins/dev-kit
claude plugin install dev-kit@dev-kit-marketplace
```

### Codex

```bash
codex plugin marketplace add <owner>/<repo>
codex plugin add dev-kit@dev-kit-marketplace
```

أو من واجهة Codex: `/plugins` ثم فعّل `dev-kit`. **ابدأ جلسة جديدة** بعد التثبيت عشان تظهر السكلز.

> البلوقنز غير متاحة في إضافة Codex للـ IDE حسب توثيق OpenAI؛ استخدم Codex CLI أو التطبيق.

---

## الاستخدام

| الخطوة | Claude Code | Codex |
|---|---|---|
| تجهيز المشروع | `/dev-kit:start-init` | `$dev-kit:start-init` |
| تنفيذ مرحلة | `/dev-kit:dev P0` | `$dev-kit:dev P0` |
| الاختبار | `/dev-kit:tester` | `$dev-kit:tester` |
| التدقيق | `/dev-kit:audit P0` | `$dev-kit:audit P0` |
| الهوية البصرية | `/dev-kit:identity` | `$dev-kit:identity` |

- أي كوماند ما يلقى `docs/PROJECT_CONFIG.md` يوقف ويطلب تشغيل `start-init` أول.
- أي كوماند يبدأ بفحص أن مشروع Notati المتصل يطابق الإعداد. عدم التطابق = توقف قبل أي كتابة.
- لو كتبت `/dev` نصاً في Codex، يستخدم السكل المقابلة بنفس المعنى.

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
```

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

**بديل عام:** `codex mcp add notati-<key> --url "<URL>"` يسجّله لكل المشاريع في `~/.codex/config.toml`. بعدها اكتب في إعداد المشروع `notati.server_codex: "notati-<key>"` حتى يتكلم كل مشروع مع سيرفره فقط.

---

## التحديث

| الأداة | الأمر |
|---|---|
| Claude Code | `claude plugin marketplace update dev-kit-marketplace` ثم `claude plugin update dev-kit@dev-kit-marketplace` |
| Codex | `codex plugin marketplace upgrade dev-kit-marketplace` ثم جلسة جديدة |

**للمطوّر:** ارفع `version` في `.claude-plugin/plugin.json` و`.codex-plugin/plugin.json` مع كل إصدار، وسجّله في `CHANGELOG.md`. بدون رفع الرقم، مستخدمو Claude Code ما يستلمون التحديث.

### تغيير اسم البلوقن

الاسم `dev-kit` مكتوب في 4 ملفات فقط، ومعه اسم المجلد `plugins/dev-kit/`: `.claude-plugin/marketplace.json` · `.agents/plugins/marketplace.json` · `plugins/dev-kit/.claude-plugin/plugin.json` · `plugins/dev-kit/.codex-plugin/plugin.json`. السكلز والتمبلتس تقرأ الاسم من المانيفست وقت التشغيل.

---

## السكلز المضمّنة (من مصدر خارجي)

| السكل | المصدر | الرخصة |
|---|---|---|
| `flutter-accessibility` | [VeryGoodOpenSource/vgv-ai-flutter-plugin](https://github.com/VeryGoodOpenSource/vgv-ai-flutter-plugin) `skills/accessibility` | MIT — `LICENSE-vgv.txt` |
| `flutter-internationalization` | نفس الريبو، `skills/internationalization` | MIT — `LICENSE-vgv.txt` |
| `flutter-ui-package` | نفس الريبو، `skills/ui-package` | MIT — `LICENSE-vgv.txt` |

- منسوخة كما هي؛ التغيير الوحيد إضافة البادئة `flutter-` لحقل `name:`. نص رخصة MIT موجود في جذر الريبو وداخل كل مجلد.
- سكلز خاصة بستاك معيّن: تُستخدم فقط لما يذكرها مشروعك في `stack_skills` أو `ux_gate` داخل `docs/PROJECT_CONFIG.md`.

## بلوقنز مقترحة (اختيارية)

الكوماندز تشتغل بدونها؛ لو مثبتة تُستخدم تلقائياً حسب جداول السكلز.

| الغرض | البلوقن المقترح | يستخدمه |
|---|---|---|
| فحص أمني شامل | `claude-security` (`anthropics/claude-plugins-official`) | `audit` |
| مراجعة التغييرات | `differential-review` (`trailofbits/skills`) | `audit` |
| واجهات خطرة وإعدادات | `sharp-edges` (`trailofbits/skills`) | `audit` |
| بناء السياق قبل التدقيق | `audit-context-building` (`trailofbits/skills`) | `audit` |
| سلسلة الاعتماديات | `supply-chain-risk-auditor` (`trailofbits/skills`) | `audit` |
| نقد التصميم | `design-skills` (`design-critique`) | `tester` (بوابة UX) |
| بيانات تصميم وألوان وخطوط | `ui-ux-pro-max` (`nextlevelbuilder/ui-ux-pro-max-skill`) | `identity` |
| أدوات الستاك (تحليل، اختبار) | بلوقن الستاك الرسمي لمشروعك | `dev` · `tester` · `audit` عبر `stack_skills` |

---

## البنية

```
.claude-plugin/marketplace.json          ← ماركت بليس Claude Code (بلوقن واحد)
.agents/plugins/marketplace.json         ← ماركت بليس Codex
plugins/dev-kit/
├── .claude-plugin/plugin.json           ← مانيفست Claude Code
├── .codex-plugin/plugin.json            ← مانيفست Codex
├── commands/                            ← مداخل Claude Code: كل ملف يحيل للسكل المقابلة
├── skills/<name>/SKILL.md               ← المصدر الوحيد لكل سير عمل
├── skills/start-init/templates/         ← قوالب ملفات المشروع
└── LICENSE-vgv.txt
README.md · CHANGELOG.md · LICENSE-vgv.txt
```

- **لماذا البلوقن في مجلد فرعي؟** Codex (`0.132`) لا يعرض بلوقن مصدره جذر الماركت بليس نفسه (`./`). المجلد الفرعي يشتغل على Claude Code وCodex معاً.
- **مصدر واحد للحقيقة:** منطق كل كوماند في `skills/<name>/SKILL.md` فقط. ملفات `commands/` مداخل لـ Claude Code، تعطي إذن قراءة مجلد البلوقن (`allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)`) وتحيل للسكل. Codex يتجاهلها (فيها `$ARGUMENTS`) ويستخدم السكلز مباشرة.
- **السكلز الرئيسية `user-invocable: false`** في Claude Code حتى لا تتكرر في قائمة `/` مع الكوماندز بنفس الاسم.
