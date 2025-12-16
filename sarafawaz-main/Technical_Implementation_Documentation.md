# Technical Implementation Documentation
# وثيقة التنفيذ التقني

## نظام سارة لإدارة الحضور والموارد البشرية
### Sara Attendance & HRM System

**الإصدار:** 1.0  
**التاريخ:** ديسمبر 2024  
**نوع التطبيق:** Full-Stack Web Application

---

## جدول المحتويات

1. [نظرة عامة على التقنيات](#1-نظرة-عامة-على-التقنيات)
2. [هيكل المشروع](#2-هيكل-المشروع)
3. [طبقة قاعدة البيانات](#3-طبقة-قاعدة-البيانات-database-layer)
4. [طبقة الخادم](#4-طبقة-الخادم-backend-layer)
5. [طبقة واجهة المستخدم](#5-طبقة-واجهة-المستخدم-frontend-layer)
6. [الوظائف الرئيسية](#6-الوظائف-الرئيسية-main-functions)
7. [نظام الأمان](#7-نظام-الأمان-security-system)
8. [تدفق العمل](#8-تدفق-العمل-system-flow)
9. [بيئة التشغيل](#9-بيئة-التشغيل-runtime-environment)

---

## 1. نظرة عامة على التقنيات

### 1.1 ملخص التقنيات المستخدمة

| الطبقة | التقنية | اللغة | الغرض |
|--------|---------|-------|-------|
| Frontend | React 18 | TypeScript | بناء واجهة المستخدم التفاعلية |
| Backend | Express.js | TypeScript | معالجة الطلبات وAPI |
| Database | PostgreSQL | SQL | تخزين البيانات |
| ORM | Drizzle ORM | TypeScript | التعامل مع قاعدة البيانات |
| Styling | Tailwind CSS | CSS | تصميم الواجهة |
| UI Components | shadcn/ui | TypeScript/JSX | مكونات جاهزة |

### 1.2 لماذا هذه التقنيات؟

#### React + TypeScript (Frontend)
```
السبب:
- React: أشهر مكتبة لبناء واجهات المستخدم
- TypeScript: يضيف Type Safety ويقلل الأخطاء البرمجية
- Component-based: سهولة إعادة استخدام الكود
```

#### Express.js + Node.js (Backend)
```
السبب:
- خفيف وسريع
- نفس اللغة (JavaScript/TypeScript) في Frontend و Backend
- مجتمع كبير ودعم واسع
```

#### PostgreSQL (Database)
```
السبب:
- قاعدة بيانات علائقية موثوقة
- دعم للـ UUID والـ JSON
- أداء عالي وأمان قوي
```

#### Drizzle ORM
```
السبب:
- Type-safe: التحقق من الأنواع في وقت الكتابة
- خفيف وسريع مقارنة بـ Prisma
- يدعم PostgreSQL بشكل كامل
```

---

## 2. هيكل المشروع

### 2.1 شجرة الملفات

```
sara-portal/
├── client/                    # Frontend (React)
│   └── src/
│       ├── components/        # مكونات قابلة لإعادة الاستخدام
│       │   └── ui/           # مكونات shadcn/ui
│       ├── hooks/            # React Hooks مخصصة
│       ├── lib/              # مكتبات مساعدة
│       ├── pages/            # صفحات التطبيق
│       │   ├── attendance/   # صفحات الحضور
│       │   ├── departments/  # صفحات الأقسام
│       │   ├── hrm/          # صفحات الموظفين
│       │   └── settings/     # صفحات الإعدادات
│       ├── App.tsx           # التطبيق الرئيسي
│       └── main.tsx          # نقطة الدخول
│
├── server/                    # Backend (Express)
│   ├── middleware/           # وسيط التحقق
│   │   └── attendance-security.ts
│   ├── utils/               # أدوات مساعدة
│   │   └── security.ts      # التشفير والأمان
│   ├── db.ts                # اتصال قاعدة البيانات
│   ├── routes.ts            # نقاط API
│   ├── storage.ts           # طبقة التخزين
│   └── index.ts             # نقطة الدخول
│
├── shared/                    # مشترك بين Frontend و Backend
│   └── schema.ts            # تعريف الجداول والأنواع
│
└── docs/                      # التوثيق
    ├── Software_Requirements_Specification.md
    └── Technical_Implementation_Documentation.md
```

### 2.2 شرح المجلدات

| المجلد | الوظيفة |
|--------|---------|
| `client/` | كود واجهة المستخدم (React) |
| `client/src/pages/` | صفحات التطبيق المختلفة |
| `client/src/components/` | مكونات قابلة لإعادة الاستخدام |
| `server/` | كود الخادم (Express) |
| `server/middleware/` | وسيط التحقق من الأمان |
| `server/utils/` | أدوات التشفير والحسابات |
| `shared/` | الكود المشترك (الأنواع والجداول) |

---

## 3. طبقة قاعدة البيانات (Database Layer)

### 3.1 الجداول الرئيسية

#### جدول المستخدمين (users)
```typescript
// الملف: shared/schema.ts
// اللغة: TypeScript

export const users = pgTable("users", {
  id: varchar("id", { length: 36 }).primaryKey().default(sql`gen_random_uuid()`),
  username: text("username").notNull().unique(),      // اسم المستخدم
  password: text("password").notNull(),               // كلمة المرور (مشفرة)
  name: text("name").notNull(),                       // الاسم الكامل
  email: text("email"),                               // البريد الإلكتروني
  role: roleEnum("role").notNull().default("employee"), // الدور
  departmentId: varchar("department_id"),             // القسم
  deviceFingerprint: text("device_fingerprint"),      // بصمة الجهاز (مشفرة)
  createdAt: timestamp("created_at").notNull().defaultNow(),
});
```

**شرح الكود:**
- `pgTable`: دالة من Drizzle لإنشاء جدول PostgreSQL
- `varchar`, `text`: أنواع الأعمدة
- `primaryKey()`: تحديد المفتاح الرئيسي
- `default(sql\`gen_random_uuid()\`)`: توليد UUID تلقائياً
- `notNull()`: الحقل إجباري
- `unique()`: القيمة فريدة

#### جدول سجلات الحضور (attendance_records)
```typescript
// الملف: shared/schema.ts

export const attendanceRecords = pgTable("attendance_records", {
  id: varchar("id").primaryKey().default(sql`gen_random_uuid()`),
  userId: varchar("user_id").notNull().references(() => users.id),  // ربط بالمستخدم
  type: attendanceTypeEnum("type").notNull(),         // نوع: حضور أو انصراف
  timestamp: timestamp("timestamp").notNull().defaultNow(),
  deviceFingerprint: text("device_fingerprint").notNull(),
  latitude: text("latitude"),                         // خط العرض
  longitude: text("longitude"),                       // خط الطول
});
```

**شرح الكود:**
- `references(() => users.id)`: Foreign Key يربط بجدول المستخدمين
- `attendanceTypeEnum`: نوع محدد (check_in أو check_out)

#### جدول المواقع المصرح بها (allowed_locations)
```typescript
// الملف: shared/schema.ts

export const allowedLocations = pgTable("allowed_locations", {
  id: varchar("id").primaryKey().default(sql`gen_random_uuid()`),
  name: text("name").notNull(),                       // اسم الموقع
  latitude: text("latitude").notNull(),               // خط العرض
  longitude: text("longitude").notNull(),             // خط الطول
  radiusMeters: text("radius_meters").default("100"), // النطاق بالأمتار
  departmentId: varchar("department_id"),             // القسم (اختياري)
  active: boolean("active").default(true),            // مفعّل أم لا
});
```

### 3.2 مخطط العلاقات (ER Diagram - نصي)

```
┌─────────────────┐       ┌──────────────────────┐
│     users       │       │  attendance_records  │
├─────────────────┤       ├──────────────────────┤
│ id (PK)         │◄──────│ user_id (FK)         │
│ username        │       │ id (PK)              │
│ password        │       │ type                 │
│ name            │       │ timestamp            │
│ role            │       │ latitude             │
│ department_id   │       │ longitude            │
│ deviceFingerprint│      │ deviceFingerprint    │
└────────┬────────┘       └──────────────────────┘
         │
         │ references
         ▼
┌─────────────────┐       ┌──────────────────────┐
│   locations     │       │   allowed_locations  │
│   (الأقسام)     │       │   (المواقع GPS)      │
├─────────────────┤       ├──────────────────────┤
│ id (PK)         │◄──────│ department_id (FK)   │
│ name            │       │ id (PK)              │
│ description     │       │ name                 │
└─────────────────┘       │ latitude, longitude  │
                          │ radiusMeters         │
                          └──────────────────────┘
```

---

## 4. طبقة الخادم (Backend Layer)

### 4.1 نقاط API الرئيسية

#### الملف: `server/routes.ts`

```typescript
// تسجيل الدخول
app.post("/api/auth/login", async (req, res) => {
  const { username, password } = req.body;
  
  // البحث عن المستخدم
  const user = await storage.getUserByUsername(username);
  
  // التحقق من كلمة المرور
  const isValid = await bcrypt.compare(password, user.password);
  
  if (isValid) {
    req.session.userId = user.id;  // حفظ الجلسة
    res.json({ user });
  }
});
```

**شرح الكود:**
- `app.post()`: تعريف endpoint من نوع POST
- `req.body`: البيانات المرسلة من العميل
- `bcrypt.compare()`: مقارنة كلمة المرور المشفرة
- `req.session`: حفظ بيانات الجلسة

#### تسجيل الحضور
```typescript
// تسجيل الحضور مع التحقق من الأمان
app.post("/api/attendance/check-in", 
  validateAttendance,  // Middleware للتحقق
  async (req, res) => {
    const { deviceFingerprint, latitude, longitude } = req.body;
    const userId = req.session.userId;
    
    // إنشاء سجل الحضور
    const record = await storage.createAttendanceRecord({
      userId,
      type: "check_in",
      deviceFingerprint,
      latitude,
      longitude
    });
    
    res.json({ success: true, record });
  }
);
```

**شرح الكود:**
- `validateAttendance`: Middleware يتحقق من الجهاز والموقع
- البيانات تُحفظ عبر `storage.createAttendanceRecord()`

### 4.2 طبقة التخزين (Storage Layer)

#### الملف: `server/storage.ts`

```typescript
// واجهة التخزين
export interface IStorage {
  // المستخدمين
  getUser(id: string): Promise<User | undefined>;
  getUserByUsername(username: string): Promise<User | undefined>;
  createUser(data: InsertUser): Promise<User>;
  
  // الحضور
  createAttendanceRecord(data: InsertAttendanceRecord): Promise<AttendanceRecord>;
  getAttendanceRecords(userId: string): Promise<AttendanceRecord[]>;
  
  // المواقع
  getAllowedLocations(): Promise<AllowedLocation[]>;
}

// التنفيذ باستخدام PostgreSQL
export class DatabaseStorage implements IStorage {
  async getUser(id: string): Promise<User | undefined> {
    const [user] = await db
      .select()
      .from(users)
      .where(eq(users.id, id));
    return user;
  }
  
  async createAttendanceRecord(data: InsertAttendanceRecord) {
    const [record] = await db
      .insert(attendanceRecords)
      .values(data)
      .returning();
    return record;
  }
}
```

**شرح الكود:**
- `IStorage`: واجهة تحدد العمليات المتاحة
- `DatabaseStorage`: التنفيذ الفعلي باستخدام Drizzle
- `db.select()`, `db.insert()`: استعلامات Drizzle ORM

---

## 5. طبقة واجهة المستخدم (Frontend Layer)

### 5.1 هيكل الصفحات

| الصفحة | الملف | الوظيفة |
|--------|-------|---------|
| الصفحة الرئيسية | `landing.tsx` | صفحة الترحيب |
| تسجيل الدخول | `login.tsx` | دخول المستخدم |
| لوحة المتدرب | `attendance/employee-dashboard.tsx` | تسجيل الحضور |
| لوحة المشرف | `attendance/supervisor-dashboard.tsx` | إدارة الفريق |
| الموظفين | `hrm/employee-list.tsx` | قائمة الموظفين |
| الأقسام | `departments/department-list.tsx` | إدارة الأقسام |
| المواقع | `settings/allowed-locations.tsx` | إدارة المواقع |
| التقارير | `reports.tsx` | عرض التقارير |

### 5.2 مثال: صفحة لوحة المتدرب

```tsx
// الملف: client/src/pages/attendance/employee-dashboard.tsx
// اللغة: TypeScript + JSX (React)

export default function EmployeeDashboard() {
  // جلب بيانات المستخدم
  const { data: currentUser } = useQuery({
    queryKey: ["/api/auth/user"],
  });
  
  // جلب حالة الحضور
  const { data: attendanceStatus } = useQuery({
    queryKey: ["/api/attendance/my-status"],
  });
  
  // دالة تسجيل الحضور
  const handleCheckIn = async () => {
    // الحصول على الموقع
    const position = await getCurrentPosition();
    
    // الحصول على بصمة الجهاز
    const fingerprint = await getDeviceFingerprint();
    
    // إرسال الطلب للخادم
    await apiRequest("POST", "/api/attendance/check-in", {
      deviceFingerprint: fingerprint,
      latitude: position.coords.latitude,
      longitude: position.coords.longitude,
    });
  };
  
  return (
    <div className="p-6" dir="rtl">
      <h1>لوحة المتدرب</h1>
      
      {/* زر تسجيل الحضور */}
      <Button onClick={handleCheckIn}>
        تسجيل الحضور
      </Button>
    </div>
  );
}
```

**شرح الكود:**
- `useQuery`: من TanStack Query لجلب البيانات
- `getCurrentPosition()`: الحصول على GPS من المتصفح
- `getDeviceFingerprint()`: الحصول على بصمة الجهاز
- `apiRequest()`: إرسال طلب للخادم
- `dir="rtl"`: اتجاه من اليمين لليسار

### 5.3 جلب بصمة الجهاز (FingerprintJS)

```tsx
// الملف: client/src/pages/attendance/employee-dashboard.tsx

import FingerprintJS from '@fingerprintjs/fingerprintjs';

// دالة الحصول على بصمة الجهاز
const getDeviceFingerprint = async (): Promise<string> => {
  const fp = await FingerprintJS.load();
  const result = await fp.get();
  return result.visitorId;  // معرف فريد للجهاز
};
```

**شرح الكود:**
- `FingerprintJS.load()`: تحميل المكتبة
- `fp.get()`: الحصول على البصمة
- `visitorId`: معرف فريد يعتمد على خصائص الجهاز

### 5.4 جلب الموقع الجغرافي

```tsx
// دالة الحصول على الموقع GPS
const getCurrentPosition = (): Promise<GeolocationPosition> => {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(
      (position) => resolve(position),
      (error) => reject(error),
      { 
        enableHighAccuracy: true,  // دقة عالية
        timeout: 10000,            // 10 ثواني
        maximumAge: 0              // بيانات جديدة
      }
    );
  });
};
```

**شرح الكود:**
- `navigator.geolocation`: API المتصفح للموقع
- `enableHighAccuracy`: استخدام GPS بدقة عالية
- `timeout`: الحد الأقصى للانتظار

---

## 6. الوظائف الرئيسية (Main Functions)

### 6.1 دالة تشفير بصمة الجهاز

```typescript
// الملف: server/utils/security.ts
// اللغة: TypeScript
// خوارزمية التشفير: AES-256-CBC

import crypto from "crypto";

const ALGORITHM = "aes-256-cbc";

export function encryptDeviceFingerprint(fingerprint: string): string {
  // توليد IV عشوائي لكل تشفير
  const iv = crypto.randomBytes(16);
  
  // تحويل المفتاح لـ 32 بايت
  const key = crypto.createHash('sha256')
    .update(ENCRYPTION_KEY)
    .digest();
  
  // إنشاء cipher وتشفير البيانات
  const cipher = crypto.createCipheriv(ALGORITHM, key, iv);
  let encrypted = cipher.update(fingerprint, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  // دمج IV مع النص المشفّر
  return iv.toString('hex') + ':' + encrypted;
}
```

**شرح الكود:**
- **AES-256-CBC**: خوارزمية تشفير قوية (256 بت)
- **IV (Initialization Vector)**: قيمة عشوائية تضاف لكل تشفير
- **crypto.createCipheriv()**: إنشاء كائن التشفير
- **النتيجة**: `IV:ENCRYPTED_TEXT` (مفصولة بـ `:`)

### 6.2 دالة فك التشفير

```typescript
// الملف: server/utils/security.ts

export function decryptDeviceFingerprint(encryptedData: string): string {
  // فصل IV عن النص المشفّر
  const parts = encryptedData.split(':');
  const iv = Buffer.from(parts[0], 'hex');
  const encryptedText = parts[1];
  
  // إعادة بناء المفتاح
  const key = crypto.createHash('sha256')
    .update(ENCRYPTION_KEY)
    .digest();
  
  // فك التشفير
  const decipher = crypto.createDecipheriv(ALGORITHM, key, iv);
  let decrypted = decipher.update(encryptedText, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}
```

### 6.3 دالة حساب المسافة (Haversine Formula)

```typescript
// الملف: server/utils/security.ts
// حساب المسافة بين نقطتين جغرافيتين

export function calculateDistance(
  lat1: number, lon1: number,  // النقطة الأولى
  lat2: number, lon2: number   // النقطة الثانية
): number {
  const R = 6371e3;  // نصف قطر الأرض بالمتر
  
  // تحويل للراديان
  const φ1 = (lat1 * Math.PI) / 180;
  const φ2 = (lat2 * Math.PI) / 180;
  const Δφ = ((lat2 - lat1) * Math.PI) / 180;
  const Δλ = ((lon2 - lon1) * Math.PI) / 180;

  // معادلة Haversine
  const a = Math.sin(Δφ / 2) * Math.sin(Δφ / 2) +
            Math.cos(φ1) * Math.cos(φ2) *
            Math.sin(Δλ / 2) * Math.sin(Δλ / 2);
  
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return R * c;  // المسافة بالمتر
}
```

**شرح المعادلة:**
- **Haversine Formula**: معادلة رياضية لحساب المسافة على سطح الكرة
- **R = 6371 km**: نصف قطر الأرض
- **النتيجة**: المسافة بالأمتار بين النقطتين

### 6.4 دالة التحقق من الموقع

```typescript
// الملف: server/utils/security.ts

export function isLocationWithinRadius(
  userLat: number, userLon: number,      // موقع المستخدم
  targetLat: number, targetLon: number,  // الموقع المستهدف
  radiusMeters: number                    // النطاق المسموح
): boolean {
  const distance = calculateDistance(userLat, userLon, targetLat, targetLon);
  return distance <= radiusMeters;
}
```

**شرح:**
- تحسب المسافة بين موقع المستخدم والموقع المصرح
- إذا كانت المسافة أقل من النطاق، يُسمح بالتسجيل

### 6.5 Middleware التحقق من الحضور

```typescript
// الملف: server/middleware/attendance-security.ts

export async function validateAttendance(req, res, next) {
  const userId = req.session.userId;
  const { deviceFingerprint, latitude, longitude } = req.body;

  // 1. التحقق من Rate Limiting (منع الطلبات المتكررة)
  const rateLimit = checkRateLimit(userId, 3, 60000);
  if (!rateLimit.allowed) {
    return res.status(429).json({ error: "Too many attempts" });
  }

  // 2. التحقق من الجهاز
  const user = await getUser(userId);
  const isDeviceValid = verifyDeviceFingerprint(
    deviceFingerprint, 
    user.deviceFingerprint
  );
  if (!isDeviceValid) {
    return res.status(403).json({ 
      error: "Device mismatch",
      code: "device_mismatch" 
    });
  }

  // 3. التحقق من الموقع الجغرافي
  const allowedLocations = await getAllowedLocations(user.departmentId);
  const isLocationValid = allowedLocations.some(location => 
    isLocationWithinRadius(
      latitude, longitude,
      location.latitude, location.longitude,
      location.radiusMeters
    )
  );
  if (!isLocationValid) {
    return res.status(403).json({ 
      error: "Location not allowed",
      code: "location_not_allowed" 
    });
  }

  // ✅ كل التحققات نجحت
  next();
}
```

**شرح تدفق التحقق:**
1. **Rate Limiting**: منع أكثر من 3 محاولات في الدقيقة
2. **Device Check**: مطابقة بصمة الجهاز
3. **Location Check**: التأكد من الموقع ضمن النطاق

---

## 7. نظام الأمان (Security System)

### 7.1 تشفير كلمات المرور

```typescript
// استخدام bcrypt للتشفير
import bcrypt from 'bcryptjs';

// عند إنشاء مستخدم جديد
const hashedPassword = await bcrypt.hash(password, 10);
// 10 = salt rounds (عدد جولات التشفير)

// عند تسجيل الدخول
const isValid = await bcrypt.compare(inputPassword, storedHash);
```

**لماذا bcrypt؟**
- تشفير أحادي الاتجاه (لا يمكن فك التشفير)
- Salt تلقائي يمنع هجمات Rainbow Table
- بطيء عمداً لمنع هجمات Brute Force

### 7.2 إدارة الجلسات

```typescript
// الملف: server/index.ts

app.use(session({
  secret: process.env.SESSION_SECRET,  // مفتاح سري
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: true,      // HTTPS فقط
    httpOnly: true,    // لا يمكن الوصول من JavaScript
    maxAge: 24 * 60 * 60 * 1000  // 24 ساعة
  },
  store: new PostgresSessionStore()  // حفظ في قاعدة البيانات
}));
```

### 7.3 ملخص آليات الأمان

| الآلية | التقنية | الغرض |
|--------|---------|-------|
| كلمات المرور | bcrypt | تشفير أحادي الاتجاه |
| بصمة الجهاز | AES-256-CBC | تشفير ثنائي الاتجاه |
| الجلسات | express-session | إدارة تسجيل الدخول |
| الموقع | Haversine + GPS | التحقق من التواجد |
| Rate Limiting | In-memory | منع الإساءة |
| HTTPS | TLS/SSL | تشفير الاتصال |

---

## 8. تدفق العمل (System Flow)

### 8.1 تدفق تسجيل الحضور

```
┌─────────────────────────────────────────────────────────────────┐
│                     تسجيل الحضور - Flow                         │
└─────────────────────────────────────────────────────────────────┘

المتدرب                    Frontend                    Backend
   │                          │                           │
   │  1. الضغط على            │                           │
   │     "تسجيل الحضور"       │                           │
   │─────────────────────────►│                           │
   │                          │                           │
   │                          │  2. طلب GPS من المتصفح    │
   │                          │◄─────────────────────────►│
   │                          │                           │
   │                          │  3. جلب Device Fingerprint│
   │                          │     (FingerprintJS)       │
   │                          │                           │
   │                          │  4. POST /api/attendance/ │
   │                          │     check-in              │
   │                          │─────────────────────────►│
   │                          │                           │
   │                          │     5. Rate Limit Check   │
   │                          │     ───────────────────►  │
   │                          │                           │
   │                          │     6. Device Validation  │
   │                          │     ───────────────────►  │
   │                          │                           │
   │                          │     7. GPS Validation     │
   │                          │     ───────────────────►  │
   │                          │                           │
   │                          │     8. Save to Database   │
   │                          │     ───────────────────►  │
   │                          │                           │
   │                          │  9. Response: Success     │
   │                          │◄─────────────────────────│
   │                          │                           │
   │  10. عرض رسالة           │                           │
   │      "تم تسجيل الحضور"   │                           │
   │◄─────────────────────────│                           │
   │                          │                           │
```

### 8.2 تدفق تسجيل الجهاز

```
┌─────────────────────────────────────────────────────────────────┐
│                     تسجيل الجهاز - Flow                         │
└─────────────────────────────────────────────────────────────────┘

1. المتدرب يفتح لوحته لأول مرة
2. النظام يكتشف عدم وجود جهاز مسجل
3. يظهر زر "تسجيل هذا الجهاز"
4. المتدرب يضغط الزر
5. FingerprintJS يجلب بصمة الجهاز
6. البصمة تُشفر بـ AES-256-CBC
7. تُحفظ في جدول users
8. الآن يمكن تسجيل الحضور
```

---

## 9. بيئة التشغيل (Runtime Environment)

### 9.1 متطلبات التشغيل

| المتطلب | القيمة |
|---------|--------|
| Node.js | v18 أو أحدث |
| PostgreSQL | v14 أو أحدث |
| RAM | 2GB minimum |
| Storage | 10GB minimum |

### 9.2 أنظمة التشغيل المدعومة

**النظام يعمل على جميع الأنظمة** لأنه Web Application:

| نظام التشغيل | الدعم |
|--------------|-------|
| Windows | ✅ (عبر المتصفح) |
| macOS | ✅ (عبر المتصفح) |
| Linux | ✅ (عبر المتصفح) |
| Android | ✅ (عبر المتصفح) |
| iOS | ✅ (عبر المتصفح) |

### 9.3 المتصفحات المدعومة

| المتصفح | الإصدار |
|---------|---------|
| Chrome | 90+ |
| Firefox | 88+ |
| Safari | 14+ |
| Edge | 90+ |

### 9.4 منصة الاستضافة

**Replit** - منصة سحابية للتطوير والنشر

```
مميزات الاستضافة:
- تشغيل تلقائي
- HTTPS مجاني
- قاعدة بيانات PostgreSQL
- نطاق مخصص (sarafawaz.com)
- Scaling تلقائي
```

### 9.5 المتغيرات البيئية (Environment Variables)

| المتغير | الغرض |
|---------|-------|
| `DATABASE_URL` | رابط قاعدة البيانات |
| `SESSION_SECRET` | مفتاح تشفير الجلسات |
| `DEVICE_ENCRYPTION_KEY` | مفتاح تشفير بصمات الأجهزة |
| `NODE_ENV` | بيئة التشغيل (development/production) |

---

## الخلاصة

نظام سارة هو تطبيق ويب متكامل يستخدم:

1. **React + TypeScript** للواجهة الأمامية
2. **Express.js + Node.js** للخادم
3. **PostgreSQL + Drizzle ORM** لقاعدة البيانات
4. **AES-256 + bcrypt** للأمان
5. **GPS + Device Fingerprint** للتحقق

النظام يعمل على **جميع أنظمة التشغيل** عبر المتصفح، ومستضاف على منصة **Replit** بنطاق مخصص.

---

**نهاية الوثيقة**

*إعداد: فريق تطوير نظام سارة*  
*التاريخ: ديسمبر 2024*
