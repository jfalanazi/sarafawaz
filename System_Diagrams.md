# System Diagrams - الرسوم البيانية للنظام
# نظام سارة لإدارة الحضور والموارد البشرية

**الإصدار:** 1.0  
**التاريخ:** ديسمبر 2024  
**المؤسسة:** مستشفى الملك خالد - حائل

---

## جدول المحتويات

1. [Use Case Diagram - مخطط حالات الاستخدام](#1-use-case-diagram---مخطط-حالات-الاستخدام)
2. [ER Diagram - مخطط الكيانات والعلاقات](#2-er-diagram---مخطط-الكيانات-والعلاقات)
3. [Sequence Diagrams - مخططات التسلسل](#3-sequence-diagrams---مخططات-التسلسل)
4. [Class Diagram - مخطط الفئات](#4-class-diagram---مخطط-الفئات)

---

## 1. Use Case Diagram - مخطط حالات الاستخدام

### 1.1 الوصف
يوضح هذا المخطط التفاعلات بين المستخدمين (الممثلين) والنظام، مع تحديد الوظائف المتاحة لكل دور.

### 1.2 الممثلون (Actors)

| الممثل | الوصف (عربي) | Description |
|--------|--------------|-------------|
| متدرب (Trainee) | المستخدم الأساسي الذي يسجل حضوره | Primary user who records attendance |
| مشرف (Supervisor) | مسؤول عن إدارة قسم معين ويرث صلاحيات المتدرب | Responsible for managing a department, inherits Trainee permissions |
| مدير (Manager) | الإدارة العليا مع صلاحيات كاملة ويرث صلاحيات المشرف | Top management with full access, inherits Supervisor permissions |

### 1.3 مخطط حالات الاستخدام (PlantUML Text Notation)

```
@startuml Sara Portal Use Case Diagram

left to right direction
skinparam packageStyle rectangle

actor "متدرب\n(Trainee)" as Trainee
actor "مشرف\n(Supervisor)" as Supervisor
actor "مدير\n(Manager)" as Manager

Supervisor --|> Trainee : extends
Manager --|> Supervisor : extends

rectangle "نظام سارة للحضور\nSara Attendance System" {
    
    ' === حالات استخدام المتدرب ===
    usecase "UC-01: تسجيل الدخول\nLogin" as UC01
    usecase "UC-02: تسجيل الخروج\nLogout" as UC02
    usecase "UC-03: تسجيل الحضور\nCheck-In" as UC03
    usecase "UC-04: تسجيل الانصراف\nCheck-Out" as UC04
    usecase "UC-05: عرض السجل الشخصي\nView Personal Records" as UC05
    usecase "UC-06: تسجيل الجهاز\nRegister Device" as UC06
    usecase "UC-07: طلب تغيير الجهاز\nRequest Device Change" as UC07
    usecase "UC-08: عرض الإحصائيات\nView Statistics" as UC08
    
    ' === حالات استخدام المشرف ===
    usecase "UC-10: إدارة مواقع القسم\nManage Department Locations" as UC10
    usecase "UC-11: إدارة جداول القسم\nManage Department Schedules" as UC11
    usecase "UC-12: الموافقة على طلبات الأجهزة\nApprove Device Requests" as UC12
    usecase "UC-13: عرض تقارير القسم\nView Department Reports" as UC13
    usecase "UC-14: إدارة متدربي القسم\nManage Department Trainees" as UC14
    
    ' === حالات استخدام المدير ===
    usecase "UC-20: إدارة الأقسام\nManage Departments" as UC20
    usecase "UC-21: إدارة المستخدمين\nManage Users" as UC21
    usecase "UC-22: إدارة المواقع العامة\nManage Global Locations" as UC22
    usecase "UC-23: إعدادات النظام\nSystem Settings" as UC23
    usecase "UC-24: عرض التقارير الشاملة\nView All Reports" as UC24
    usecase "UC-25: تصدير التقارير\nExport Reports" as UC25
    
    ' === حالات استخدام مضمنة (Include) ===
    usecase "التحقق من GPS\nVerify GPS Location" as VerifyGPS
    usecase "التحقق من بصمة الجهاز\nVerify Device Fingerprint" as VerifyDevice
    usecase "تشفير البيانات\nEncrypt Data" as EncryptData
    
    ' === علاقات المتدرب ===
    Trainee --> UC01
    Trainee --> UC02
    Trainee --> UC03
    Trainee --> UC04
    Trainee --> UC05
    Trainee --> UC06
    Trainee --> UC07
    Trainee --> UC08
    
    ' === علاقات المشرف (إضافية) ===
    Supervisor --> UC10
    Supervisor --> UC11
    Supervisor --> UC12
    Supervisor --> UC13
    Supervisor --> UC14
    
    ' === علاقات المدير (إضافية) ===
    Manager --> UC20
    Manager --> UC21
    Manager --> UC22
    Manager --> UC23
    Manager --> UC24
    Manager --> UC25
    
    ' === علاقات Include ===
    UC03 ..> VerifyGPS : <<include>>
    UC03 ..> VerifyDevice : <<include>>
    UC04 ..> VerifyGPS : <<include>>
    UC04 ..> VerifyDevice : <<include>>
    UC06 ..> EncryptData : <<include>>
    VerifyDevice ..> EncryptData : <<include>>
    
    ' === علاقات Extend ===
    UC25 ..> UC13 : <<extend>>
    UC25 ..> UC24 : <<extend>>
}

@enduml
```

### 1.4 تمثيل نصي للعلاقات (Text-Based Diagram)

```
+-----------------------------------------------------------------------------------+
|                         نظام سارة للحضور (Sara Attendance System)                  |
+-----------------------------------------------------------------------------------+
|                                                                                   |
|    ┌─────────────────────────────────────────────────────────────────────────┐   |
|    │                    حالات استخدام المتدرب (Trainee)                       │   |
|    ├─────────────────────────────────────────────────────────────────────────┤   |
|    │  ○ UC-01: تسجيل الدخول                                                  │   |
|    │  ○ UC-02: تسجيل الخروج                                                  │   |
|    │  ○ UC-03: تسجيل الحضور ──────<<include>>──── التحقق من GPS             │   |
|    │                        └───<<include>>──── التحقق من بصمة الجهاز        │   |
|    │  ○ UC-04: تسجيل الانصراف ────<<include>>──── التحقق من GPS             │   |
|    │                          └───<<include>>──── التحقق من بصمة الجهاز      │   |
|    │  ○ UC-05: عرض السجل الشخصي                                              │   |
|    │  ○ UC-06: تسجيل الجهاز ──────<<include>>──── تشفير البيانات             │   |
|    │  ○ UC-07: طلب تغيير الجهاز                                              │   |
|    │  ○ UC-08: عرض الإحصائيات الشخصية                                        │   |
|    └─────────────────────────────────────────────────────────────────────────┘   |
|           ▲                                                                       |
|           │ <<extends>>                                                           |
|           │                                                                       |
|    ┌─────────────────────────────────────────────────────────────────────────┐   |
|    │                حالات استخدام المشرف (Supervisor) - إضافية               │   |
|    ├─────────────────────────────────────────────────────────────────────────┤   |
|    │  ○ UC-10: إدارة مواقع القسم                                             │   |
|    │  ○ UC-11: إدارة جداول عمل القسم                                         │   |
|    │  ○ UC-12: الموافقة على طلبات الأجهزة                                    │   |
|    │  ○ UC-13: عرض تقارير القسم ─────<<extend>>───── تصدير PDF/Excel        │   |
|    │  ○ UC-14: إدارة متدربي القسم                                            │   |
|    └─────────────────────────────────────────────────────────────────────────┘   |
|           ▲                                                                       |
|           │ <<extends>>                                                           |
|           │                                                                       |
|    ┌─────────────────────────────────────────────────────────────────────────┐   |
|    │                  حالات استخدام المدير (Manager) - إضافية                │   |
|    ├─────────────────────────────────────────────────────────────────────────┤   |
|    │  ○ UC-20: إدارة جميع الأقسام                                            │   |
|    │  ○ UC-21: إدارة المستخدمين والصلاحيات                                   │   |
|    │  ○ UC-22: إدارة المواقع العامة                                          │   |
|    │  ○ UC-23: إعدادات النظام                                                │   |
|    │  ○ UC-24: عرض التقارير الشاملة ─────<<extend>>───── تصدير PDF/Excel    │   |
|    │  ○ UC-25: تصدير التقارير (PDF/Excel)                                    │   |
|    └─────────────────────────────────────────────────────────────────────────┘   |
|                                                                                   |
+-----------------------------------------------------------------------------------+

الممثلون (Actors):
  [متدرب] ────────────────┐
      ▲                    │
      │ extends            │ يستخدم جميع حالات المتدرب
  [مشرف] ─────────────────┤
      ▲                    │ يستخدم جميع حالات المشرف + المتدرب
      │ extends            │
  [مدير] ─────────────────┘ يستخدم جميع حالات النظام
```

### 1.5 جدول حالات الاستخدام التفصيلي

#### حالات الاستخدام للمتدرب (Trainee Use Cases)

| معرف | حالة الاستخدام | الوصف | المتطلبات المسبقة | Include/Extend |
|------|----------------|-------|-------------------|----------------|
| UC-01 | تسجيل الدخول | إدخال اسم المستخدم وكلمة المرور | حساب مفعّل | - |
| UC-02 | تسجيل الخروج | إنهاء الجلسة الحالية | مُسجّل الدخول | - |
| UC-03 | تسجيل الحضور | تسجيل بداية الدوام | جهاز مسجل + موقع مصرح | <<include>> GPS, Device |
| UC-04 | تسجيل الانصراف | تسجيل نهاية الدوام | تسجيل حضور سابق | <<include>> GPS, Device |
| UC-05 | عرض السجل الشخصي | مراجعة سجلات الحضور | مُسجّل الدخول | - |
| UC-06 | تسجيل الجهاز | ربط الجهاز بالحساب | جهاز غير مسجل | <<include>> Encrypt |
| UC-07 | طلب تغيير الجهاز | طلب استبدال الجهاز | جهاز مسجل سابقاً | - |
| UC-08 | عرض الإحصائيات | مراجعة إحصائيات الحضور | مُسجّل الدخول | - |

#### حالات الاستخدام للمشرف (Supervisor Use Cases)

| معرف | حالة الاستخدام | الوصف | النطاق |
|------|----------------|-------|--------|
| UC-10 | إدارة مواقع القسم | إضافة/تعديل/حذف المواقع المصرح بها | القسم فقط |
| UC-11 | إدارة جداول القسم | إضافة/تعديل جداول العمل | القسم فقط |
| UC-12 | الموافقة على الطلبات | مراجعة طلبات تغيير الأجهزة | متدربي القسم |
| UC-13 | عرض تقارير القسم | تقارير حضور القسم | القسم فقط |
| UC-14 | إدارة المتدربين | عرض وإدارة متدربي القسم | القسم فقط |

#### حالات الاستخدام للمدير (Manager Use Cases)

| معرف | حالة الاستخدام | الوصف | النطاق |
|------|----------------|-------|--------|
| UC-20 | إدارة الأقسام | إضافة/تعديل/حذف الأقسام | النظام كاملاً |
| UC-21 | إدارة المستخدمين | إضافة/تعديل المستخدمين والصلاحيات | جميع المستخدمين |
| UC-22 | إدارة المواقع العامة | مواقع تسجيل لجميع الأقسام | جميع الأقسام |
| UC-23 | إعدادات النظام | تكوين إعدادات النظام | النظام كاملاً |
| UC-24 | التقارير الشاملة | تقارير على مستوى المؤسسة | جميع الأقسام |
| UC-25 | تصدير التقارير | تصدير PDF و Excel | جميع التقارير |

---

## 2. ER Diagram - مخطط الكيانات والعلاقات

### 2.1 الوصف
يوضح هذا المخطط هيكل قاعدة البيانات والعلاقات بين الجداول مع الـ Cardinality الصحيحة.

### 2.2 ملاحظات حول العلاقات (Cardinality)
- `||--||` : One to One (إلزامي)
- `||--o|` : One to Zero or One (اختياري)
- `||--|{` : One to Many (إلزامي على الأقل واحد)
- `||--o{` : One to Zero or Many (اختياري - صفر أو أكثر)
- `}|--|{` : Many to Many

### 2.3 مخطط ER (Mermaid)

```mermaid
erDiagram
    USERS ||--o{ ATTENDANCE_RECORDS : "يسجل"
    USERS ||--o{ DEVICE_CHANGE_REQUESTS : "يطلب"
    USERS ||--o{ GROUP_MEMBERS : "ينتمي"
    USERS ||--o{ GROUPS : "يشرف على"
    USERS }o--o| LOCATIONS : "ينتمي لقسم"
    
    LOCATIONS ||--o{ USERS : "يحتوي"
    LOCATIONS ||--o{ GROUPS : "يحتوي"
    LOCATIONS ||--o{ ALLOWED_LOCATIONS : "يحتوي مواقع"
    LOCATIONS ||--o{ WORK_SCHEDULES : "يحتوي جداول"
    
    GROUPS ||--o{ GROUP_MEMBERS : "يحتوي"
    GROUPS ||--o{ ATTENDANCE_RECORDS : "مرتبط"
    GROUPS }o--o| LOCATIONS : "تابع لقسم"
    
    ALLOWED_LOCATIONS }o--o| LOCATIONS : "تابع لقسم"
    
    WORK_SCHEDULES }o--o| LOCATIONS : "تابع لقسم"
    
    DEVICE_CHANGE_REQUESTS }o--o| USERS : "مراجع"
    
    ATTENDANCE_ATTEMPTS }o--o| USERS : "محاولة من"
    
    QR_CODES ||--o{ ATTENDANCE_RECORDS : "مستخدم في"

    USERS {
        varchar id PK "UUID - المعرف الفريد"
        text username UK "اسم المستخدم - فريد"
        text password "كلمة المرور - bcrypt مشفرة"
        text name "الاسم الكامل"
        text email "البريد الإلكتروني - اختياري"
        enum role "الدور: employee|supervisor|manager"
        varchar department_id FK "القسم - اختياري"
        text device_fingerprint "بصمة الجهاز - AES-256 مشفرة"
        timestamp created_at "تاريخ الإنشاء"
    }
    
    LOCATIONS {
        varchar id PK "UUID - المعرف الفريد"
        text name "اسم القسم"
        text description "الوصف - اختياري"
        varchar created_by FK "المنشئ - اختياري"
        timestamp created_at "تاريخ الإنشاء"
    }
    
    GROUPS {
        varchar id PK "UUID - المعرف الفريد"
        text name "اسم المجموعة"
        varchar supervisor_id FK "المشرف - إلزامي"
        varchar location_id FK "القسم - اختياري"
        timestamp created_at "تاريخ الإنشاء"
    }
    
    GROUP_MEMBERS {
        varchar id PK "UUID - المعرف الفريد"
        varchar group_id FK "المجموعة - إلزامي"
        varchar user_id FK "المستخدم - إلزامي"
        varchar added_by FK "مضاف بواسطة - اختياري"
        timestamp created_at "تاريخ الإضافة"
    }
    
    ATTENDANCE_RECORDS {
        varchar id PK "UUID - المعرف الفريد"
        varchar user_id FK "المستخدم - إلزامي"
        varchar group_id FK "المجموعة - اختياري"
        varchar location_id FK "القسم - اختياري"
        enum type "النوع: check_in|check_out"
        timestamp timestamp "وقت التسجيل"
        varchar qr_code_id FK "رمز QR - اختياري"
        text device_fingerprint "بصمة الجهاز - إلزامي"
        text latitude "خط العرض - اختياري"
        text longitude "خط الطول - اختياري"
        boolean manual_entry "إدخال يدوي"
        text manual_entry_reason "سبب الإدخال اليدوي"
        varchar manual_entry_by FK "المدخل اليدوي"
        timestamp edited_at "تاريخ التعديل"
        timestamp created_at "تاريخ الإنشاء"
    }
    
    DEVICE_CHANGE_REQUESTS {
        varchar id PK "UUID - المعرف الفريد"
        varchar user_id FK "المستخدم - إلزامي"
        text old_device_fingerprint "البصمة القديمة - اختياري"
        text new_device_fingerprint "البصمة الجديدة - إلزامي"
        text reason "السبب - إلزامي"
        enum status "الحالة: pending|approved|rejected"
        varchar reviewed_by FK "مراجع بواسطة - اختياري"
        timestamp reviewed_at "تاريخ المراجعة"
        text review_notes "ملاحظات المراجعة"
        timestamp created_at "تاريخ الإنشاء"
    }
    
    ALLOWED_LOCATIONS {
        varchar id PK "UUID - المعرف الفريد"
        text name "اسم الموقع - إلزامي"
        text latitude "خط العرض - إلزامي"
        text longitude "خط الطول - إلزامي"
        text radius_meters "النطاق بالأمتار - افتراضي 100"
        varchar department_id FK "القسم - اختياري للمواقع العامة"
        boolean active "فعّال - افتراضي true"
        varchar created_by FK "المنشئ - اختياري"
        timestamp created_at "تاريخ الإنشاء"
    }
    
    WORK_SCHEDULES {
        varchar id PK "UUID - المعرف الفريد"
        text name "اسم الجدول - إلزامي"
        varchar department_id FK "القسم - اختياري للجداول العامة"
        text start_time "وقت البداية - إلزامي"
        text end_time "وقت النهاية - إلزامي"
        text_array days_of_week "أيام العمل - إلزامي"
        boolean active "فعّال - افتراضي true"
        varchar created_by FK "المنشئ - اختياري"
        timestamp created_at "تاريخ الإنشاء"
    }
    
    ATTENDANCE_ATTEMPTS {
        varchar id PK "UUID - المعرف الفريد"
        varchar user_id FK "المستخدم - اختياري"
        enum status "الحالة: success|failed_device|failed_location|..."
        enum type "النوع: check_in|check_out"
        text device_fingerprint "بصمة الجهاز"
        text latitude "خط العرض"
        text longitude "خط الطول"
        text failure_reason "سبب الفشل"
        timestamp created_at "تاريخ المحاولة"
    }
    
    QR_CODES {
        varchar id PK "UUID - المعرف الفريد"
        text code UK "الرمز - فريد"
        timestamp expires_at "تاريخ الانتهاء"
        boolean used "مستخدم - افتراضي false"
        timestamp created_at "تاريخ الإنشاء"
    }
```

### 2.4 وصف الجداول

| الجدول | الوصف (عربي) | Description | عدد الحقول |
|--------|--------------|-------------|------------|
| users | المستخدمين | System users (trainees, supervisors, managers) | 9 |
| locations | الأقسام | Departments/Locations | 5 |
| groups | المجموعات | Work groups under supervisors | 5 |
| group_members | أعضاء المجموعات | Many-to-many user-group membership | 5 |
| attendance_records | سجلات الحضور | Check-in/check-out records | 14 |
| device_change_requests | طلبات تغيير الأجهزة | Device change approval workflow | 10 |
| allowed_locations | المواقع المصرح بها | GPS locations where attendance is allowed | 9 |
| work_schedules | جداول العمل | Working hours and days configuration | 9 |
| attendance_attempts | محاولات التسجيل | Audit log of all attendance attempts | 9 |
| qr_codes | رموز QR | Time-limited QR codes for verification | 5 |

### 2.5 جدول العلاقات مع Cardinality

| العلاقة | النوع | الإلزامية | الوصف |
|---------|-------|-----------|-------|
| users.departmentId → locations.id | Many-to-One | اختياري | مستخدم يمكن أن ينتمي لقسم واحد أو لا ينتمي لأي قسم |
| users → attendance_records | One-to-Many | - | كل مستخدم له صفر أو أكثر من سجلات الحضور |
| users → device_change_requests | One-to-Many | - | كل مستخدم يمكنه تقديم صفر أو أكثر من طلبات تغيير الجهاز |
| groups.supervisorId → users.id | Many-to-One | إلزامي | كل مجموعة يجب أن يكون لها مشرف واحد |
| groups.locationId → locations.id | Many-to-One | اختياري | المجموعة يمكن أن تنتمي لقسم أو تكون عامة |
| allowed_locations.departmentId → locations.id | Many-to-One | اختياري | الموقع يمكن أن يكون خاص بقسم أو عام |
| work_schedules.departmentId → locations.id | Many-to-One | اختياري | الجدول يمكن أن يكون خاص بقسم أو عام |
| device_change_requests.reviewedBy → users.id | Many-to-One | اختياري | الطلب قد يكون مراجع أو لا (pending) |

---

## 3. Sequence Diagrams - مخططات التسلسل

### 3.1 تسجيل الحضور (Check-In Process)

```mermaid
sequenceDiagram
    autonumber
    participant T as المتدرب
    participant UI as واجهة المستخدم
    participant API as API Server
    participant SEC as Security Middleware
    participant DB as قاعدة البيانات
    participant GPS as GPS Service
    
    T->>UI: الضغط على تسجيل الحضور
    UI->>GPS: طلب الموقع الحالي
    GPS-->>UI: إحداثيات latitude و longitude
    
    UI->>API: POST /api/attendance/check-in
    Note over UI,API: deviceFingerprint, latitude, longitude
    
    API->>SEC: التحقق من الجلسة
    SEC-->>API: المستخدم مصادق
    
    API->>SEC: التحقق من بصمة الجهاز
    SEC->>DB: مقارنة deviceFingerprint
    
    alt الجهاز غير مطابق
        SEC-->>API: فشل - جهاز غير مسجل
        API->>DB: تسجيل محاولة فاشلة في attendance_attempts
        API-->>UI: خطأ الجهاز غير مسجل
        UI-->>T: عرض رسالة الخطأ
    else الجهاز مطابق
        SEC-->>API: الجهاز صحيح
        
        API->>SEC: التحقق من الموقع الجغرافي
        SEC->>DB: جلب المواقع المصرح بها من allowed_locations
        DB-->>SEC: قائمة المواقع
        SEC->>SEC: حساب المسافة باستخدام Haversine
        
        alt الموقع خارج النطاق
            SEC-->>API: فشل - موقع غير مصرح
            API->>DB: تسجيل محاولة فاشلة
            API-->>UI: خطأ أنت خارج المنطقة المسموحة
            UI-->>T: عرض رسالة الخطأ
        else الموقع صحيح
            SEC-->>API: الموقع صحيح
            
            API->>DB: التحقق من عدم وجود حضور سابق اليوم
            DB-->>API: لا يوجد حضور اليوم
            
            API->>DB: إنشاء سجل حضور جديد type check_in
            DB-->>API: تم الإنشاء بنجاح
            
            API->>DB: تسجيل محاولة ناجحة في attendance_attempts
            API-->>UI: تم تسجيل الحضور بنجاح
            UI-->>T: عرض رسالة النجاح
        end
    end
```

### 3.2 تسجيل الانصراف (Check-Out Process)

```mermaid
sequenceDiagram
    autonumber
    participant T as المتدرب
    participant UI as واجهة المستخدم
    participant API as API Server
    participant SEC as Security Middleware
    participant DB as قاعدة البيانات
    participant GPS as GPS Service
    
    T->>UI: الضغط على تسجيل الانصراف
    UI->>GPS: طلب الموقع الحالي
    GPS-->>UI: إحداثيات latitude و longitude
    
    UI->>API: POST /api/attendance/check-out
    Note over UI,API: deviceFingerprint, latitude, longitude
    
    API->>SEC: التحقق من الجلسة والجهاز والموقع
    SEC-->>API: جميع التحققات ناجحة
    
    API->>DB: البحث عن آخر تسجيل حضور check_in اليوم
    DB-->>API: سجل الحضور الأخير
    
    alt لا يوجد حضور سابق
        API-->>UI: خطأ لم تسجل حضورك اليوم
        UI-->>T: عرض رسالة الخطأ
    else يوجد حضور سابق
        API->>DB: التحقق من عدم وجود انصراف مسبق
        
        alt انصراف مسجل مسبقا
            API-->>UI: خطأ تم تسجيل الانصراف مسبقا
            UI-->>T: عرض رسالة الخطأ
        else لم يسجل انصراف
            API->>DB: إنشاء سجل انصراف type check_out
            DB-->>API: تم الإنشاء بنجاح
            
            API->>API: حساب ساعات العمل
            API-->>UI: تم تسجيل الانصراف بنجاح مع عرض ساعات العمل
            UI-->>T: عرض رسالة النجاح
        end
    end
```

### 3.3 تسجيل الجهاز (Device Registration)

```mermaid
sequenceDiagram
    autonumber
    participant T as المتدرب
    participant UI as واجهة المستخدم
    participant FP as FingerprintJS
    participant API as API Server
    participant ENC as Encryption Service
    participant DB as قاعدة البيانات
    
    T->>UI: الدخول لصفحة تسجيل الجهاز
    UI->>FP: طلب بصمة الجهاز
    FP->>FP: جمع معلومات الجهاز Browser OS Screen Canvas WebGL
    FP-->>UI: deviceFingerprint hash
    
    UI->>API: POST /api/users/register-device
    Note over UI,API: deviceFingerprint
    
    API->>DB: التحقق من المستخدم
    DB-->>API: بيانات المستخدم
    
    alt الجهاز مسجل مسبقا
        API-->>UI: خطأ لديك جهاز مسجل
        UI-->>T: عرض رسالة يمكنك طلب التغيير
    else لا يوجد جهاز مسجل
        API->>ENC: تشفير البصمة باستخدام AES-256-CBC
        ENC-->>API: البصمة المشفرة
        
        API->>DB: تحديث deviceFingerprint في جدول users
        DB-->>API: تم التحديث
        
        API-->>UI: تم تسجيل الجهاز بنجاح
        UI-->>T: عرض رسالة النجاح
    end
```

### 3.4 طلب تغيير الجهاز (Device Change Request)

```mermaid
sequenceDiagram
    autonumber
    participant T as المتدرب
    participant UI as واجهة المستخدم
    participant API as API Server
    participant DB as قاعدة البيانات
    participant S as المشرف
    
    T->>UI: تقديم طلب تغيير الجهاز
    UI->>API: POST /api/device-requests
    Note over UI,API: newDeviceFingerprint, reason
    
    API->>DB: إنشاء طلب جديد status pending
    DB-->>API: تم الإنشاء
    API-->>UI: تم تقديم الطلب
    UI-->>T: الطلب قيد المراجعة
    
    Note over T,S: لاحقا يراجع المشرف
    
    S->>UI: فتح قائمة الطلبات المعلقة
    UI->>API: GET /api/device-requests
    API->>DB: جلب الطلبات
    DB-->>API: قائمة الطلبات
    API-->>UI: عرض الطلبات
    
    S->>UI: مراجعة واتخاذ قرار
    
    alt الموافقة
        S->>UI: الموافقة على الطلب
        UI->>API: PATCH /api/device-requests/:id
        Note over UI,API: status approved
        
        API->>DB: تحديث حالة الطلب
        API->>DB: تحديث deviceFingerprint للمستخدم
        DB-->>API: تم التحديث
        API-->>UI: تمت الموافقة
    else الرفض
        S->>UI: رفض الطلب
        UI->>API: PATCH /api/device-requests/:id
        Note over UI,API: status rejected, reviewNotes
        
        API->>DB: تحديث حالة الطلب
        DB-->>API: تم التحديث
        API-->>UI: تم الرفض
    end
```

### 3.5 تسجيل الدخول (Login Process)

```mermaid
sequenceDiagram
    autonumber
    participant U as المستخدم
    participant UI as واجهة المستخدم
    participant API as API Server
    participant AUTH as Auth Service
    participant DB as قاعدة البيانات
    participant SESSION as Session Store
    
    U->>UI: إدخال اسم المستخدم وكلمة المرور
    UI->>API: POST /api/auth/login
    Note over UI,API: username, password
    
    API->>DB: البحث عن المستخدم
    DB-->>API: بيانات المستخدم
    
    alt المستخدم غير موجود
        API-->>UI: خطأ بيانات غير صحيحة
        UI-->>U: عرض رسالة الخطأ
    else المستخدم موجود
        API->>AUTH: التحقق من كلمة المرور
        AUTH->>AUTH: bcrypt.compare
        
        alt كلمة المرور خاطئة
            AUTH-->>API: فشل المطابقة
            API-->>UI: خطأ بيانات غير صحيحة
            UI-->>U: عرض رسالة الخطأ
        else كلمة المرور صحيحة
            AUTH-->>API: نجاح المطابقة
            
            API->>SESSION: إنشاء جلسة جديدة
            SESSION-->>API: Session ID
            
            API-->>UI: تم تسجيل الدخول
            
            alt المستخدم متدرب employee
                UI->>UI: التوجيه للوحة المتدرب
            else المستخدم مشرف supervisor
                UI->>UI: التوجيه للوحة المشرف
            else المستخدم مدير manager
                UI->>UI: التوجيه للوحة المدير
            end
            
            UI-->>U: عرض لوحة التحكم
        end
    end
```

### 3.6 إنشاء تقرير (Report Generation)

```mermaid
sequenceDiagram
    autonumber
    participant M as المدير او المشرف
    participant UI as واجهة المستخدم
    participant API as API Server
    participant DB as قاعدة البيانات
    participant PDF as PDF Generator jspdf
    participant XLS as Excel Generator xlsx
    
    M->>UI: فتح صفحة التقارير
    M->>UI: تحديد الفلاتر التاريخ القسم نوع التقرير
    
    UI->>API: GET /api/reports/daily
    Note over UI,API: startDate endDate departmentId
    
    API->>DB: جلب سجلات الحضور مع الفلاتر المحددة
    DB-->>API: سجلات الحضور
    
    API->>API: معالجة وتجميع البيانات وحساب الإحصائيات
    
    API-->>UI: بيانات التقرير
    UI-->>M: عرض التقرير
    
    alt تصدير PDF
        M->>UI: الضغط على تصدير PDF
        UI->>PDF: إنشاء مستند PDF
        PDF->>PDF: تنسيق البيانات
        PDF-->>UI: ملف PDF
        UI-->>M: تحميل الملف
    else تصدير Excel
        M->>UI: الضغط على تصدير Excel
        UI->>XLS: إنشاء ملف Excel
        XLS->>XLS: تنسيق البيانات
        XLS-->>UI: ملف XLSX
        UI-->>M: تحميل الملف
    end
```

---

## 4. Class Diagram - مخطط الفئات

### 4.1 الوصف
يوضح هذا المخطط البنية الهيكلية للكود من ناحية الفئات والخصائص والعلاقات.

### 4.2 مخطط الفئات (Mermaid)

```mermaid
classDiagram
    direction TB
    
    class User {
        +String id
        +String username
        +String password
        +String name
        +String email
        +Role role
        +String departmentId
        +String deviceFingerprint
        +Date createdAt
        +login() Boolean
        +logout() void
        +checkIn(latitude, longitude) AttendanceRecord
        +checkOut(latitude, longitude) AttendanceRecord
        +registerDevice(fingerprint) Boolean
        +requestDeviceChange(newFingerprint, reason) DeviceChangeRequest
    }
    
    class Location {
        +String id
        +String name
        +String description
        +String createdBy
        +Date createdAt
        +getEmployees() User[]
        +getGroups() Group[]
        +getAllowedLocations() AllowedLocation[]
        +getWorkSchedules() WorkSchedule[]
    }
    
    class Group {
        +String id
        +String name
        +String supervisorId
        +String locationId
        +Date createdAt
        +addMember(userId) GroupMember
        +removeMember(userId) Boolean
        +getMembers() User[]
    }
    
    class GroupMember {
        +String id
        +String groupId
        +String userId
        +String addedBy
        +Date createdAt
    }
    
    class AttendanceRecord {
        +String id
        +String userId
        +String groupId
        +String locationId
        +AttendanceType type
        +Date timestamp
        +String deviceFingerprint
        +String latitude
        +String longitude
        +Boolean manualEntry
        +String manualEntryReason
        +Date createdAt
        +calculateWorkHours() Number
    }
    
    class DeviceChangeRequest {
        +String id
        +String userId
        +String oldDeviceFingerprint
        +String newDeviceFingerprint
        +String reason
        +RequestStatus status
        +String reviewedBy
        +Date reviewedAt
        +String reviewNotes
        +Date createdAt
        +approve(reviewerId, notes) Boolean
        +reject(reviewerId, notes) Boolean
    }
    
    class AllowedLocation {
        +String id
        +String name
        +String latitude
        +String longitude
        +String radiusMeters
        +String departmentId
        +Boolean active
        +String createdBy
        +Date createdAt
        +isWithinRadius(lat, lon) Boolean
    }
    
    class WorkSchedule {
        +String id
        +String name
        +String departmentId
        +String startTime
        +String endTime
        +String[] daysOfWeek
        +Boolean active
        +String createdBy
        +Date createdAt
        +isWorkingTime(datetime) Boolean
    }
    
    class AttendanceAttempt {
        +String id
        +String userId
        +AttemptStatus status
        +AttendanceType type
        +String deviceFingerprint
        +String latitude
        +String longitude
        +String failureReason
        +Date createdAt
    }
    
    class QRCode {
        +String id
        +String code
        +Date expiresAt
        +Boolean used
        +Date createdAt
        +isValid() Boolean
        +markAsUsed() void
    }
    
    class SecurityService {
        <<service>>
        +encryptFingerprint(plaintext) String
        +decryptFingerprint(ciphertext) String
        +verifyFingerprint(input, stored) Boolean
        +hashPassword(password) String
        +verifyPassword(password, hash) Boolean
    }
    
    class GPSService {
        <<service>>
        +calculateDistance(lat1, lon1, lat2, lon2) Number
        +isWithinRadius(userLat, userLon, locLat, locLon, radius) Boolean
        +getHaversineDistance(lat1, lon1, lat2, lon2) Number
    }
    
    class AuthService {
        <<service>>
        +login(username, password) Session
        +logout(sessionId) void
        +validateSession(sessionId) User
        +createSession(userId) String
        +destroySession(sessionId) void
    }
    
    class ReportService {
        <<service>>
        +generateDailyReport(date, departmentId) Report
        +generateMonthlyReport(year, month, departmentId) Report
        +exportToPDF(data) File
        +exportToExcel(data) File
        +calculateStatistics(records) Statistics
    }
    
    class Role {
        <<enumeration>>
        EMPLOYEE
        SUPERVISOR
        MANAGER
    }
    
    class AttendanceType {
        <<enumeration>>
        CHECK_IN
        CHECK_OUT
    }
    
    class RequestStatus {
        <<enumeration>>
        PENDING
        APPROVED
        REJECTED
    }
    
    class AttemptStatus {
        <<enumeration>>
        SUCCESS
        FAILED_DEVICE
        FAILED_LOCATION
        FAILED_SCHEDULE
        FAILED_RATE_LIMIT
    }
    
    User "1" --> "0..*" AttendanceRecord : records
    User "1" --> "0..*" DeviceChangeRequest : requests
    User "1" --> "0..*" GroupMember : memberships
    User "0..*" --> "0..1" Location : department
    User "1" --> "0..*" Group : supervises
    
    Location "1" --> "0..*" User : employees
    Location "1" --> "0..*" Group : groups
    Location "1" --> "0..*" AllowedLocation : allowedLocations
    Location "1" --> "0..*" WorkSchedule : schedules
    
    Group "1" --> "0..*" GroupMember : members
    Group "0..*" --> "1" User : supervisor
    Group "0..*" --> "0..1" Location : location
    
    AllowedLocation "0..*" --> "0..1" Location : department
    WorkSchedule "0..*" --> "0..1" Location : department
    
    DeviceChangeRequest "0..*" --> "0..1" User : reviewer
    
    AttendanceRecord ..> SecurityService : uses
    AttendanceRecord ..> GPSService : uses
    
    User ..> AuthService : uses
    User ..> SecurityService : uses
    
    AllowedLocation ..> GPSService : uses
```

### 4.3 وصف الفئات الرئيسية

#### فئات الكيانات (Entity Classes)

| الفئة | الوصف | المسؤولية الرئيسية |
|-------|-------|-------------------|
| User | المستخدم | تمثيل بيانات المستخدم وعملياته (حضور، جهاز، صلاحيات) |
| Location | القسم | تمثيل الأقسام التنظيمية وعلاقاتها |
| Group | المجموعة | تمثيل مجموعات العمل تحت إشراف مشرف |
| AttendanceRecord | سجل الحضور | تخزين بيانات تسجيل الحضور والانصراف |
| DeviceChangeRequest | طلب تغيير الجهاز | إدارة سير عمل طلبات تغيير الأجهزة |
| AllowedLocation | الموقع المصرح | تحديد المواقع الجغرافية المسموحة للتسجيل |
| WorkSchedule | جدول العمل | تحديد أوقات وأيام العمل |
| AttendanceAttempt | محاولة التسجيل | سجل تدقيق لجميع محاولات الحضور |
| QRCode | رمز QR | إدارة رموز التحقق المؤقتة |

#### فئات الخدمات (Service Classes)

| الفئة | الوصف | التقنيات المستخدمة |
|-------|-------|-------------------|
| SecurityService | خدمة الأمان | AES-256-CBC, bcrypt |
| GPSService | خدمة GPS | Haversine formula |
| AuthService | خدمة المصادقة | express-session, passport |
| ReportService | خدمة التقارير | jspdf, xlsx |

### 4.4 التعدادات (Enumerations)

| التعداد | القيم | الاستخدام |
|---------|-------|-----------|
| Role | employee, supervisor, manager | تحديد مستوى صلاحيات المستخدم |
| AttendanceType | check_in, check_out | نوع سجل الحضور |
| RequestStatus | pending, approved, rejected | حالة طلب تغيير الجهاز |
| AttemptStatus | success, failed_device, failed_location, failed_schedule, failed_rate_limit | نتيجة محاولة التسجيل |

---

## 5. خوارزميات رئيسية

### 5.1 خوارزمية Haversine لحساب المسافة

تُستخدم لحساب المسافة بين نقطتين على سطح الأرض بناءً على إحداثيات GPS.

```
المدخلات: (lat1, lon1), (lat2, lon2) بالدرجات
المخرجات: المسافة بالأمتار

الخوارزمية:
1. R = 6371000 (نصف قطر الأرض بالأمتار)
2. φ1 = lat1 × π/180 (تحويل لراديان)
3. φ2 = lat2 × π/180
4. Δφ = (lat2 - lat1) × π/180
5. Δλ = (lon2 - lon1) × π/180
6. a = sin²(Δφ/2) + cos(φ1) × cos(φ2) × sin²(Δλ/2)
7. c = 2 × atan2(√a, √(1-a))
8. distance = R × c
```

### 5.2 تشفير AES-256-CBC

يُستخدم لتشفير بصمة الجهاز قبل تخزينها في قاعدة البيانات.

```
المدخلات: النص الأصلي (deviceFingerprint), مفتاح التشفير (DEVICE_ENCRYPTION_KEY)
المخرجات: النص المشفر (Base64)

الخوارزمية:
1. إنشاء IV عشوائي (16 bytes) باستخدام crypto.randomBytes
2. إنشاء cipher باستخدام AES-256-CBC والمفتاح والـ IV
3. تشفير النص: encrypted = cipher.update(plaintext) + cipher.final()
4. دمج IV مع النص المشفر: result = IV + encrypted
5. تحويل إلى Base64: output = result.toString('base64')
```

### 5.3 تشفير كلمة المرور (bcrypt)

```
التشفير:
1. توليد salt بـ 10 rounds
2. hash = bcrypt.hash(password, salt)

التحقق:
1. isMatch = bcrypt.compare(inputPassword, storedHash)
```

---

**نهاية الوثيقة**

*إعداد: فريق تطوير نظام سارة*  
*التاريخ: ديسمبر 2024*
