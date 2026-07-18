# MD-automation

مركز الأتمتة الخاص بمنظومة MD1 (mdm1.org و md1usd.com): بناء/نشر المواقع عبر GitHub Pages،
نشر تلقائي على تيليجرام، مولّد صفحات دوري، وسكربتات النمو التجريبية.

## البنية

```
.github/
  dependabot.yml            # تحديث أسبوعي لإصدارات actions و pip
  workflows/
    build.yml                # يشغّل reusable-build.yml عند كل push/PR على main
    deploy.yml                # يشغّل reusable-deploy.yml بعد نجاح Build
    auto-fix.yml              # يشغّل reusable-autofix.yml عند فتح/تصنيف issue بـ build-failed
    watchdog.yml              # يشغّل reusable-watchdog.yml كل ساعة
    publish.yml                # يشغّل reusable-publish.yml يدويًا (نشر من طابور محتوى عام)
    telegram-publish.yml       # ينشر يوميًا عبر social/telegram/bot.py
    generate-pages.yml         # يشغّل page-generator/build-pages.js كل ساعة
    pipeline.yml                # Master Pipeline: يفحص بناء mdm1.org و MD1usd.com دوريًا
    codeql.yml                  # فحص أمان الكود (CodeQL)
    reusable-*.yml               # ووركفلوهات قابلة لإعادة الاستخدام، يمكن لريبوهات
                                   # المواقع (mdm1.org / md1usd.com) استدعاءها مباشرة
                                   # عبر: uses: md1god/MD-automation/.github/workflows/<name>@main

social/
  requirements.txt
  stats.py                      # تسجيل إحصائيات النشر
  telegram/
    bot.py                       # ينشر منشور واحد بالترتيب من posts_sequence.py على القناة
    config.py                    # اسم القناة والتأخير بين المنشورات
    posts_sequence.py            # قائمة/تسلسل المنشورات

page-generator/                # (منقول من MD-robot) مولّد صفحات HTML دوري لموقع MD1usd
  build-pages.js
  build-state.json
  pages-index.json
  content-library/*.json        # عناصر المحتوى، كل عنصر بملف JSON مستقل
  pages/                        # الصفحات المولدة

growth-engines/                # سكربتات تجريبية غير موصولة بأي workflow حاليًا (Message Engine)
  global_growth.py
  global_growth_status.json
  global_humanitarian_growth.py
  global_multilingual_growth.py
  silent_growth_engine.py
```

## أهم الإصلاحات في هذه النسخة

- تم حذف مجلد `DiDo` (نسخة كاملة عن هذا الريبو نفسه — بما فيها ملفات `.git` الداخلية —
  كانت متسربة بالغلط داخل مستودع MD-robot). كان هذا هو مصدر التكرار الأكبر.
- تم حذف ملفين زائدين من `.github/workflows`: `yml.reusable` (نسخة قديمة من
  `reusable-build.yml`) و `.ymlreusable-deploy` (ملف فارغ/تالف الاسم).
- تم توحيد `Deploy.yml` / `deploy.yml` (كانا موجودين بنفس المحتوى وباختلاف حالة الأحرف فقط)
  في ملف واحد `deploy.yml`.
- تم اعتماد `page-generator` من **أحدث نسخة حية** لمنطق البناء (نظام queue + content-library)
  بدل النسخة الأقدم (`pages-data.json` + عشوائية بسيطة) الموجودة في `files.zip`، والتي تم إسقاطها
  لأنها نموذج أولي متجاوَز.
- تم تحديث `generate-pages.yml` ليعمل داخل `page-generator/` بعد نقل الملفات لمجلدها الجديد،
  وأضفت خطوة "فحص وجود تغييرات فعلية" قبل `commit` حتى لا يفشل الـ push على "nothing to commit".
- تم توحيد إصدار `actions/checkout` على `v6` في كل الووركفلوهات (كانت مبعثرة بين v4 و v6).
- كل ملفات YAML تم فحصها آليًا (`yaml.safe_load`) وكلها سليمة الصياغة.
- `page-generator/build-pages.js` تم تشغيله فعليًا للتأكد أنه يولّد صفحة جديدة بدون أخطاء،
  ثم أعدت الحالة (`build-state.json`, `pages-index.json`, `pages/`) لوضعها الأصلي.
- كل سكربتات Python (`social/`, `growth-engines/`) فُحصت بـ `py_compile` وسليمة الصياغة.

## ملاحظة تحتاج قرارك

يوجد نظاما نشر منفصلان لتيليجرام:
1. `telegram-publish.yml` + `social/telegram/bot.py` — ينشر منشورًا واحدًا يوميًا من
   `posts_sequence.py` على قناة ثابتة (`CHANNEL_USERNAME`)، عبر `TELEGRAM_BOT_TOKEN`.
2. `publish.yml` → `reusable-publish.yml` — نظام عام أبسط (طابور ملفات `.txt` في
   `content-queue/`)، عبر `BOT_TOKEN` و `CHAT_ID`، يُشغَّل يدويًا فقط.

هذا ليس تكرارًا حرفيًا (المنطق مختلف) لكنه تراكب في الغرض. أبقيتهما كما هما لأن حذف أحدهما
قرار يخصك، لكن يفضّل تحدد أيهما الفعلي المستخدم لتوحيد السيكريتات (`TELEGRAM_BOT_TOKEN` مقابل
`BOT_TOKEN`/`CHAT_ID`) وتحذف الثاني لاحقًا.

## قبل الرفع على GitHub

1. أنشئ الريبو الجديد (أو صفّر نفس `md1god/MD-automation` الحالي).
2. تأكد من إضافة الـ Secrets المطلوبة: `TELEGRAM_BOT_TOKEN`, `BOT_TOKEN`, `CHAT_ID`, `SITES_PAT`.
3. فعّل GitHub Pages بمصدر "GitHub Actions" قبل أول تشغيل لـ `deploy.yml` (وإلا سيفشل لعدم
   وجود environment باسم `github-pages`).
4. `git push` عادي — كل الووركفلوهات ستُفعّل تلقائيًا حسب جدولها.
