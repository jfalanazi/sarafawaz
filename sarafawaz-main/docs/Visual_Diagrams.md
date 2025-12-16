# Visual Diagrams - Sara Portal
# Attendance and HRM Management System

**Version:** 1.0  
**Date:** December 2024

---

## 1. Use Case Diagram

### Generated Image

![Use Case Diagram](../attached_assets/generated_images/use_case_diagram_sara_system.png)

### PlantUML Code

```plantuml
@startuml Sara Portal Use Case Diagram

left to right direction
skinparam packageStyle rectangle
skinparam actorStyle awesome

actor "Trainee" as Trainee
actor "Supervisor" as Supervisor
actor "Manager" as Manager

Supervisor --|> Trainee
Manager --|> Supervisor

rectangle "Sara Attendance System" {
    
    usecase "UC-01: Login" as UC01
    usecase "UC-02: Logout" as UC02
    usecase "UC-03: Check-In" as UC03
    usecase "UC-04: Check-Out" as UC04
    usecase "UC-05: View Personal Records" as UC05
    usecase "UC-06: Register Device" as UC06
    usecase "UC-07: Request Device Change" as UC07
    usecase "UC-08: View Statistics" as UC08
    
    usecase "UC-10: Manage Department Locations" as UC10
    usecase "UC-11: Manage Department Schedules" as UC11
    usecase "UC-12: Approve Device Requests" as UC12
    usecase "UC-13: View Department Reports" as UC13
    usecase "UC-14: Manage Department Trainees" as UC14
    
    usecase "UC-20: Manage Departments" as UC20
    usecase "UC-21: Manage Users" as UC21
    usecase "UC-22: Manage Global Locations" as UC22
    usecase "UC-23: System Settings" as UC23
    usecase "UC-24: View All Reports" as UC24
    
    usecase "Verify GPS Location" as VerifyGPS
    usecase "Verify Device Fingerprint" as VerifyDevice
    usecase "Encrypt Data" as EncryptData
    usecase "Export to PDF/Excel" as ExportReport
    
    Trainee --> UC01
    Trainee --> UC02
    Trainee --> UC03
    Trainee --> UC04
    Trainee --> UC05
    Trainee --> UC06
    Trainee --> UC07
    Trainee --> UC08
    
    Supervisor --> UC10
    Supervisor --> UC11
    Supervisor --> UC12
    Supervisor --> UC13
    Supervisor --> UC14
    
    Manager --> UC20
    Manager --> UC21
    Manager --> UC22
    Manager --> UC23
    Manager --> UC24
    
    UC03 ..> VerifyGPS : <<include>>
    UC03 ..> VerifyDevice : <<include>>
    UC04 ..> VerifyGPS : <<include>>
    UC04 ..> VerifyDevice : <<include>>
    UC06 ..> EncryptData : <<include>>
    VerifyDevice ..> EncryptData : <<include>>
    UC13 ..> ExportReport : <<include>>
    UC24 ..> ExportReport : <<include>>
}

@enduml
```

### How to Render
1. Copy the code above
2. Go to: https://www.plantuml.com/plantuml/
3. Paste the code and click "Submit"
4. Download the generated image

---

## 2. Class Diagram

### Generated Image

![Class Diagram](../attached_assets/generated_images/class_diagram_system_structure.png)

### Mermaid Code

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
        +checkIn(lat, lon, fingerprint) AttendanceRecord
        +checkOut(lat, lon, fingerprint) AttendanceRecord
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
    
    class AttendanceRecord {
        +String id
        +String userId
        +String groupId
        +AttendanceType type
        +Date timestamp
        +String deviceFingerprint
        +String latitude
        +String longitude
        +Boolean manualEntry
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
        +Date createdAt
        +isWorkingTime(datetime) Boolean
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
    }
    
    class ReportService {
        <<service>>
        +generateDailyReport(date, deptId) Report
        +generateMonthlyReport(year, month, deptId) Report
        +exportToPDF(data) File
        +exportToExcel(data) File
    }
    
    class IStorage {
        <<interface>>
        +getUser(id) User
        +getUserByUsername(username) User
        +createUser(data) User
        +updateUser(id, data) User
        +getAttendanceRecords(userId) AttendanceRecord[]
        +createAttendanceRecord(data) AttendanceRecord
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
    
    User "1" --> "0..*" AttendanceRecord : records
    User "1" --> "0..*" DeviceChangeRequest : requests
    User "0..*" --> "0..1" Location : department
    User "1" --> "0..*" Group : supervises
    User ..> Role : has
    
    Location "1" --> "0..*" User : employees
    Location "1" --> "0..*" Group : groups
    Location "1" --> "0..*" AllowedLocation : gpsLocations
    Location "1" --> "0..*" WorkSchedule : schedules
    
    Group "0..*" --> "1" User : supervisor
    Group "0..*" --> "0..1" Location : location
    
    AllowedLocation "0..*" --> "0..1" Location : department
    WorkSchedule "0..*" --> "0..1" Location : department
    
    DeviceChangeRequest "0..*" --> "0..1" User : reviewer
    DeviceChangeRequest ..> RequestStatus : has
    
    AttendanceRecord ..> AttendanceType : has
    
    AuthService --> IStorage : uses
    AuthService --> SecurityService : uses
    ReportService --> IStorage : uses
    AllowedLocation --> GPSService : uses
```

### How to Render
1. Copy the code above (without the ```mermaid wrapper)
2. Go to: https://mermaid.live
3. Paste the code in the editor
4. Download as PNG or SVG

