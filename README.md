# سامانه ثبت سفارش مشتریان

وب‌اپ تک‌صفحه‌ای فارسی (راست‌به‌چپ). مشتریان و محصولات به‌صورت کاشی‌های پازلی نمایش داده می‌شوند، لیست‌ها از اکسل وارد می‌شوند و داده‌ها بین چند نفر به‌صورت زنده مشترک است.

## راه‌اندازی

### ۱) انتشار روی GitHub Pages
1. فایل `index.html` را در یک ریپازیتوری بگذارید.
2. Settings → Pages → Branch: `main` → Save.

### ۲) اشتراک‌گذاری بین چند نفر (Firebase، رایگان)
1. در https://console.firebase.google.com یک پروژه بسازید.
2. Build → **Firestore Database** → Create database (Production mode و نزدیک‌ترین منطقه).
3. **ورود با رمز:** Build → **Authentication** → Get started → Sign-in method → **Email/Password** را فعال کنید.
   - در تب Users، با «Add user» برای هر همکار یک ایمیل و رمز بسازید (ایمیل می‌تواند ساختگی باشد، مثل `ali@shop.com`).
   - **نقش‌ها:** بعد از ساخت هر کاربر، UID او را از ستون «User UID» کپی کنید. در Firestore → Start collection → نام `users` → Document ID را برابر همان UID بگذارید و یک فیلد از نوع string با نام `role` بسازید. مقدارش `admin` (مدیر: ورود اکسل، افزودن مشتری/محصول، ثبت سفارش) یا `seller` (فروشنده: فقط مشاهده و ثبت سفارش) باشد. کاربری که سند نقش نداشته باشد نمی‌تواند وارد شود.
   - **مهم:** در Authentication → Settings → User actions گزینه **Enable create (sign-up)** را غیرفعال کنید تا کسی نتواند خودش حساب بسازد.
4. در Firestore → Rules این قوانین را بگذارید و Publish بزنید (مدیر همه‌چیز، فروشنده فقط سفارش‌ها را تغییر می‌دهد):
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       function role() {
         return exists(/databases/$(database)/documents/users/$(request.auth.uid))
           ? get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role
           : null;
       }
       function isAdmin() { return request.auth != null && role() == 'admin'; }
       function isStaff() { return request.auth != null && (role() == 'admin' || role() == 'seller'); }

       match /customers/{id} { allow read: if isStaff(); allow write: if isAdmin(); }
       match /products/{id}  { allow read: if isStaff(); allow write: if isAdmin(); }
       match /orders/{id}    { allow read, write: if isStaff(); }
       match /users/{uid}    { allow read: if request.auth != null && request.auth.uid == uid; allow write: if false; }
     }
   }
   ```
   علاوه بر پنهان شدن بخش‌های مدیریتی برای فروشنده، خود قوانین سرور هم نوشتن در لیست مشتریان و محصولات را برای او رد می‌کنند.
   مقدار `apiKey` در کد سایت عمومی است و این طبیعی است؛ امنیت واقعی از همین قوانین و بستن ثبت‌نام تامین می‌شود.
5. Project settings (چرخ‌دنده) → Your apps → آیکون `</>` (Web) → ثبت اپ → مقادیر `firebaseConfig` را کپی کنید.
6. در ابتدای اسکریپت `index.html` مقادیر `FIREBASE_CONFIG` (`apiKey`, `authDomain`, `projectId`, `appId`) را پر کنید و commit کنید.

اگر `FIREBASE_CONFIG` خالی باشد، برنامه در حالت محلی (فقط روی همان مرورگر) کار می‌کند.

## فرمت اکسل
- شیت **مشتریان**: ستون‌های `نام`، `تلفن`، `آدرس`
- شیت **محصولات**: ستون‌های `نام`، `قیمت`

تشخیص شیت‌ها بر اساس وجود ستون «قیمت» است. دکمه «دانلود فایل نمونه» در برنامه یک نمونه می‌دهد. ورود مجدد فایل، مشتریان و محصولات هم‌نام را به‌روز می‌کند و سفارش‌ها را پاک نمی‌کند.

## نکته
اگر دو نفر هم‌زمان سفارش یک مشتری را ویرایش کنند، آخرین ذخیره معتبر است.
