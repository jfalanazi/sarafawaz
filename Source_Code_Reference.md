# Sara Portal - Source Code Reference
# نظام الحضور وإدارة الموارد البشرية

**Version:** 1.0  
**Date:** December 2024  
**Project:** King Khalid Hospital - Trainee Attendance System

---

## Table of Contents

1. [Database Schema (shared/schema.ts)](#1-database-schema)
2. [API Routes (server/routes.ts)](#2-api-routes)
3. [Storage Interface (server/storage.ts)](#3-storage-interface)
4. [Security Module (server/utils/security.ts)](#4-security-module)
5. [Attendance Security Middleware](#5-attendance-security-middleware)
6. [Frontend Router (client/src/App.tsx)](#6-frontend-router)

---

## 1. Database Schema

**File:** `shared/schema.ts`

This file defines all database tables using Drizzle ORM.

### Tables:
- `users` - User accounts (trainees, supervisors, managers)
- `locations` - Departments
- `groups` - Work groups
- `groupMembers` - Group membership (many-to-many)
- `attendanceRecords` - Check-in/Check-out records
- `deviceChangeRequests` - Device change approval requests
- `allowedLocations` - GPS locations for attendance
- `workSchedules` - Work schedule configurations
- `attendanceAttempts` - Audit log for all attempts

```typescript
import { sql } from "drizzle-orm";
import { pgTable, text, varchar, timestamp, boolean, pgEnum } from "drizzle-orm/pg-core";
import { relations } from "drizzle-orm";
import { createInsertSchema } from "drizzle-zod";
import { z } from "zod";

// Enums
export const roleEnum = pgEnum("role", ["employee", "supervisor", "manager"]);
export const deviceRequestStatusEnum = pgEnum("device_request_status", ["pending", "approved", "rejected"]);
export const attendanceTypeEnum = pgEnum("attendance_type", ["check_in", "check_out"]);
export const attemptStatusEnum = pgEnum("attempt_status", ["success", "failed_device", "failed_location", "failed_schedule", "failed_rate_limit"]);

// ==================== TABLES ====================

// Users table
export const users = pgTable("users", {
  id: varchar("id", { length: 36 }).primaryKey().default(sql`gen_random_uuid()`),
  username: text("username").notNull().unique(),
  password: text("password").notNull(),
  name: text("name").notNull(),
  email: text("email"),
  role: roleEnum("role").notNull().default("employee"),
  departmentId: varchar("department_id", { length: 36 }).references(() => locations.id, { onDelete: 'set null' }),
  deviceFingerprint: text("device_fingerprint"),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});

// Locations table (Departments)
export const locations = pgTable("locations", {
  id: varchar("id", { length: 36 }).primaryKey().default(sql`gen_random_uuid()`),
  name: text("name").notNull(),
  description: text("description"),
  createdBy: varchar("created_by", { length: 36 }).references(() => users.id, { onDelete: 'set null' }),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});

// Groups table
export const groups = pgTable("groups", {
  id: varchar("id", { length: 36 }).primaryKey().default(sql`gen_random_uuid()`),
  name: text("name").notNull(),
  supervisorId: varchar("supervisor_id", { length: 36 }).notNull().references(() => users.id, { onDelete: 'cascade' }),
  locationId: varchar("location_id", { length: 36 }).references(() => locations.id, { onDelete: 'set null' }),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});

// Attendance records table
export const attendanceRecords = pgTable("attendance_records", {
  id: varchar("id", { length: 36 }).primaryKey().default(sql`gen_random_uuid()`),
  userId: varchar("user_id", { length: 36 }).notNull().references(() => users.id, { onDelete: 'cascade' }),
  groupId: varchar("group_id", { length: 36 }).references(() => groups.id, { onDelete: 'set null' }),
  locationId: varchar("location_id", { length: 36 }).references(() => locations.id, { onDelete: 'set null' }),
  type: attendanceTypeEnum("type").notNull(),
  timestamp: timestamp("timestamp").notNull().defaultNow(),
  deviceFingerprint: text("device_fingerprint").notNull(),
  latitude: text("latitude"),
  longitude: text("longitude"),
  manualEntry: boolean("manual_entry").notNull().default(false),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});

// Device change requests table
export const deviceChangeRequests = pgTable("device_change_requests", {
  id: varchar("id", { length: 36 }).primaryKey().default(sql`gen_random_uuid()`),
  userId: varchar("user_id", { length: 36 }).notNull().references(() => users.id, { onDelete: 'cascade' }),
  oldDeviceFingerprint: text("old_device_fingerprint"),
  newDeviceFingerprint: text("new_device_fingerprint").notNull(),
  reason: text("reason").notNull(),
  status: deviceRequestStatusEnum("status").notNull().default("pending"),
  reviewedBy: varchar("reviewed_by", { length: 36 }).references(() => users.id, { onDelete: 'set null' }),
  reviewedAt: timestamp("reviewed_at"),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});

// Allowed GPS Locations table
export const allowedLocations = pgTable("allowed_locations", {
  id: varchar("id", { length: 36 }).primaryKey().default(sql`gen_random_uuid()`),
  name: text("name").notNull(),
  latitude: text("latitude").notNull(),
  longitude: text("longitude").notNull(),
  radiusMeters: text("radius_meters").notNull().default("100"),
  departmentId: varchar("department_id", { length: 36 }).references(() => locations.id, { onDelete: 'cascade' }),
  active: boolean("active").notNull().default(true),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});

// Work Schedules table
export const workSchedules = pgTable("work_schedules", {
  id: varchar("id", { length: 36 }).primaryKey().default(sql`gen_random_uuid()`),
  name: text("name").notNull(),
  departmentId: varchar("department_id", { length: 36 }).references(() => locations.id, { onDelete: 'cascade' }),
  startTime: text("start_time").notNull(),
  endTime: text("end_time").notNull(),
  daysOfWeek: text("days_of_week").array().notNull(),
  active: boolean("active").notNull().default(true),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});

// Attendance Attempts (Audit Log)
export const attendanceAttempts = pgTable("attendance_attempts", {
  id: varchar("id", { length: 36 }).primaryKey().default(sql`gen_random_uuid()`),
  userId: varchar("user_id", { length: 36 }).references(() => users.id, { onDelete: 'cascade' }),
  status: attemptStatusEnum("status").notNull(),
  type: attendanceTypeEnum("type").notNull(),
  deviceFingerprint: text("device_fingerprint"),
  latitude: text("latitude"),
  longitude: text("longitude"),
  failureReason: text("failure_reason"),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});

// ==================== TYPES ====================

export type User = typeof users.$inferSelect;
export type InsertUser = z.infer<typeof insertUserSchema>;
export type Location = typeof locations.$inferSelect;
export type Group = typeof groups.$inferSelect;
export type AttendanceRecord = typeof attendanceRecords.$inferSelect;
export type DeviceChangeRequest = typeof deviceChangeRequests.$inferSelect;
export type AllowedLocation = typeof allowedLocations.$inferSelect;
export type WorkSchedule = typeof workSchedules.$inferSelect;
```

---

## 2. API Routes

**File:** `server/routes.ts`

This file defines all REST API endpoints.

### Key Endpoints:

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/login` | User authentication |
| POST | `/api/auth/logout` | User logout |
| GET | `/api/auth/me` | Get current user |
| POST | `/api/devices/register` | Register device fingerprint |
| POST | `/api/attendance/check-in` | Record check-in |
| POST | `/api/attendance/check-out` | Record check-out |
| GET | `/api/attendance/today` | Get today's attendance |
| GET | `/api/attendance/monthly-stats` | Monthly statistics |
| POST | `/api/device-requests/create` | Request device change |
| GET | `/api/settings/allowed-locations` | GPS locations |
| GET | `/api/settings/work-schedules` | Work schedules |

```typescript
import type { Express } from "express";
import { createServer, type Server } from "http";
import session from "express-session";
import { storage } from "./storage";
import crypto from "crypto";
import bcrypt from "bcryptjs";
import { z } from "zod";
import { validateAttendance } from "./middleware/attendance-security";
import { encryptDeviceFingerprint } from "./utils/security";

export async function registerRoutes(app: Express): Promise<Server> {
  // Session Configuration
  app.use(
    session({
      secret: process.env.SESSION_SECRET,
      resave: false,
      saveUninitialized: false,
      cookie: {
        secure: process.env.NODE_ENV === "production",
        httpOnly: true,
        maxAge: 1000 * 60 * 60 * 24, // 24 hours
      },
    })
  );

  // ==================== AUTHENTICATION ====================
  
  // Middleware: Require Authentication
  const requireAuth = (req: any, res: any, next: any) => {
    if (!req.session.userId) {
      return res.status(401).json({ error: "غير مصرح" });
    }
    next();
  };

  // Middleware: Require Specific Role
  const requireRole = (...roles: string[]) => {
    return async (req: any, res: any, next: any) => {
      const user = await storage.getUser(req.session.userId);
      if (!user || !roles.includes(user.role)) {
        return res.status(403).json({ error: "غير مصرح - صلاحيات غير كافية" });
      }
      next();
    };
  };

  // Login
  app.post("/api/auth/login", async (req, res) => {
    const { username, password } = req.body;
    const user = await storage.getUserByUsername(username);
    
    if (!user) {
      return res.status(401).json({ error: "اسم المستخدم أو كلمة المرور غير صحيحة" });
    }

    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      return res.status(401).json({ error: "اسم المستخدم أو كلمة المرور غير صحيحة" });
    }

    req.session.userId = user.id;
    res.json({ user: { id: user.id, name: user.name, role: user.role } });
  });

  // ==================== DEVICE REGISTRATION ====================
  
  app.post("/api/devices/register", requireAuth, async (req, res) => {
    const userId = req.session.userId!;
    const { deviceFingerprint } = req.body;

    const user = await storage.getUser(userId);
    
    // If device already registered, need supervisor approval
    if (user?.deviceFingerprint) {
      return res.status(400).json({ 
        error: "لديك جهاز مسجل بالفعل. يجب طلب تغيير الجهاز من المشرف.",
        code: "device_already_registered"
      });
    }

    // Encrypt and save fingerprint
    const encryptedFingerprint = encryptDeviceFingerprint(deviceFingerprint);
    await storage.updateUserDevice(userId, encryptedFingerprint);

    res.json({ success: true, message: "تم تسجيل الجهاز بنجاح" });
  });

  // ==================== ATTENDANCE (with Device + GPS Validation) ====================
  
  // Check-In with validation middleware
  app.post("/api/attendance/check-in", requireAuth, 
    (req, res, next) => { req.body.type = "check_in"; next(); },
    validateAttendance,
    async (req, res) => {
      const userId = req.session.userId!;
      const validatedData = req.validatedAttendanceData!;

      // Check if already checked in today
      const todayRecords = await storage.getTodayAttendanceForUser(userId);
      const existingCheckIn = todayRecords.find(r => r.type === "check_in");
      
      if (existingCheckIn) {
        return res.status(400).json({ error: "لقد سجلت دخولك بالفعل اليوم" });
      }

      // Create attendance record
      const record = await storage.createAttendanceRecord({
        userId,
        type: "check_in",
        deviceFingerprint: validatedData.deviceFingerprint,
        latitude: validatedData.latitude,
        longitude: validatedData.longitude,
      });

      res.json({ success: true, record, message: "تم تسجيل الدخول بنجاح" });
    }
  );

  // Check-Out with validation middleware
  app.post("/api/attendance/check-out", requireAuth,
    (req, res, next) => { req.body.type = "check_out"; next(); },
    validateAttendance,
    async (req, res) => {
      const userId = req.session.userId!;
      const validatedData = req.validatedAttendanceData!;

      // Check if checked in today
      const todayRecords = await storage.getTodayAttendanceForUser(userId);
      const checkIn = todayRecords.find(r => r.type === "check_in");
      
      if (!checkIn) {
        return res.status(400).json({ error: "يجب تسجيل الدخول أولاً" });
      }

      // Create attendance record
      const record = await storage.createAttendanceRecord({
        userId,
        type: "check_out",
        deviceFingerprint: validatedData.deviceFingerprint,
        latitude: validatedData.latitude,
        longitude: validatedData.longitude,
      });

      // Calculate hours worked
      const hoursWorked = (record.timestamp - checkIn.timestamp) / (1000 * 60 * 60);

      res.json({ 
        success: true, 
        record, 
        hoursWorked,
        message: `تم تسجيل الخروج بنجاح. عملت ${hoursWorked.toFixed(2)} ساعة` 
      });
    }
  );

  return createServer(app);
}
```

---

## 3. Storage Interface

**File:** `server/storage.ts`

This file defines the database access layer using Drizzle ORM.

```typescript
import { db } from "./db";
import { users, groups, locations, attendanceRecords, deviceChangeRequests } from "@shared/schema";
import { eq, and, gte, lte, desc } from "drizzle-orm";

export interface IStorage {
  // Users
  getUser(id: string): Promise<User | undefined>;
  getUserByUsername(username: string): Promise<User | undefined>;
  createUser(user: InsertUser): Promise<User>;
  updateUser(id: string, data: Partial<InsertUser>): Promise<User | undefined>;
  updateUserDevice(id: string, deviceFingerprint: string): Promise<User | undefined>;
  deleteUser(id: string): Promise<void>;
  getAllUsers(): Promise<User[]>;
  
  // Locations (Departments)
  getLocation(id: string): Promise<Location | undefined>;
  getAllLocations(): Promise<Location[]>;
  createLocation(location: InsertLocation): Promise<Location>;
  updateLocation(id: string, data: Partial<InsertLocation>): Promise<Location | undefined>;
  deleteLocation(id: string): Promise<void>;
  
  // Attendance Records
  createAttendanceRecord(record: InsertAttendanceRecord): Promise<AttendanceRecord>;
  getTodayAttendanceForUser(userId: string): Promise<AttendanceRecord[]>;
  getAttendanceRecordsByUser(userId: string, startDate?: Date, endDate?: Date): Promise<AttendanceRecord[]>;
  
  // Device Change Requests
  createDeviceChangeRequest(request: InsertDeviceChangeRequest): Promise<DeviceChangeRequest>;
  getPendingDeviceRequests(supervisorId?: string): Promise<DeviceChangeRequest[]>;
  updateDeviceRequestStatus(requestId: string, status: string, reviewerId: string): Promise<DeviceChangeRequest | undefined>;
  
  // Allowed Locations (GPS)
  getAllAllowedLocations(departmentId?: string): Promise<AllowedLocation[]>;
  createAllowedLocation(location: InsertAllowedLocation): Promise<AllowedLocation>;
  updateAllowedLocation(id: string, data: Partial<InsertAllowedLocation>): Promise<AllowedLocation | undefined>;
  deleteAllowedLocation(id: string): Promise<void>;
  
  // Work Schedules
  getAllWorkSchedules(departmentId?: string): Promise<WorkSchedule[]>;
  createWorkSchedule(schedule: InsertWorkSchedule): Promise<WorkSchedule>;
}

export class DatabaseStorage implements IStorage {
  // Users
  async getUser(id: string): Promise<User | undefined> {
    const [user] = await db.select().from(users).where(eq(users.id, id));
    return user;
  }

  async getUserByUsername(username: string): Promise<User | undefined> {
    const [user] = await db.select().from(users).where(eq(users.username, username));
    return user;
  }

  async createUser(insertUser: InsertUser): Promise<User> {
    const [user] = await db.insert(users).values(insertUser).returning();
    return user;
  }

  async updateUserDevice(id: string, deviceFingerprint: string): Promise<User | undefined> {
    const [user] = await db.update(users)
      .set({ deviceFingerprint })
      .where(eq(users.id, id))
      .returning();
    return user;
  }

  // Attendance Records
  async getTodayAttendanceForUser(userId: string): Promise<AttendanceRecord[]> {
    const today = new Date();
    today.setHours(0, 0, 0, 0);
    const tomorrow = new Date(today);
    tomorrow.setDate(tomorrow.getDate() + 1);

    return await db
      .select()
      .from(attendanceRecords)
      .where(
        and(
          eq(attendanceRecords.userId, userId),
          gte(attendanceRecords.timestamp, today),
          lte(attendanceRecords.timestamp, tomorrow)
        )
      )
      .orderBy(attendanceRecords.timestamp);
  }
}

export const storage = new DatabaseStorage();
```

---

## 4. Security Module

**File:** `server/utils/security.ts`

This file contains all security functions including encryption, GPS calculations, and rate limiting.

### Security Features:
- **AES-256-CBC Encryption** for device fingerprints
- **Haversine Formula** for GPS distance calculation
- **Rate Limiting** to prevent brute force attacks
- **Work Schedule Validation**

```typescript
import crypto from "crypto";

const ALGORITHM = "aes-256-cbc";

// ==================== DEVICE FINGERPRINT ENCRYPTION ====================

/**
 * Encrypt Device Fingerprint using AES-256-CBC
 */
export function encryptDeviceFingerprint(fingerprint: string): string {
  const iv = crypto.randomBytes(16);
  const key = crypto.createHash('sha256').update(ENCRYPTION_KEY).digest();
  
  const cipher = crypto.createCipheriv(ALGORITHM, key, iv);
  let encrypted = cipher.update(fingerprint, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  return iv.toString('hex') + ':' + encrypted;
}

/**
 * Decrypt Device Fingerprint
 */
export function decryptDeviceFingerprint(encryptedData: string): string {
  const parts = encryptedData.split(':');
  const iv = Buffer.from(parts[0], 'hex');
  const encryptedText = parts[1];
  
  const key = crypto.createHash('sha256').update(ENCRYPTION_KEY).digest();
  
  const decipher = crypto.createDecipheriv(ALGORITHM, key, iv);
  let decrypted = decipher.update(encryptedText, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}

/**
 * Verify Device Fingerprint
 */
export function verifyDeviceFingerprint(
  providedFingerprint: string,
  storedEncryptedFingerprint: string
): boolean {
  try {
    const decrypted = decryptDeviceFingerprint(storedEncryptedFingerprint);
    return decrypted === providedFingerprint;
  } catch {
    return false;
  }
}

// ==================== GPS LOCATION VALIDATION ====================

/**
 * Calculate distance between two GPS points using Haversine formula
 * @returns Distance in meters
 */
export function calculateDistance(
  lat1: number, lon1: number,
  lat2: number, lon2: number
): number {
  const R = 6371e3; // Earth's radius in meters
  const φ1 = (lat1 * Math.PI) / 180;
  const φ2 = (lat2 * Math.PI) / 180;
  const Δφ = ((lat2 - lat1) * Math.PI) / 180;
  const Δλ = ((lon2 - lon1) * Math.PI) / 180;

  const a =
    Math.sin(Δφ / 2) * Math.sin(Δφ / 2) +
    Math.cos(φ1) * Math.cos(φ2) * Math.sin(Δλ / 2) * Math.sin(Δλ / 2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return R * c;
}

/**
 * Check if user location is within allowed radius
 */
export function isLocationWithinRadius(
  userLat: number, userLon: number,
  allowedLat: number, allowedLon: number,
  radiusMeters: number
): boolean {
  const distance = calculateDistance(userLat, userLon, allowedLat, allowedLon);
  return distance <= radiusMeters;
}

// ==================== WORK SCHEDULE VALIDATION ====================

/**
 * Check if current time is within work schedule
 */
export function isWithinWorkSchedule(
  currentTime: Date,
  startTime: string, // "HH:MM"
  endTime: string,   // "HH:MM"
  daysOfWeek: string[] // ["0", "1", "2", "3", "4"]
): boolean {
  // Check day
  const currentDay = currentTime.getDay().toString();
  if (!daysOfWeek.includes(currentDay)) {
    return false;
  }
  
  // Check time
  const currentMinutes = currentTime.getHours() * 60 + currentTime.getMinutes();
  const [startHour, startMin] = startTime.split(':').map(Number);
  const [endHour, endMin] = endTime.split(':').map(Number);
  
  const startMinutes = startHour * 60 + startMin;
  const endMinutes = endHour * 60 + endMin;
  
  return currentMinutes >= startMinutes && currentMinutes <= endMinutes;
}

// ==================== RATE LIMITING ====================

const rateLimitStore = new Map<string, RateLimitEntry>();

/**
 * Check Rate Limit
 * @param key - Unique key (userId, IP, deviceId)
 * @param maxAttempts - Maximum attempts allowed
 * @param windowMs - Time window in milliseconds
 * @param blockDurationMs - Block duration after exceeding limit
 */
export function checkRateLimit(
  key: string,
  maxAttempts: number = 3,
  windowMs: number = 60000,
  blockDurationMs: number = 300000
): { allowed: boolean; remainingAttempts: number; blockedUntil?: number } {
  const now = Date.now();
  const entry = rateLimitStore.get(key);
  
  // If blocked
  if (entry?.blockedUntil && entry.blockedUntil > now) {
    return { allowed: false, remainingAttempts: 0, blockedUntil: entry.blockedUntil };
  }
  
  // Reset window if expired
  if (!entry || now - entry.firstAttempt > windowMs) {
    rateLimitStore.set(key, { count: 1, firstAttempt: now });
    return { allowed: true, remainingAttempts: maxAttempts - 1 };
  }
  
  // Increment counter
  entry.count++;
  
  // If exceeded limit
  if (entry.count > maxAttempts) {
    entry.blockedUntil = now + blockDurationMs;
    rateLimitStore.set(key, entry);
    return { allowed: false, remainingAttempts: 0, blockedUntil: entry.blockedUntil };
  }
  
  return { allowed: true, remainingAttempts: maxAttempts - entry.count };
}
```

---

## 5. Attendance Security Middleware

**File:** `server/middleware/attendance-security.ts`

This middleware validates all attendance attempts with 4 layers of security.

### Validation Layers:
1. **Rate Limiting** - Prevent brute force
2. **Device Verification** - Match encrypted fingerprint
3. **GPS Location** - Check if within allowed radius
4. **Work Schedule** - Check if within working hours (disabled)

```typescript
import type { Request, Response, NextFunction } from "express";
import { verifyDeviceFingerprint, isLocationWithinRadius, checkRateLimit } from "../utils/security";

export async function validateAttendance(req: Request, res: Response, next: NextFunction) {
  const userId = req.session?.userId;
  const { deviceFingerprint, latitude, longitude } = req.body;

  // 1. Rate Limiting
  const rateLimitKey = `attendance:${userId}`;
  const rateLimit = checkRateLimit(rateLimitKey, 3, 60000, 300000);
  
  if (!rateLimit.allowed) {
    await logAttendanceAttempt({ userId, status: "failed_rate_limit", ... });
    return res.status(429).json({ error: "Too many attempts. Please try again later." });
  }

  // 2. Device Verification
  const deviceValidation = await validateDevice(userId, deviceFingerprint);
  if (!deviceValidation.success) {
    await logAttendanceAttempt({ userId, status: "failed_device", ... });
    return res.status(403).json({ error: deviceValidation.error, code: "device_mismatch" });
  }

  // 3. GPS Location Validation (Required)
  if (!latitude || !longitude) {
    return res.status(400).json({ error: "Location data is required", code: "missing_data" });
  }

  const locationValidation = await validateLocation(userId, parseFloat(latitude), parseFloat(longitude));
  if (!locationValidation.success) {
    await logAttendanceAttempt({ userId, status: "failed_location", ... });
    return res.status(403).json({ error: locationValidation.error, code: "location_not_allowed" });
  }

  // 4. Work Schedule Validation (Currently Disabled - 24/7 operation)
  // Uncomment to enable work schedule validation
  /*
  const scheduleValidation = await validateWorkSchedule(userId);
  if (!scheduleValidation.success) {
    return res.status(403).json({ error: scheduleValidation.error });
  }
  */

  // All validations passed
  await logAttendanceAttempt({ userId, status: "success", ... });
  resetRateLimit(rateLimitKey);

  req.validatedAttendanceData = { deviceFingerprint, latitude, longitude };
  next();
}

async function validateDevice(userId: string, providedFingerprint: string): Promise<ValidationResult> {
  const user = await db.query.users.findFirst({ where: eq(users.id, userId) });
  
  if (!user?.deviceFingerprint) {
    return { success: false, error: "No device registered" };
  }

  const isValid = verifyDeviceFingerprint(providedFingerprint, user.deviceFingerprint);
  if (!isValid) {
    return { success: false, error: "Device mismatch" };
  }

  return { success: true };
}

async function validateLocation(userId: string, userLat: number, userLon: number): Promise<ValidationResult> {
  const user = await db.query.users.findFirst({ where: eq(users.id, userId) });

  const allowedLocs = await db.query.allowedLocations.findMany({
    where: and(eq(allowedLocations.active, true), eq(allowedLocations.departmentId, user?.departmentId))
  });

  for (const location of allowedLocs) {
    if (isLocationWithinRadius(userLat, userLon, parseFloat(location.latitude), parseFloat(location.longitude), parseFloat(location.radiusMeters))) {
      return { success: true };
    }
  }

  return { success: false, error: "You are not within any allowed location" };
}
```

---

## 6. Frontend Router

**File:** `client/src/App.tsx`

This file defines all frontend routes using Wouter.

### Routes:

| Path | Component | Description |
|------|-----------|-------------|
| `/` | Home | Landing page |
| `/login` | LoginPage | Unified login |
| `/dashboard` | Dashboard | Supervisor/Manager dashboard |
| `/attendance/employee` | EmployeeDashboard | Trainee attendance |
| `/attendance/supervisor` | SupervisorDashboard | Supervisor view |
| `/hrm/employees` | EmployeeListPage | Employee management |
| `/departments` | DepartmentListPage | Department management |
| `/settings/allowed-locations` | AllowedLocationsPage | GPS settings |
| `/settings/work-schedules` | WorkSchedulesPage | Schedule settings |

```typescript
import { Switch, Route } from "wouter";
import { QueryClientProvider } from "@tanstack/react-query";

// Pages
import Home from "@/pages/home";
import Dashboard from "@/pages/dashboard";
import LoginPage from "@/pages/login";
import EmployeeDashboard from "@/pages/attendance/employee-dashboard";
import SupervisorDashboard from "@/pages/attendance/supervisor-dashboard";
import EmployeeListPage from "@/pages/hrm/employee-list";
import DepartmentListPage from "@/pages/departments/department-list";
import AllowedLocationsPage from "@/pages/settings/allowed-locations";
import WorkSchedulesPage from "@/pages/settings/work-schedules";

// Layouts
import { AttendanceLayout } from "@/components/layout/attendance-layout";
import { HRMLayout } from "@/components/layout/hrm-layout";

function Router() {
  return (
    <Switch>
      <Route path="/" component={Home} />
      <Route path="/login" component={LoginPage} />
      <Route path="/dashboard" component={Dashboard} />
      
      {/* Attendance Routes */}
      <Route path="/attendance/employee">
        {() => <AttendanceLayout><EmployeeDashboard /></AttendanceLayout>}
      </Route>
      <Route path="/attendance/supervisor">
        {() => <AttendanceLayout><SupervisorDashboard /></AttendanceLayout>}
      </Route>
      
      {/* HRM Routes */}
      <Route path="/hrm/employees">
        {() => <HRMLayout><EmployeeListPage /></HRMLayout>}
      </Route>
      
      {/* Department Routes */}
      <Route path="/departments">
        {() => <HRMLayout><DepartmentListPage /></HRMLayout>}
      </Route>
      
      {/* Settings Routes */}
      <Route path="/settings/allowed-locations">
        {() => <AttendanceLayout><AllowedLocationsPage /></AttendanceLayout>}
      </Route>
      <Route path="/settings/work-schedules">
        {() => <AttendanceLayout><WorkSchedulesPage /></AttendanceLayout>}
      </Route>
      
      <Route component={NotFound} />
    </Switch>
  );
}

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TooltipProvider>
        <Toaster />
        <Router />
      </TooltipProvider>
    </QueryClientProvider>
  );
}

export default App;
```

---

## Summary

| File | Purpose | Lines |
|------|---------|-------|
| `shared/schema.ts` | Database tables & types | ~330 |
| `server/routes.ts` | API endpoints | ~1100 |
| `server/storage.ts` | Database operations | ~450 |
| `server/utils/security.ts` | Security functions | ~245 |
| `server/middleware/attendance-security.ts` | Attendance validation | ~370 |
| `client/src/App.tsx` | Frontend routing | ~195 |

---

## Technologies Used

| Category | Technology |
|----------|------------|
| **Frontend** | React 18, TypeScript, TailwindCSS, shadcn/ui |
| **Backend** | Express.js, TypeScript |
| **Database** | PostgreSQL, Drizzle ORM |
| **Security** | AES-256-CBC, bcrypt, FingerprintJS |
| **Authentication** | Session-based (express-session) |

---

**End of Document**

*Prepared by: Sara Portal Development Team*  
*Date: December 2024*