---

## 3. Sequence Diagram - Check-In Process

### Generated Image

![Sequence Diagram - Check-In](../attached_assets/generated_images/sequence_diagram_check-in_process.png)

### Mermaid Code (Fixed - No activate/deactivate)

```mermaid
sequenceDiagram
    autonumber
    participant T as Trainee
    participant UI as User Interface
    participant API as API Server
    participant SEC as Security Middleware
    participant DB as Database
    participant GPS as GPS Service
    
    T->>UI: 1. Click Check-In Button
    UI->>GPS: 2. Request Current Location
    GPS-->>UI: 3. Return Coordinates (lat, lon)
    
    UI->>API: 4. POST /api/attendance/check-in
    Note over UI,API: Body: deviceFingerprint, latitude, longitude
    
    API->>SEC: 5. Validate Session
    SEC-->>API: 6. Session Valid
    
    API->>SEC: 7. Verify Device Fingerprint
    SEC->>DB: 8. Query user.deviceFingerprint
    DB-->>SEC: 9. Return Stored Fingerprint
    SEC->>SEC: 10. Decrypt and Compare
    
    alt Device Not Matched
        SEC-->>API: 11a. Failure: Device Not Registered
        API->>DB: 12a. Log Failed Attempt
        DB-->>API: 13a. Logged
        API-->>UI: 14a. HTTP 403 Error
        UI-->>T: 15a. Display Error Message
    else Device Matched
        SEC-->>API: 11b. Success: Device Valid
        
        API->>SEC: 12b. Verify GPS Location
        SEC->>DB: 13b. Get allowed_locations
        DB-->>SEC: 14b. Return Location List
        SEC->>SEC: 15b. Calculate Distance (Haversine)
        
        alt Location Out of Range
            SEC-->>API: 16a. Failure: Outside Area
            API->>DB: 17a. Log Failed Attempt
            DB-->>API: 18a. Logged
            API-->>UI: 19a. HTTP 403 Error
            UI-->>T: 20a. Display Error Message
        else Location Valid
            SEC-->>API: 16b. Success: Location Valid
            API->>DB: 17b. Check existing check-in
            DB-->>API: 18b. No existing record
            API->>DB: 19b. Create attendance record
            DB-->>API: 20b. Record Created
            API->>DB: 21b. Log success attempt
            DB-->>API: 22b. Logged
            API-->>UI: 23b. HTTP 200 Success
            UI-->>T: 24b. Display Success Message
        end
    end
```

