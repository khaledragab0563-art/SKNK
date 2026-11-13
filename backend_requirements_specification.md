# 📋 مستند متطلبات الباك إند لمنصة Sknk (سكنك - جامعة المنيا)

## 1. نظرة عامة على المشروع (Project Overview)
منصة **Sknk** هي وسيط إلكتروني لإدارة سكن طلاب جامعة المنيا. 
* **فكرة العمل:** المالك يسجل حساباً ويقوم برفع وحدته السكنية $\leftarrow$ تظهر للإدارة بحالة "قيد المراجعة" (`pending`) $\leftarrow$ توافق الإدارة عليها لتصبح "منشورة" (`approved`) $\leftarrow$ يتصفح الطالب الوحدات، يختار ما يعجبه، ويقدم طلب حجز (مع إمكانية حجز أماكن للأصدقاء) $\leftarrow$ تتواصل الإدارة وتؤكد الحجز ليتحول إلى "مؤكد" (`confirmed`) وتغلق الشقة تلقائياً.
* **السياسة الأساسية:** التعامل الأول والأخير (المعاينة، التعاقد، التحصيل) يتم حصرياً عبر **إدارة المنصة** (عزل تام لبيانات الملاك عن الطلاب وعدم تواصلهم المباشر خارج المنصة).

---

## 2. أدوار المستخدمين وصلاحياتهم (RBAC - Roles & Permissions)
المنصة تعتمد على 3 أدوار رئيسية:
1. **Student (الطالب):**
   * تصفح وفلترة الوحدات المتاحة.
   * تقديم طلبات الحجز (فردي أو جماعي للأصدقاء).
   * عرض ومتابعة حالة طلباته الشخصية.
2. **Owner (المالك):**
   * إضافة، تعديل، وحذف وحداته الخاصة.
   * متابعة حالة وحداته (قيد المراجعة، منشور، محجوز).
3. **Admin (مدير المنصة):**
   * صلاحيات مطلقة: الموافقة/رفض الإعلانات، تأكيد/إلغاء الحجوزات، عرض كافة بيانات الملاك والطلاب السرية (أرقام الهواتف، والعناوين التفصيلية).

---

## 3. هيكل قاعدة البيانات المقترح (Database Schema Design)

### أ. جدول المستخدمين (`Users`)
* `id` (ObjectId / PK)
* `name` (String)
* `email` (String, Unique)
* `password` (String, Hashed)
* `role` (Enum: `'student'`, `'owner'`, `'admin'`)
* `phone` (String)
* `createdAt` (Timestamp)

### ب. جدول الوحدات السكنية (`Properties`)
* `id` (ObjectId / PK)
* `ownerId` (Reference to Users)
* `title` (String) - *عنوان الوحدة المختصر*
* `area` (String) - *المنطقة يكتبها المالك بحرية (مثل: حي شلبي، عدنان المالكي)*
* `type` (Enum: `'شقة كاملة'`, `'غرفة مستقلة'`, `'سرير (مشاركة)'`)
* `detailedAddress` (String) - *العنوان التفصيلي السري (يظهر للإدارة فقط)*
* `price` (Number) - *الإيجار الشهري*
* `ownerPhone` (String) - *هاتف المالك السري (يظهر للإدارة فقط)*
* `rooms` (Number)
* `bathrooms` (Number)
* `capacity` (Number) - *الاستيعاب الأقصى للأفراد*
* `image` (String) - *رابط الصورة أو مسار التخزين*
* `desc` (String) - *المميزات والمواصفات*
* `status` (Enum: `'pending'`, `'approved'`, `'rejected'`, `'booked'`, `'rented'`)
* `createdAt` (Timestamp)

### ج. جدول الحجوزات (`Bookings`)
* `id` (String / ObjectId) - *رقم مرجعي مثل SKNK-XXXX*
* `studentId` (Reference to Users)
* `propertyId` (Reference to Properties)
* `studentName` (String)
* `studentPhone` (String)
* `studentCollege` (String) - *كلية جامعة المنيا والفرقة*
* `bookingType` (Enum: `'حجز فوري (عربون)'`, `'طلب معاينة'`)
* `spotsCount` (Number) - *عدد الأماكن المطلوبة*
* `friendsNames` (String, Optional) - *أسماء الأصدقاء المرافقين*
* `status` (Enum: `'pending'`, `'confirmed'`, `'cancelled'`)
* `createdAt` (Timestamp)

---

## 4. مسارات الـ API المطلوبة (API Endpoints Overview)

### أ. المصادقة والمستخدمين (Auth)
* `POST /api/auth/register` (تسجيل حساب جديد - طالب أو مالك)
* `POST /api/auth/login` (تسجيل الدخول وإرجاع JWT Token)
* `GET /api/auth/me` (جلب بيانات المستخدم الحالي)

### ب. الوحدات السكنية (Properties)
* `GET /api/properties` (جلب الوحدات المنشورة فقط `approved` مع دعم الفلترة بالمنطقة والنوع والبحث)
* `POST /api/properties` (إضافة وحدة جديدة بواسطة المالك - الحالة الافتراضية `pending`)
* `GET /api/properties/owner` (جلب وحدات المالك المسجل الدخول حالياً)
* `DELETE /api/properties/:id` (حذف وحدة بواسطة المالك أو الأدمن)
* `PATCH /api/properties/:id/status` (تغيير حالة الوحدة بواسطة الأدمن: موافقة/رفض/محجوز)

### ج. الحجوزات والطلبات (Bookings)
* `POST /api/bookings` (إنشاء طلب حجز جديد من قبل الطالب)
* `GET /api/bookings/my` (جلب حجوزات الطالب الحالي)
* `GET /api/admin/bookings` (جلب كافة الحجوزات للأدمن)
* `PATCH /api/admin/bookings/:id/confirm` (تأكيد الحجز من الأدمن $\rightarrow$ تحويل حالة الشقة إلى `booked` تلقائياً)
* `DELETE /api/admin/bookings/:id` (حذف أو إلغاء طلب الحجز)