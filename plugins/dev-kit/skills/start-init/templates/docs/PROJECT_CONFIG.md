# إعداد المشروع — {{PROJECT_NAME}}

> **للوكيل:** كل الكوماندز (`dev` و`tester` و`audit` و`identity`) تقرأ هذا الملف **قبل أي شيء**. كل قيمة خاصة بالمشروع تعيش هنا فقط، لا داخل البلوقن.
> **قواعد التحديث:** يُنشئه `start-init` ويكمّله. عدّله يدوياً أو بإعادة تشغيل `start-init`. **ممنوع** كتابة رابط Notati أو أي توكن هنا.
> **الصيغة:** كتلة `yaml` واحدة هي المصدر. الشرح تحتها للقراءة فقط.

```yaml
config_version: 1

project:
  name: "{{PROJECT_NAME}}"
  description: "{{PROJECT_DESCRIPTION}}"
  stack: "{{STACK}}"                 # مثال: "node", "python", "dart", "odoo"
  related_repos: []                  # مسارات ريبوهات مرتبطة تقرأ نفس الوثائق (اختياري)

language:
  docs: "{{DOCS_LANGUAGE}}"          # ar افتراضياً
  report: "{{REPORT_LANGUAGE}}"      # ar افتراضياً
  code: "en"
  commits: "en"

phases:
  prefix: "P"                        # رقم التاسك = <prefix><رقم المرحلة>-<3 خانات>، مثال: P0-001
  index: "docs/phases/00-index.md"
  map: "docs/phases/notati-map.md"
  non_coding_label: "غير برمجي"      # تاسك بهذا الليبل لا يُنفَّذ؛ تُجهَّز مواده فقط
  decision_gates: []                 # تاسكات بوابة قرار: يُجهَّز تقريرها فقط ولا يُقرَّر فيها. مثال: ["P2-004"]

# الحالات: صف لكل حالة في Notati. file = النص الحرفي في ملف المرحلة.
# role: not_started | in_progress | review | done | blocked | other
statuses:
  - { file: "لم يبدأ",      notati: "todo",        role: not_started }
  - { file: "قيد العمل",    notati: "in_progress", role: in_progress }
  - { file: "قيد المراجعة", notati: "review",      role: review }
  - { file: "مكتمل",        notati: "done",        role: done }
  - { file: "متوقف",        notati: "blocked",     role: blocked }

notati:
  enabled: {{NOTATI_ENABLED}}        # true / false
  server: "notati"                   # اسم سيرفر MCP المسجَّل محلياً في Claude Code (لا الرابط)
  server_codex: ""                   # اسمه في Codex لو يختلف (مثلاً "notati-<key>" عند التسجيل العام)
  project_name: "{{NOTATI_PROJECT_NAME}}"
  key: "{{NOTATI_KEY}}"              # مفتاح المشروع، مثال: "AB"
  project_id: "{{NOTATI_PROJECT_ID}}"
  phase_ids: {}                      # مثال: { P0: "abc123" }
  ignored_tasks: []                  # تاسكات مكررة أو ملغاة لا تُحدَّث أبداً. مثال: ["AB-12"]

verify:
  analyze: "{{VERIFY_ANALYZE}}"      # فحص ثابت للمشروع كله
  test: "{{VERIFY_TEST}}"            # كل الاختبارات
  test_path: "{{VERIFY_TEST_PATH}}"  # اختبارات جزء واحد؛ {path} يُستبدل بالمسار
  format: "{{VERIFY_FORMAT}}"        # فحص التنسيق (يفشل لو فيه تغيير)
  extra: []                          # فحوص إضافية: - { name: "", command: "", when: "" }

code_rules: "AGENTS.md#5"            # مكان القواعد التي لا تُكسر

stack_skills: []                     # جدول "Skills & tools to use" الخاص بالستاك
#  - { trigger: "كتابة اختبارات", use: "اسم-السكل", commands: [dev, tester] }

ux_gate:
  enabled: {{UX_GATE_ENABLED}}       # true لو المشروع فيه شاشات
  design_reference: ""               # مرجع بصري فقط، مثال: "docs/demo.html"
  ux_rules: ""                       # ملف/قسم قواعد تجربة الاستخدام وميزانية الخطوات
  page_template: ""                  # قالب الصفحات الداخلية إن وجد
  accessibility_skill: ""            # سكل فحص الوصول للستاك، مثال: "<stack>-accessibility" (سكل مضمّنة أو من بلوقن الستاك)
  i18n_skill: ""                     # سكل الترجمة والاتجاه للستاك
  token_file: ""                     # ملف التوكنز الوحيد المسموح فيه ألوان وخطوط حرفية
  hardcoded_style_patterns: []       # أنماط Grep للألوان/الخطوط الحرفية خارج ملف التوكنز
  checks: [accessibility, step_budget, shortcuts, rtl_strings, tokens_only, heuristic_review]

audit:
  enabled: {{AUDIT_ENABLED}}
  performance_budget: ""             # ملف/قسم ميزانية الأداء الملزمة
  references: []                     # وثائق إضافية يقرأها audit (عقود، قاعدة البيانات...)
  extra_axes: []                     # نقاط تدقيق خاصة بالمشروع
#  - { axis: "حسابات", point: "", how: "" }
  severity_examples: {}              # أمثلة خاصة بالمشروع لكل خطورة (اختياري)

identity:
  enabled: false
  product_context: ""                # المنتج والجمهور والشعار واللغة
  key_screen: ""                     # الشاشة التي تُعرض عليها الاتجاهات
  design_reference: ""               # ملف المرجع البصري الحالي (ألوان مؤقتة)
  token_task: ""                     # التاسك المسؤول عن ملف التوكنز
  token_file: ""
  output: "docs/design/identity.md"
  options_file: "docs/design/identity-options.md"
  preview_file: "docs/design/identity-preview.html"
  constraints: []                    # قيود إضافية فوق القيود العامة

measurements:
  dir: "docs/measurements"           # تقارير tester-all / audit-all عند التشغيل على كل المشروع

memory:
  current_state_max_lines: 15
  session_log_limit: 10
  decisions_archive_threshold: 40
```

## شرح الحقول

| الحقل | المعنى |
|---|---|
| `statuses` | ربط كل حالة في Notati بنص حرفي في ملف المرحلة. الكوماندز تستخدم `role` لا النص. حالة إضافية (مثل "مؤجل") تُضاف بـ `role: other` |
| `notati.key` و`notati.project_id` | يتحقق منهما كل كوماند عبر `get_context` قبل أي كتابة |
| `notati.ignored_tasks` | تاسكات لا تُحدَّث أبداً (مكررة أو ملغاة) |
| `verify.*` | أوامر التحقق. أمر فاضي = "غير مُعدّ" ويُذكر كذلك في سطر الإثبات |
| `phases.decision_gates` | تاسكات بوابة قرار: تقرير فقط، القرار لصاحب المشروع |
| `stack_skills` | السكلز الخاصة بالستاك ومتى تُستخدم |
| `ux_gate` | بوابة تجربة الاستخدام في `tester` وقراءات الشاشات في `dev` |
| `audit.extra_axes` | محاور تدقيق خاصة فوق المحاور العامة (أمان، حسابات، أداء) |
| `identity` | إعداد كوماند الهوية البصرية |