### How to Render
1. Copy the code above (without the ```mermaid wrapper)
2. Go to: https://mermaid.live
3. Paste the code in the editor
4. Download as PNG or SVG

---

## 4. Sequence Diagram - Login Process

### Mermaid Code

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant UI as User Interface
    participant API as API Server
    participant AUTH as Auth Service
    participant DB as Database
    participant SESSION as Session Store
    
    U->>UI: 1. Enter Username and Password
    UI->>API: 2. POST /api/auth/login
    Note over UI,API: Body: username, password
    
    API->>DB: 3. SELECT user WHERE username=input
    DB-->>API: 4. Return User Record
    
    alt User Not Found
        API-->>UI: 5a. HTTP 401 Error
        UI-->>U: 6a. Display Error Message
    else User Found
        API->>AUTH: 5b. Verify Password
        AUTH->>AUTH: 6b. bcrypt.compare()
        
        alt Password Incorrect
            AUTH-->>API: 7a. Comparison Failed
            API-->>UI: 8a. HTTP 401 Error
            UI-->>U: 9a. Display Error Message
        else Password Correct
            AUTH-->>API: 7b. Comparison Successful
            API->>SESSION: 8b. Create Session
            SESSION-->>API: 9b. Return Session ID
            API-->>UI: 10b. HTTP 200 Success
            
            alt Role is employee
                UI->>UI: 11. Navigate to /trainee/dashboard
            else Role is supervisor
                UI->>UI: 11. Navigate to /supervisor/dashboard
            else Role is manager
                UI->>UI: 11. Navigate to /manager/dashboard
            end
            
            UI-->>U: 12. Display Dashboard
        end
    end
```

---

## 5. Sequence Diagram - Device Registration

### Mermaid Code

```mermaid
sequenceDiagram
    autonumber
    participant T as Trainee
    participant UI as User Interface
    participant FP as FingerprintJS
    participant API as API Server
    participant ENC as Encryption Service
    participant DB as Database
    
    T->>UI: 1. Navigate to Device Registration
    UI->>FP: 2. Initialize and Get Fingerprint
    FP->>FP: 3. Collect Device Info
    FP-->>UI: 4. Return deviceFingerprint
    
    UI->>API: 5. POST /api/users/register-device
    Note over UI,API: Body: deviceFingerprint
    
    API->>DB: 6. Check existing fingerprint
    DB-->>API: 7. Return current value
    
    alt Device Already Registered
        API-->>UI: 8a. HTTP 400 Error
        UI-->>T: 9a. Display: Request Change Instead
    else No Device Registered
        API->>ENC: 8b. Encrypt Fingerprint
        ENC->>ENC: 9b. Generate IV + AES-256-CBC
        ENC-->>API: 10b. Return Encrypted
        API->>DB: 11b. UPDATE user fingerprint
        DB-->>API: 12b. Update Successful
        API-->>UI: 13b. HTTP 200 Success
        UI-->>T: 14b. Display Success Message
    end
```

---

## 6. Sequence Diagram - Device Change Request

### Mermaid Code

```mermaid
sequenceDiagram
    autonumber
    participant T as Trainee
    participant UI as User Interface
    participant API as API Server
    participant DB as Database
    participant S as Supervisor
    
    T->>UI: 1. Fill Device Change Form
    UI->>API: 2. POST /api/device-requests
    Note over UI,API: Body: newFingerprint, reason
    
    API->>DB: 3. Create request (status: pending)
    DB-->>API: 4. Request Created
    API-->>UI: 5. HTTP 201 Created
    UI-->>T: 6. Display: Request Pending
    
    Note over T,S: Later - Supervisor Reviews
    
    S->>UI: 7. Open Pending Requests
    UI->>API: 8. GET /api/device-requests
    API->>DB: 9. Fetch pending requests
    DB-->>API: 10. Return list
    API-->>UI: 11. Return requests
    UI-->>S: 12. Display Requests
    
    S->>UI: 13. Select and Decide
    
    alt Approve
        UI->>API: 14a. PATCH status=approved
        API->>DB: 15a. Update request status
        API->>DB: 16a. Update user fingerprint
        DB-->>API: 17a. Updated
        API-->>UI: 18a. Success
        UI-->>S: 19a. Approval Confirmed
    else Reject
        UI->>API: 14b. PATCH status=rejected
        API->>DB: 15b. Update request status
        DB-->>API: 16b. Updated
        API-->>UI: 17b. Success
        UI-->>S: 18b. Rejection Confirmed
    end
```

---

## Summary

| Diagram | Type | Code Format | Image File |
|---------|------|-------------|------------|
| Use Case | UML | PlantUML | `use_case_diagram_sara_system.png` |
| Class | UML | Mermaid | `class_diagram_system_structure.png` |
| Sequence (Check-In) | UML | Mermaid | `sequence_diagram_check-in_process.png` |
| Sequence (Login) | UML | Mermaid | - |
| Sequence (Device Reg) | UML | Mermaid | - |
| Sequence (Device Change) | UML | Mermaid | - |

### Online Rendering Tools

| Tool | URL | Supports |
|------|-----|----------|
| PlantUML Online | https://www.plantuml.com/plantuml/ | PlantUML code |
| Mermaid Live Editor | https://mermaid.live | Mermaid code |

---

**End of Document**

*Prepared by: Sara Portal Development Team*  
*Date: December 2024*
