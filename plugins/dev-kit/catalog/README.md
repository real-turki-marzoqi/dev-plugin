# كتالوج السكلز الخارجية

**الخلاصة:** `skills-catalog.json` قائمة مراجع لسكلز خارجية موثوقة مع أوامر تثبيتها. **البلوقن لا ينسخ أي سكل خارجية داخله**؛ `start-init` يقرأ الكتالوج، يصفّيه حسب نوع مشروعك، ويثبّت اللي توافق عليه فقط.

---

## كيف يشتغل

1. **`start-init`** يقترح أنواع المشروع من الفحص (`web` · `mobile` · `desktop` · `backend` · `odoo`) وتأكدها أنت.
2. يصفّي الكتالوج: كل سكل `project_types` حقها فيها `all` أو نوع من أنواع مشروعك.
3. يفحص المثبّت عندك (`claude plugin list --json` و`codex plugin list` ومجلدات السكلز في المشروع).
4. يعرض الناقص في جدول (الاسم، الغرض، المصدر، الرخصة) ويسألك: **تثبيت الكل / اختيار / تخطي**. لا شيء يتثبت بدون موافقتك.
5. يثبّت:

   | النوع | النطاق |
   |---|---|
   | `plugin` | على مستوى المستخدم، مرة وحدة للجهاز |
   | `npx-skills` / `manual` | على مستوى المشروع فقط |

6. يفعّل البلوقنز المختارة **لهذا المشروع فقط** (`.claude/settings.json` ← `enabledPlugins`، و`.codex/config.toml` ← `[plugins."<id>"]`)، ويطفّي بلوقنز الكتالوج الثانية فيه، عشان سكلز الموبايل ما تشتغل في مشروع ويب.
7. يكتب في `docs/PROJECT_CONFIG.md`: `project_types` و`stack_skills.<الكوماند>` و`skills_catalog.reviewed`.
8. إعادة التشغيل تعرض **الجديد فقط**: سكلز انضافت للكتالوج، أو سكلز صارت مناسبة لأن نوع مشروع انضاف.

## الحقول

| الحقل | المعنى |
|---|---|
| `id` | معرّف ثابت في الكتالوج (يُحفظ في `skills_catalog.reviewed`) |
| `purpose_ar` | الغرض بسطر عربي |
| `source_repo` | `owner/repo` على GitHub |
| `install_type` | `plugin` · `npx-skills` · `manual` (لـ Claude Code) |
| `plugin_id` | `<plugin>@<marketplace>` لنوع `plugin`؛ يُستخدم للفحص والتفعيل |
| `marketplace_repo` | ريبو الماركت بليس (لـ `extraKnownMarketplaces`)؛ `null` للماركت بليس الرسمي |
| `skills` | أسماء السكلز داخل المصدر، و`invoke` = الاسم اللي يُكتب في `stack_skills` |
| `claude_install` | أمر التثبيت في Claude Code كما هو |
| `codex_install` | أمر التثبيت في Codex كما هو |
| `project_types` | `web` · `mobile` · `desktop` · `backend` · `odoo` · `all` |
| `used_by` | الكوماندز اللي تستخدمها |
| `stage` | وصف المرحلة بالعربي |
| `stages` | المرحلة لكل كوماند (المفتاح اللي تقرأه الكوماندز) |
| `license` | SPDX من الريبو |
| `verified_on` | تاريخ التحقق: الريبو موجود، والرخصة، وأمر التثبيت مطابق لـ README/المانيفست. **لم يُشغَّل الأمر** |
| `notes_ar` | ملاحظات (اختياري) |

### المراحل المعتمدة

| الكوماند | المراحل |
|---|---|
| `dev` | `تخطيط` · `تنفيذ` · `شاشات` |
| `tester` | `تصحيح الأخطاء` · `بوابة UX` · `اختبار` |
| `audit` | `تدقيق أمني` · `روائح الكود` · `تحقق من الإصلاح` |
| `identity` | `هوية بصرية` |

## إضافة سكل جديدة

1. تحقق: الريبو موجود ومُصان، **الرخصة واضحة** (بدون رخصة = لا تُضاف)، وأمر التثبيت من README أو المانيفست.
2. أضف كائناً في `skills` داخل `skills-catalog.json` بكل الحقول، و`verified_on` بتاريخ اليوم. مرحلة جديدة غير موجودة في الجدول فوق تحتاج تعديل الكوماند المعني.
3. حدّث `updated` في رأس الملف.
4. تأكد إن الملف JSON صالح: `python -c "import json; json.load(open('plugins/dev-kit/catalog/skills-catalog.json', encoding='utf-8'))"`.
5. ارفع `version` في `plugins/dev-kit/.claude-plugin/plugin.json` وسجّل التغيير في `CHANGELOG.md`، ثم ارفع على GitHub.
6. في المشاريع: `plugin marketplace update` ثم `start-init` يعرض السكل الجديدة فقط.
