# رفع مستودع البيانات على حسابك في GitHub (خطوة بخطوة)

> أسماء الأزرار في GitHub وGitHub Desktop تتغير أحيانًا. ما كتبتُه هنا من معرفتي ولم أجرّبه على حسابك،
> فإن اختلف شيء فاتبع المعنى وراجع https://docs.github.com

## 0) قرارات قبل أن تبدأ
1. **خاص أم عام؟** أنصح بـ **Private** (خاص). السبب: خطوط QCF4 وبيانات صفحاتها ترخيصها غير مؤكد (انظر `LICENSES.md`).
   حسابات GitHub المجانية تسمح بمستودعات خاصة (حسب علمي).
2. **اسم المستودع:** `maqraa-data` (أو ما تشاء).
3. **ماذا يدخل المستودع؟** كل محتويات هذا المجلد. أما `fonts/qcf4/` و`mushaf/qcf4_pages.json` فتولّدهما أنت (القسم 3) ثم تضيفهما إن قررتَ ذلك.

## 1) تجهيز جهازك (مرة واحدة)
- **Python 3.10 أو أحدث** (python.org). وبعده في الطرفية: `pip install requests`
- **Node.js** (nodejs.org) — لازم فقط إن أردتَ إعادة توليد `meta/` من الصفر. ملفات `meta/` جاهزة، فيمكنك تأجيله.
- **GitHub Desktop** (desktop.github.com) — أسهل الطرق. وبديله الويب أو أوامر git (انظر 2).

## 2) إنشاء المستودع ورفعه

### الطريقة أ: GitHub Desktop (الأسهل، وهي الموصى بها)
1. فك ضغط `maqraa-data.zip` في مكان ثابت، مثل `D:\maqraa\maqraa-data`.
2. افتح GitHub Desktop وسجّل الدخول بحسابك.
3. من القائمة: **File ← Add local repository…** واختر مجلد `maqraa-data`.
   سيقول لك إنه ليس مستودع git، فاضغط الرابط **create a repository** ثم **Create repository**.
4. ستظهر قائمة الملفات المتغيرة. اكتب في خانة العنوان `Initial data` واضغط **Commit to main**.
5. اضغط **Publish repository**، واترك خيار **Keep this code private** مفعّلًا، ثم **Publish repository**.
6. انتهى. أي تعديل لاحق على الملفات يظهر في GitHub Desktop: اكتب وصفًا ← Commit ← **Push origin**.

### الطريقة ب: من موقع GitHub مباشرة
1. على github.com اضغط **New repository**، اكتب الاسم، اختر **Private**، ثم **Create repository**.
2. اضغط الرابط **uploading an existing file**.
3. اسحب مجلدات `meta` و`tafsir` و`fonts` و`docs` و`tools` وملفات الجذر (`README.md` وغيرها) إلى الصفحة، ثم **Commit changes**.
   حدود الرفع من المتصفح (حسب علمي): نحو 25 ميغابايت للملف و100 ملف في الدفعة الواحدة. أكبر ملفاتنا (تفسير السعدي) 6 ميغابايت، فلا مشكلة. ارفع خطوط QCF4 في دفعة مستقلة.

### الطريقة ج: أوامر git
```
cd D:\maqraa\maqraa-data
git init
git add .
git commit -m "Initial data"
git branch -M main
git remote add origin https://github.com/اسم_حسابك/maqraa-data.git
git push -u origin main
```
(أنشئ المستودع الفارغ على الموقع أولًا. وإن طلب كلمة مرور فهو يريد Personal Access Token لا كلمة مرور الحساب.)

## 3) توليد الملفين اللذين تتحكم فيهما أنت
بعد قرارك بشأن الترخيص (`LICENSES.md`)، من داخل مجلد المستودع:
```
python tools/build_pages.py
python tools/apply_pagination.py --pages-file mushaf/qcf4_pages.json
python tools/collect_fonts.py
python tools/validate.py --pages-file mushaf/qcf4_pages.json
python tools/make_manifest.py --version 2026.10.1
python tools/validate.py --manifest
```
- `build_pages.py` يحتاج إنترنت (يجلب 604 صفحة، نحو دقيقة).
- `collect_fonts.py` ينسخ الخطوط من `C:\Windows\Fonts`، ويخبرك إن نقص منها شيء.
- `validate.py` يجب أن ينتهي بعبارة «كل الفحوص نجحت ✅». وإلا فأرسل لي سطر الفشل.
- بعدها اعمل Commit وPush.

## 4) مطابقة الأرقام مع مصحفك المطبوع (دورك أنت)
افتح `docs/VERIFY_CHECKLIST_AR.md` وقارن بمصحف المدينة المطبوع (طبعة 1441هـ، ‏604 صفحات):
- الأهم: بدايات الأجزاء الثلاثين وصفحاتها.
- ثم بعض الأحزاب والأرباع وبدايات سور مختارة.
أرسل لي ما اختلف فقط (رقم الصف وما في مصحفك).

## 5) ما الذي أحتاجه منك بعد ذلك؟
- إن كان المستودع **خاصًا** فلا أستطيع قراءته. المطلوب منك: احتفظ بنسخة المجلد على جهازك، وعندما أبني لك التطبيق سأكتب سكربتًا يضم ملفات هذا المجلد (من مسار تحدده) إلى التطبيق وقت البناء.
- أخبرني بنتيجة `validate.py` وبقرارك في الترخيص: هل تضمّ خطوط QCF4 داخل التطبيق أم يبقى يقرؤها من جهاز المستخدم.

## 6) أمان
- لا ترفع رمز بوت تيليجرام ولا ملف `.env` (مُدرَج في `.gitignore`).
- لا تضع أي أسرار في هذا المستودع.
