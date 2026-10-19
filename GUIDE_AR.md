# دليل المبتدئ - خطوة بخطوة (مع Netlify للاستضافة)

> ملاحظة: Firebase يبقى مطلوباً لقاعدة البيانات وتسجيل الدخول. Netlify يستضيف فقط صفحة الرابط (/q/الكود).

## 1) مشروع Firebase (مرة واحدة)
1. console.firebase.google.com ثم Add project (مثلاً nutriclinic-yousfi).
2. Add app ثم Android، اسم الحزمة: nutriclinic.yousfi.aistudio.com
3. حمّل google-services.json وضعه في مجلد app/ مكان الملف الوهمي.
4. Authentication ثم Get started ثم فعّل Email/Password و Anonymous.
5. Users ثم Add user (بريد الطبيبة وكلمة مرور قوية) وانسخ User UID.
6. Settings ثم User actions ثم ألغِ Enable create (sign-up).
7. Firestore Database ثم Create database (Production).
8. تبويب Rules: الصق محتوى firebase/firestore.rules بعد استبدال REPLACE_WITH_DOCTOR_UID بالـ UID ثم Publish.
   (لا تحتاج سطر أوامر لهذه الخطوة.)

## 2) الاستضافة على Netlify
1. أنشئ حساباً على netlify.com.
2. افتح app.netlify.com/drop واسحب إليه المجلد: firebase/hosting/public
3. Site configuration ثم Change site name: اختر nutriclinic-yousfi (الرابط: nutriclinic-yousfi.netlify.app).
4. جرّب: https://nutriclinic-yousfi.netlify.app/q/DRY-ABCD-2345 (يجب أن تظهر الصفحة).
5. جرّب: https://nutriclinic-yousfi.netlify.app/.well-known/assetlinks.json (يجب أن يظهر JSON).

## 3) ربط التطبيق بالنطاق
افتح app/build.gradle.kts وغيّر السطر:
    val webHost = "nutriclinic-yousfi.netlify.app"
ليطابق اسم موقعك على Netlify تماماً.

## 4) بصمة التطبيق (لفتح الرابط مباشرة داخل التطبيق)
1. في Android Studio: Gradle (يمين) > app > Tasks > android > signingReport
2. انسخ SHA-1 وأضفه في Firebase: Project settings > تطبيقك > Add fingerprint.
3. انسخ SHA-256 وضعه في firebase/hosting/public/assetlinks.json مكان REPLACE_WITH_SHA256_CERT_FINGERPRINT
   (وانسخ نفس المحتوى إلى .well-known/assetlinks.json)، ثم اسحب مجلد public إلى Netlify مرة ثانية (Deploys > Drag and drop).
بدون هذه الخطوة: يبقى كل شيء يعمل (الكود، QR، زر "فتح في التطبيق" بالصفحة)، فقط الرابط لا يفتح التطبيق تلقائياً.

## 5) تشغيل التطبيق
Android Studio ثم File > Open > مجلد NutriClinic > انتظر Sync > Run على هاتفك.

## 6) الإشعارات الفورية (اختيارية)
تحتاج خطة Blaze في Firebase وأداة سطر الأوامر:
    npm install -g firebase-tools
    firebase login
    cd firebase
    firebase use --add
    cd functions && npm install && cd ..
    firebase deploy --only functions
بدونها: يعمل التطبيق كاملاً، وتصل التنبيهات للطبيبة أثناء فتح التطبيق، وتجد الاستبيانات الجديدة عند فتحه.

## 7) App Check (قبل الإطلاق للمرضى)
راجع README.md الخطوة 7.

## بدون Android Studio: بناء التطبيق على GitHub (مجاني)
1. أنشئ حساباً على github.com ثم New repository (اسمه nutriclinic، اتركه Public أو Private).
2. من متصفح الحاسوب: Add file > Upload files، واسحب **محتويات** مجلد NutriClinic (app, firebase, gradle, .github ...) وليس الـ zip، ثم Commit changes.
   (مجلد .github مخفي: فعّل "إظهار الملفات المخفية" في ويندوز/ماك قبل السحب.)
3. تبويب Actions > Build APK > Run workflow. انتظر 10-15 دقيقة حتى يصبح أخضر.
4. افتح التشغيل الناجح > Artifacts > NutriClinic-APK > حمّله (zip) وفك ضغطه لتجد app-debug.apk.
5. انقله للهاتف وثبّته. هذا يعمل من متصفح أي حاسوب، ويمكن تنزيل الناتج من الهاتف مباشرة.
