# تحويل التطبيق إلى APK حقيقي بإشعارات تعمل حتى لو كان مغلقًا (باستخدام Capacitor)

هذا الدليل يشرح خطوة بخطوة كيف تبني تطبيق أندرويد حقيقي من ملفات `index.html` باستخدام
Capacitor، بحيث تعمل تذكيرات المهام والعادات كـ **منبّهات حقيقية على مستوى نظام التشغيل**
حتى لو كان التطبيق مغلقًا تمامًا (مو مجرد في الخلفية).

> ⚠️ هذه الخطوات تُنفَّذ على **جهاز الكمبيوتر الخاص بك** (وليس هنا في المحادثة)، لأنها
> تحتاج Android Studio ومحاكي/هاتف حقيقي للتجربة والبناء. الكود داخل `index.html` أصبح
> جاهزًا بالفعل ويكتشف تلقائيًا إذا كان يعمل داخل Capacitor أو في متصفح عادي.

---

## 1) المتطلبات قبل البدء

ثبّت هذه البرامج على جهازك إن لم تكن مثبّتة:

| البرنامج | الرابط | ملاحظة |
|---|---|---|
| **Node.js** (نسخة 18 أو أحدث) | nodejs.org | يشمل `npm` تلقائيًا |
| **Android Studio** | developer.android.com/studio | يشمل Android SDK + محاكي |
| **JDK 17** | يأتي مضمّنًا غالبًا مع Android Studio | تأكد أن `JAVA_HOME` مضبوط |

للتأكد من التثبيت، افتح Terminal (أو CMD/PowerShell على ويندوز) واكتب:

```bash
node -v
npm -v
```

---

## 2) تجهيز مجلد المشروع

1. أنشئ مجلدًا جديدًا فارغًا على جهازك، مثلًا `focus-todo-apk`
2. انسخ داخله الملفات التالية (نفس التي حمّلتها من هذه المحادثة):
   - `package.json`
   - `capacitor.config.json`
3. أنشئ داخله مجلدًا فرعيًا اسمه **`www`**، وانسخ داخل `www` كل ملفات التطبيق:
   - `index.html`
   - `service-worker.js`
   - `manifest.json`
   - `icon-192.png`, `icon-512.png`, `icon-192-maskable.png`, `icon-512-maskable.png`, `logo-icon.png`

شكل المجلد النهائي يجب أن يكون هكذا:

```
focus-todo-apk/
├── package.json
├── capacitor.config.json
└── www/
    ├── index.html
    ├── service-worker.js
    ├── manifest.json
    └── icon-*.png ...
```

---

## 3) تثبيت الحزم وإضافة منصّة أندرويد

داخل مجلد المشروع (`focus-todo-apk`)، شغّل بالترتيب:

```bash
npm install
npx cap init "Focus Todo" "com.yourname.focustodo" --web-dir=www
npx cap add android
```

> 💡 لو ظهرت رسالة أن `capacitor.config.json` موجود بالفعل أثناء `cap init`، تجاهلها
> واستمر — الملف المرفق هنا مُجهّز مسبقًا بنفس الإعدادات.

هذا سينشئ مجلد `android/` يحتوي على مشروع أندرويد كامل (Gradle) جاهز للبناء.

---

## 4) تثبيت إضافة الإشعارات المحلية (LocalNotifications)

```bash
npm install @capacitor/local-notifications
npx cap sync android
```

أمر `npx cap sync` هذا **مهم جدًا** — نفّذه في كل مرة:
- تضيف/تحدّث فيها حزمة Capacitor
- تعدّل فيها أي ملف داخل `www/` (مثل `index.html`) وتريد أن التغيير ينعكس في تطبيق أندرويد

---

## 5) إضافة صلاحية الإشعارات في أندرويد 13+

افتح الملف:
```
android/app/src/main/AndroidManifest.xml
```

وتأكد من وجود هذا السطر بجانب باقي صلاحيات `<uses-permission>` (أضِفه إذا لم يكن موجودًا):

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
```

هذه الصلاحيات ضرورية حتى يستطيع Android 13+ عرض الإشعارات، وحتى تُجدوَل المنبّهات بدقة.

---

## 6) فتح المشروع في Android Studio والتجربة

```bash
npx cap open android
```

هذا سيفتح Android Studio تلقائيًا. من هناك:

1. انتظر حتى ينتهي Gradle من التحميل والفهرسة (أول مرة قد تأخذ عدة دقائق)
2. اختر جهازًا/محاكيًا من القائمة العلوية
3. اضغط ▶️ **Run** لتشغيل التطبيق

جرّب داخل التطبيق:
- سجّل دخولك أو أنشئ حسابًا
- من الشاشة الرئيسية اضغط **"تفعيل"** في بانر الإشعارات — سيظهر طلب صلاحية أندرويد الحقيقي (وليس نافذة المتصفح)
- أضف مهمة بتذكير بعد دقيقتين من الوقت الحالي، ثم **أغلق التطبيق تمامًا** (اسحبه من قائمة التطبيقات الأخيرة) وانتظر — يجب أن يصلك الإشعار رغم إغلاق التطبيق

---

## 7) بناء ملف APK للتوزيع

من داخل Android Studio:

1. من القائمة: **Build → Generate Signed Bundle / APK**
2. اختر **APK**
3. أنشئ *keystore* جديد (مرة واحدة فقط) واحفظ بياناته في مكان آمن — ستحتاجها لأي تحديث مستقبلي لنفس التطبيق
4. اختر **release**، واضغط **Finish**

ستجد ملف الـ APK النهائي هنا:
```
android/app/release/app-release.apk
```

انسخه لهاتفك وثبّته (فعّل "Install from unknown sources" إذا طُلب منك ذلك).

---

## 8) كل مرة تُحدّث فيها الكود مستقبلًا

عندما تحصل على نسخة جديدة من `index.html` (أو أي ملف داخل `www/`):

```bash
# 1) انسخ الملفات الجديدة إلى مجلد www/ (استبدل القديمة)
# 2) نفّذ:
npx cap sync android
npx cap open android
# 3) أعد البناء (Run أو Generate Signed APK) كما في الخطوتين 6 و7
```

---

## استكشاف الأخطاء الشائعة

| المشكلة | الحل |
|---|---|
| الإشعار ما يوصل أبدًا حتى والتطبيق مفتوح | تأكد أنك ضغطت "تفعيل" ووافقت على صلاحية الإشعارات داخل التطبيق نفسه (وليس فقط صلاحيات النظام) |
| الإشعار يوصل والتطبيق مفتوح لكن مو لما يكون مغلق | بعض الشركات المصنّعة (Xiaomi، Huawei، Oppo...) عندها "توفير طاقة" عدواني يقتل التطبيقات ويمنع المنبّهات. اذهب لإعدادات البطارية الخاصة بالتطبيق واستثنِه من "تحسين البطارية" |
| خطأ `SDK location not found` عند البناء | افتح Android Studio → Preferences → Appearance & Behavior → System Settings → Android SDK، وتأكد من مسار الـ SDK، ثم أضِف نفس المسار في متغير بيئة `ANDROID_HOME` |
| `npx cap sync` يعطي خطأ متعلق بالحزم | احذف مجلد `node_modules` وملف `package-lock.json` ثم شغّل `npm install` من جديد |

---

## ملخص سريع للأوامر (بعد إعداد المشروع أول مرة)

```bash
npm install
npx cap init "Focus Todo" "com.yourname.focustodo" --web-dir=www
npx cap add android
npm install @capacitor/local-notifications
npx cap sync android
npx cap open android
```
