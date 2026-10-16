---
name: report-format
description: Shared final-report formats for the dev, tester, audit, identity and start-init skills — language and layout rules, one template per command, and the common "final check" table. Load it whenever one of those skills says so; not meant to be run on its own.
user-invocable: false
---

# report-format — the final report

## 1. Rules for every report

| Rule | Detail |
|---|---|
| Language | `language.report` from the config (default Arabic). Every message to the user uses it. |
| Conclusion first | First section is one line: the result in numbers. |
| Tables | Any pairing of a label with a term, value, status, path or number goes in a table. |
| Isolate technical tokens | Code, paths, IDs, commands, error codes in `backticks`, ideally in their own table cell — never buried mid-sentence (RTL readability). |
| Number + unit | One token: `5 تاسكات`, `120 ms`. |
| Short | Bullets with one idea each; no long paragraphs. |
| Evidence | **Never claim anything without evidence** from this session. Unverified = say "غير متحقق". |
| Secrets | Never include a Notati URL or any token. |
| User preferences | If the user's own instructions define a report style, follow them on top of this. |

## 2. Common "final check" table (every command that touched code)

```
## الفحص النهائي
| الفحص | النتيجة |
|---|---|
| `<verify.analyze>` | ✅ / ❌ / غير مُعدّ |
| `<verify.test>` | X passed / Y failed |
| `<verify.format>` | ✅ / ❌ / غير مُعدّ |
| Notati | متزامن / غير متاح / غير مفعّل |
```

Show the actual configured command in the first column.

## 3. Templates

### dev
```
## الخلاصة
سطر واحد: كم تاسك اتنفذ من كم، والحالة العامة.

## اللي خلص (قيد المراجعة)
| التاسك | Notati | اللي اتعمل | الإثبات |

## اللي ما خلصش
| التاسك | الحالة | السبب |

## محتاج منك
- القرارات المعلقة (من الذاكرة القسم 4).

## أخطاء اتعلمت منها
- عدد الدروس الجديدة في `docs/LESSONS.md` وأهمها.

## الفحص النهائي
(الجدول المشترك)

## الخطوة الجاية
- شغّل `tester` لمراجعة التاسكات.
```

### tester
```
## الخلاصة
سطر واحد: كم تاسك اتراجع، كم اتقفل، كم اتصلح، كم متوقف.

## اتقفلت سليمة
| التاسك | Notati | الاختبارات | الإثبات |

## اتصلحت ثم اتقفلت
| التاسك | الخطأ | الإصلاح | الإثبات |

## ما اتقفلتش
| التاسك | الحالة | السبب |

## محتاج منك
- ...

## الفحص النهائي
(الجدول المشترك)

## الخطوة الجاية
- شغّل `audit` على التاسكات المكتملة، أو `dev` للمرحلة التالية.
```

### audit
```
## الخلاصة
سطر واحد: كم تاسك اتدقق، كم ثغرة اتلقت واتصلحت، وأي شيء حرج معلق.

## ثغرات حرجة معلقة (لو فيه — أول شيء)
| الثغرة | التاسك | محتاج منك |

## الثغرات المصلحة
| الرقم | التاسك | المحور | الخطورة | المشكلة | الإثبات قبل / بعد |

## اشتباه غير مؤكد
- ...

## الفحص النهائي
(الجدول المشترك)
```

### identity
```
## الخلاصة
سطر واحد: الهوية اللي اتعتمدت، وإيش اللي اتحدث.

## الهوية المعتمدة
| العنصر | القيمة |
| اللون الأساسي | `#......` |
| الخط | ... (الرخصة) |

## الإثبات
| الفحص | النتيجة |
| أقل تباين للنص | 7.1:1 ✅ |
| تمييز ألوان الحالات لعمى الألوان | ✅ |
| أرقام جدولية `tnum` | ✅ |

## الملفات اللي اتحدثت
- ...

## الخطوة الجاية
- ...
```

### start-init
```
## الخلاصة
سطر واحد: كم ملف أُنشئ، كم موجود، وحالة ربط Notati.

## الملفات
| الملف | الحالة |   ← موجود / أُنشئ / يحتاج مراجعة

## الإعداد
| الحقل | القيمة |

## Notati
| البند | القيمة |   ← المشروع، المفتاح، عدد المراحل، حالة التسجيل

## يحتاج منك
- ...

## الخطوة الجاية
- شغّل `dev P0`.
```

### Stop warnings
A guard stop (missing config, Notati project mismatch) is the **first line** of the reply, prefixed with `⚠️`, followed only by what the user must do.
