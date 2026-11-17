# يزيد — مشروع Capacitor (Android + iOS)

يحوّل صفحة الويب (`www/index.html`) إلى تطبيق أصلي، مع **إشعارات أذان محلية تعمل والتطبيق مغلق** (تُجدول 7 أيام وتتجدد عند فتح التطبيق)، وتحديد موقع أصلي.

## ⚡ أسهل طريقة للحصول على ملف APK (بدون Android Studio)
المشروع يحتوي على بناء تلقائي مجاني عبر GitHub Actions:
1. أنشئ حسابًا مجانيًا في github.com ثم مستودعًا جديدًا (New repository) وسمّه `taqwa`.
2. ارفع **كل محتويات هذا المجلد** إلى المستودع (Add file ← Upload files)، بما فيها مجلد `.github` (قد يكون مخفيًا في ملفاتك).
3. افتح تبويب **Actions** ← **Build Android APK** ← **Run workflow**.
4. بعد نحو 5–10 دقائق افتح التشغيل المكتمل ✅ ونزّل **taqwa-apk** من قسم Artifacts، ثم فك الضغط لتجد `app-debug.apk`.
5. انقل الملف لجوالك وثبّته (اسمح بالتثبيت من مصدر خارجي). قد يحذّرك Play Protect لأن الملف غير موقّع للمتجر، وهذا طبيعي للتجربة.

ملاحظة: هذا ملف تجريبي (debug) للتوزيع الشخصي. للنشر في Google Play تحتاج AAB موقّعًا كما في الخطوات أدناه.

## 1) التجهيز (مرة واحدة)
المتطلبات: Node.js، وAndroid Studio للأندرويد، وMac مع Xcode لـ iOS.

```bash
npm install @capacitor/core @capacitor/cli @capacitor/android @capacitor/ios \
            @capacitor/local-notifications @capacitor/geolocation
npx cap add android
npx cap add ios        # على Mac فقط
npx cap sync
```
غيّر `appId` في `capacitor.config.json` إلى معرّفك (مثل `com.اسمك.taqwa`)، لأنه لا يتغير بعد النشر.

## 2) الأيقونة
ضع صورة 1024×1024 في `assets/icon.png` (توجد أيقونة مؤقتة)، ثم:
```bash
npm install -D @capacitor/assets && npx capacitor-assets generate
```

## 3) الأذونات
**Android** (`android/app/src/main/AndroidManifest.xml`) تأكد من وجود:
```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM"/>
```
راجع سياسة Google Play بخصوص الإنذارات الدقيقة (Exact alarms) قبل الرفع، وراجع توثيق مكوّن LocalNotifications للتفاصيل.

**iOS** (`ios/App/App/Info.plist`):
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>نستخدم موقعك لحساب مواقيت الصلاة واتجاه القبلة.</string>
<key>UIBackgroundModes</key>
<array><string>audio</string></array>
```
(`audio` ليستمر تشغيل التلاوة والشاشة مقفلة.)

## 4) صوت الأذان (اختياري)
- أندرويد: ضع الملف في `android/app/src/main/res/raw/adhan.wav`.
- iOS: أضف `adhan.wav` إلى مشروع Xcode (مدته أقل من 30 ثانية).
- ثم في `www/index.html` غيّر `const ADHAN_SOUND=null` إلى `"adhan.wav"` ثم `npx cap sync`.
استخدم تسجيلًا تملك حقوقه.

## 5) التشغيل والبناء
```bash
npx cap sync
npx cap open android   # Build > Generate Signed Bundle (AAB)
npx cap open ios       # Product > Archive
```

## 6) قائمة النشر
**مشتركة:** سياسة الخصوصية على رابط عام (قالب في `PRIVACY.md`)، لقطات شاشة، وصف عربي وإنجليزي، تصنيف عمري.
**Apple:** حساب مطوّر (99$ سنويًا)، تعبئة App Privacy (لا بيانات تُجمع)، وملاحظات للمراجع تشرح ميزات التطبيق الأصلية (إشعارات الأذان، الموقع، البوصلة).
**Google Play:** حساب (25$ مرة واحدة)، نموذج Data safety، وللحساب الشخصي الجديد: اختبار مغلق بـ 12 مختبِرًا لمدة 14 يومًا قبل طلب الإنتاج.

## ⚠️ قبل النشر: حقوق الصوت
التلاوات تُحمَّل من روابط خوادم خارجية (mp3quran وarchive.org). استخدامها في تطبيق منشور يحتاج إذنًا أو التزامًا بشروط الاستخدام الرسمية، أو استضافة ملفات مرخّصة. راجع هذا أولًا، فقد يؤدي إلى رفض أو إزالة التطبيق.

## ملاحظات تقنية
- التشغيل الصوتي بالخلفية على أندرويد قد يتوقف عند إطفاء الشاشة داخل WebView؛ للحل الكامل استخدم مكوّن تشغيل أصلي.
- مواقيت الصلاة تُحسب بطريقة أم القرى وقد تختلف دقيقة عن التقويم الرسمي.
- الصفحة تعمل أيضًا في المتصفح (بدون الإشعارات الأصلية).
