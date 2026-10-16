# maqraa-data — مستودع بيانات «مقرأة القرآن الكريم»

هذا المستودع هو **المصدر الوحيد** لبيانات التطبيق: السور، والأجزاء، والأحزاب والأرباع، والصفحات، والتفاسير، والخطوط.
يُبنى التطبيق منه وقت البناء، فتصير البيانات **داخل البرنامج** ويعمل دون إنترنت، ولا يبقى خارجيًا إلا **روابط الصوتيات**.

## هيكل المستودع

```
maqraa-data/
├─ README.md                 هذا الملف
├─ LICENSES.md               تراخيص المصادر (اقرأه قبل التوزيع)
├─ GITHUB_SETUP_AR.md        دليل رفع المستودع على حسابك خطوة بخطوة
├─ manifest.json             قائمة الملفات وبصماتها ورقم إصدار البيانات (يُولَّد)
├─ meta/                     «الأماكن»: ملفات صغيرة
│   ├─ chapters.json         114 سورة: الأسماء، عدد الآيات، صفحة البداية والنهاية
│   ├─ juz.json              30 جزءًا: أول آية وصفحتها
│   ├─ hizb_rub.json         240 ربعًا (تشمل الأحزاب الستين): أول آية وصفحتها
│   ├─ pages.json            604 صفحة: أول آية في كل صفحة وجزؤها
│   ├─ sajdas.json           15 موضع سجدة تلاوة
│   └─ info.json             وصف البيانات ومصدرها
├─ tafsir/                   ستة تفاسير، ملف لكل تفسير
├─ fonts/
│   ├─ ui/                   خطوط الواجهة (OFL): Amiri وAmiri Quran وScheherazade New
│   └─ qcf4/                 خطوط مصحف المدينة QCF4 (تُنسخ من جهازك بأداة)
├─ mushaf/                   qcf4_pages.json: صفحات المصحف برموز الخط (يُولَّد بأداة)
├─ docs/VERIFY_CHECKLIST_AR.md   قائمة لمطابقة الأرقام مع مصحفك المطبوع
└─ tools/                    أدوات التوليد والتحقق
```

## ما الموجود جاهزًا وما الذي تولّده أنت

| الملف/المجلد | الحالة | كيف يُنشأ | المصدر |
|---|---|---|---|
| `meta/*.json` | ✅ جاهز | `node tools/build_meta.mjs` ثم `python tools/apply_pagination.py ...` | quran-meta (MIT) + ترقيم QCF4 |
| `tafsir/*.json` (6 ملفات، نحو 17 ميغابايت) | ✅ جاهز | `python tools/build_tafsir.py` | tafsir_api (MIT) |
| `fonts/ui/` (5 خطوط + 3 تراخيص) | ✅ جاهز | ملفات جاهزة | Google Fonts (OFL) |
| `mushaf/qcf4_pages.json` (نحو 8 ميغابايت) | ⏳ **تولّده أنت** | `python tools/build_pages.py` | quran-qcf4 (راجع LICENSES.md) |
| `fonts/qcf4/` (48 خطًا، نحو 100 ميغابايت) | ⏳ **تنسخه أنت** | `python tools/collect_fonts.py` | خطوطك من مجمّع الملك فهد |
| `manifest.json` | ⏳ يُولَّد بعد اكتمال الباقي | `python tools/make_manifest.py --version 2026.10.1` | — |

**لماذا لا يأتي `qcf4_pages.json` والخطوط جاهزة؟** لأن ترخيصها غير واضح (لا ملف ترخيص، ويُطلب احترام شروط المجمّع). القرار بتوزيعها قرارك أنت. انظر `LICENSES.md`.

## صيغة الملفات (أمثلة)

`meta/chapters.json`
```json
{ "id": 2, "name_ar": "البَقَرَة", "name_plain": "البقرة", "name_en": "Al-Baqarah", "meaning_en": "The Cow",
  "name_fr": "...", "meaning_fr": "...", "ayahs": 286, "meccan": false, "ruku": 40,
  "revelation_order": 87, "page_start": 2, "page_end": 49 }
```
`meta/juz.json`  ←  `{ "juz": 7, "surah": 5, "ayah": 82, "page": 121 }`

`meta/hizb_rub.json`  ←  `{ "rub": 9, "hizb": 3, "quarter": 1, "juz": 2, "surah": 2, "ayah": 142, "page": 22 }`
(الحزب = 4 أرباع، والجزء = 8 أرباع؛ فبداية كل حزب هي الربع الأول منه)

`meta/pages.json`  ←  `{ "page": 42, "surah": 2, "ayah": 253, "juz": 3, "continues_prev": false }`

`tafsir/<slug>.json`
```json
{ "edition": "ar-tafsir-muyassar", "label": "التفسير الميسر", "lang": "ar",
  "source": "https://github.com/spa5k/tafsir_api", "ayahs": { "1:1": "...", "2:255": "..." } }
```

## ترتيب العمل (كل مرة تحدّث فيها)
```
pip install requests
python tools/build_pages.py                                   # (مرة واحدة) صفحات المصحف
python tools/apply_pagination.py --pages-file mushaf/qcf4_pages.json
python tools/collect_fonts.py                                 # الخطوط من جهازك
python tools/validate.py --pages-file mushaf/qcf4_pages.json
python tools/make_manifest.py --version 2026.10.2
python tools/validate.py --manifest
```
(يقبل `validate.py` أيضًا `--pages-dir` لمجلد صفحات خام.)

## رقم إصدار البيانات
صيغته `السنة.الشهر.رقم` (مثل `2026.10.1`). يُكتب في `manifest.json`، ويُرفع كلما تغيّر أي ملف بيانات.

## الخصوصية
يُفضَّل أن يكون المستودع **خاصًا (Private)** خصوصًا إن ضممتَ خطوط QCF4. التطبيق لا يقرأ من GitHub وقت التشغيل؛ البيانات مضمّنة فيه.
