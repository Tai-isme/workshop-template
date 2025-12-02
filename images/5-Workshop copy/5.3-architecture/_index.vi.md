---
title : "Kiến trúc"
date :  "2025-09-15" 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Kiến Trúc Tổng Quan

Hệ thống Excel Import được thiết kế theo mô hình **Serverless Event-Driven Architecture**, tận dụng các managed services của AWS để giảm thiểu operational overhead và tối ưu chi phí.

![Architecture Diagram](/images/6-Excel-Workshop/architecture-diagram.png)

#### 🏗️ Các Tầng Kiến Trúc

#### **1. Frontend Layer (React Application)**

**Công nghệ:**
- React 18 với Vite build tool
- AWS SDK for JavaScript (Cognito integration)
- Axios cho API calls

**Chức năng:**
```
┌─────────────────────────────────────┐
│  React Frontend (Port 5173)         │
├─────────────────────────────────────┤
│  • LoginPage: Xác thực user         │
│  • RegisterPage: Đăng ký mới        │
│  • VerifyPage: Confirm email OTP    │
│  • ImportPage: Upload Excel + View  │
│  • CallbackPage: OAuth redirect     │
└─────────────────────────────────────┘
         │
         │ HTTPS (CORS enabled)
         ▼
```

**Authentication Flow:**
- User register → Cognito gửi OTP qua email
- User confirm → Activate account
- User login → Nhận JWT tokens (IdToken, AccessToken, RefreshToken)
- Mỗi API request gửi kèm IdToken trong Authorization header

---

#### **2. API Layer (API Gateway + Lambda)**

**API Gateway Configuration:**
```yaml
Type: REST API
Stage: dev (hoặc prod)
CORS: Enabled cho origin: http://localhost:5173
Authorizer: Cognito User Pool
```

**Endpoints:**

| Method | Path | Lambda Function | Auth Required | Description |
|--------|------|-----------------|---------------|-------------|
| POST | /register | RegisterFunction | ❌ | Đăng ký user mới |
| POST | /confirm | ConfirmFunction | ❌ | Xác nhận email OTP |
| POST | /login | LoginFunction | ❌ | Đăng nhập |
| POST | /logout | LogoutFunction | ✅ | Đăng xuất |
| POST | /upload-url | GenerateUploadUrlFunction | ✅ | Tạo pre-signed URL |
| GET | /import/jobs | ListImportJobsFunction | ✅ | List tất cả jobs |
| GET | /jobs/{jobId} | GetJobStatusFunction | ✅ | Xem chi tiết job |

**Request/Response Flow:**
```
Client → API Gateway → Cognito Authorizer (verify JWT)
                    ↓
                  Lambda → DynamoDB/Cognito/S3
                    ↓
                  Response ← JSON
```

---

#### **3. Processing Layer (Lambda Functions)**

**Lambda Functions Details:**

**A. Authentication Functions (4 functions):**

```
RegisterFunction
├── Input: { email, password }
├── Process: Cognito.signUp()
├── Output: { userId, status: "PENDING_CONFIRMATION" }
└── No DynamoDB interaction

ConfirmFunction
├── Input: { email, confirmationCode }
├── Process: Cognito.confirmSignUp()
├── Output: { status: "CONFIRMED" }
└── No DynamoDB interaction

LoginFunction
├── Input: { email, password }
├── Process: Cognito.initiateAuth()
├── Output: { tokens: { IdToken, AccessToken, RefreshToken } }
└── No DynamoDB interaction

LogoutFunction
├── Input: { accessToken }
├── Process: Cognito.globalSignOut()
├── Output: { message: "Logged out successfully" }
└── No DynamoDB interaction
```

**B. Import Orchestration Functions (3 functions):**

```
GenerateUploadUrlFunction (Critical)
├── Input: { fileName, fileSize }
├── Process:
│   1. Create ImportJob record trong DynamoDB
│   2. Generate S3 pre-signed URL (PUT)
│   3. Set expiration: 15 minutes
├── Output: { jobId, uploadUrl, expiresIn }
└── Tables: ImportJobsTable (write)

ListImportJobsFunction
├── Input: query params (limit, lastKey)
├── Process: DynamoDB.scan() hoặc query()
├── Output: { jobs: [...], nextToken }
└── Tables: ImportJobsTable (read)

GetJobStatusFunction
├── Input: { jobId }
├── Process: DynamoDB.getItem()
├── Output: { job details, status, stats }
└── Tables: ImportJobsTable (read)
```

**C. Core Processing Function (1 function - quan trọng nhất):**

```
ImportS3TriggerFunction
├── Trigger: S3 Event (ObjectCreated:*)
├── Timeout: 15 minutes (900 seconds)
├── Memory: 2048 MB
├── Process Flow:
│   1. Get file từ S3 (event.Records[0].s3)
│   2. Parse Excel bằng Apache POI
│   3. Validate từng row (email format, required fields)
│   4. Batch write vào DynamoDB (25 items/batch)
│   5. Update ImportJob status & statistics
├── Output: Updated ImportJob record
└── Tables: StudentsTable, CoursesTable, ImportJobsTable
```

**Environment Variables cho tất cả Lambdas:**
```yaml
STUDENTS_TABLE: workshop-students
COURSES_TABLE: workshop-courses
IMPORT_JOBS_TABLE: workshop-import-jobs
IMPORT_BUCKET: workshop-excel-imports-{AccountId}
USER_POOL_ID: us-east-1_xxxxxxxxx
USER_POOL_CLIENT_ID: xxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

#### **4. Storage Layer**

**A. Amazon S3 (File Storage):**

```
Bucket: workshop-excel-imports-{AccountId}
├── Prefix: imports/
│   ├── {jobId}-{fileName}.xlsx
│   └── Auto-triggered Lambda on upload
├── CORS: Configured for direct upload
├── Lifecycle Policy: Delete after 7 days
└── Event Notifications:
    └── ObjectCreated:* → ImportS3TriggerFunction
```

**S3 Event Configuration:**
```yaml
Events:
  - s3:ObjectCreated:*
Filter:
  Prefix: imports/
Target:
  - ImportS3TriggerFunction (Lambda ARN)
```

---

**B. Amazon DynamoDB (NoSQL Database):**

**Table 1: workshop-students**
```
Partition Key: studentId (String, UUID)
Attributes:
  - studentCode (String, unique, GSI)
  - fullName (String)
  - email (String, GSI)
  - dateOfBirth (String, ISO format)
  - phoneNumber (String, optional)
  - address (String, optional)
  - createdAt (Number, timestamp)

Global Secondary Indexes:
  1. email-index: email (PK)
  2. studentCode-index: studentCode (PK)

Capacity Mode: PAY_PER_REQUEST (On-Demand)
```

**Table 2: workshop-courses**
```
Partition Key: courseId (String, UUID)
Attributes:
  - courseCode (String, unique, GSI)
  - courseName (String)
  - credits (Number)
  - instructor (String, optional)
  - semester (String, optional)
  - createdAt (Number, timestamp)

Global Secondary Indexes:
  1. courseCode-index: courseCode (PK)

Capacity Mode: PAY_PER_REQUEST (On-Demand)
```

**Table 3: workshop-import-jobs**
```
Partition Key: jobId (String, UUID)
Attributes:
  - userId (String, from Cognito)
  - fileName (String)
  - fileSize (Number, bytes)
  - s3Key (String)
  - status (String: PENDING|PROCESSING|COMPLETED|FAILED)
  - totalRecords (Number)
  - processedRecords (Number)
  - failedRecords (Number)
  - errorMessage (String, optional)
  - createdAt (Number, timestamp)
  - updatedAt (Number, timestamp)
  - completedAt (Number, timestamp, optional)

Capacity Mode: PAY_PER_REQUEST (On-Demand)
```

---

**C. Amazon Cognito (User Management):**

```
User Pool: ExcelWorkshopUsers
├── Authentication: Email + Password
├── Password Policy:
│   ├── Min length: 8
│   ├── Require uppercase: Yes
│   ├── Require lowercase: Yes
│   ├── Require numbers: Yes
│   └── Require symbols: No
├── Auto-verify: Email
├── Email config: Cognito default (limit 50 emails/day)
└── User Pool Client:
    ├── Auth flows: USER_PASSWORD_AUTH, USER_SRP_AUTH
    ├── Token validity: 1 hour (Access/Id), 30 days (Refresh)
    └── No client secret (public client)
```

---

#### 🔄 Luồng Hoạt Động Chi Tiết

#### **Luồng 1: User Registration & Login**

```
1. User nhập email + password trên RegisterPage
   ↓
2. Frontend gọi POST /register
   ↓
3. RegisterFunction → Cognito.signUp()
   ↓
4. Cognito gửi OTP code qua email
   ↓
5. User nhập code trên VerifyPage
   ↓
6. Frontend gọi POST /confirm
   ↓
7. ConfirmFunction → Cognito.confirmSignUp()
   ↓
8. Account activated ✅
   ↓
9. User nhập email + password trên LoginPage
   ↓
10. Frontend gọi POST /login
    ↓
11. LoginFunction → Cognito.initiateAuth()
    ↓
12. Return JWT tokens (lưu localStorage)
    ↓
13. Redirect to ImportPage
```

---

#### **Luồng 2: Excel File Upload (Critical Flow)**

```
1. User chọn file .xlsx trên ImportPage
   ↓
2. Frontend validate file (size, extension)
   ↓
3. Frontend gọi POST /upload-url
   Headers: { Authorization: Bearer {IdToken} }
   Body: { fileName: "students.xlsx", fileSize: 45678 }
   ↓
4. API Gateway verify JWT với Cognito
   ↓
5. GenerateUploadUrlFunction:
   a. Tạo jobId (UUID)
   b. Put item vào ImportJobsTable:
      {
        jobId, userId, fileName, fileSize,
        status: "PENDING",
        s3Key: "imports/{jobId}-{fileName}",
        createdAt: now()
      }
   c. S3.getSignedUrl("putObject") với:
      - Bucket: workshop-excel-imports-{AccountId}
      - Key: imports/{jobId}-{fileName}
      - Expires: 900 seconds (15 min)
      - ContentType: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
   ↓
6. Return { jobId, uploadUrl, expiresIn: 900 }
   ↓
7. Frontend upload file trực tiếp lên S3:
   PUT {uploadUrl}
   Body: file binary
   ↓
8. S3 lưu file thành công
   ↓
9. S3 Event Notification trigger ImportS3TriggerFunction
```

---

#### **Luồng 3: Excel Processing (Core Logic)**

```
ImportS3TriggerFunction triggered:
├── Input: S3 Event JSON
│   {
│     "Records": [{
│       "s3": {
│         "bucket": { "name": "workshop-excel-imports-123456" },
│         "object": { "key": "imports/uuid-students.xlsx" }
│       }
│     }]
│   }
│
├── Step 1: Extract thông tin
│   bucketName = event.Records[0].s3.bucket.name
│   objectKey = event.Records[0].s3.object.key
│   jobId = objectKey.split('/')[1].split('-')[0]
│
├── Step 2: Update job status → PROCESSING
│   DynamoDB.updateItem(ImportJobsTable, {
│     jobId: jobId,
│     status: "PROCESSING",
│     updatedAt: now()
│   })
│
├── Step 3: Download file từ S3
│   file = S3.getObject({ Bucket: bucketName, Key: objectKey })
│   inputStream = file.Body
│
├── Step 4: Parse Excel với Apache POI
│   Workbook workbook = WorkbookFactory.create(inputStream)
│   Sheet sheet = workbook.getSheetAt(0)
│   
│   Header row (row 0): studentCode | fullName | email | courseCode
│   Data rows: row 1 đến row N
│
├── Step 5: Process từng row
│   List<Student> students = []
│   List<Course> courses = []
│   int totalRecords = 0
│   int failedRecords = 0
│   List<String> errors = []
│   
│   for (Row row : sheet) {
│     if (row.getRowNum() == 0) continue; // Skip header
│     
│     try {
│       // Extract data
│       String studentCode = getCellValue(row, 0)
│       String fullName = getCellValue(row, 1)
│       String email = getCellValue(row, 2)
│       String courseCode = getCellValue(row, 3)
│       
│       // Validate
│       if (!isValidEmail(email)) {
│         throw Exception("Invalid email format")
│       }
│       
│       // Create entities
│       Student student = new Student()
│       student.setStudentId(UUID.randomUUID())
│       student.setStudentCode(studentCode)
│       student.setFullName(fullName)
│       student.setEmail(email)
│       
│       Course course = new Course()
│       course.setCourseId(UUID.randomUUID())
│       course.setCourseCode(courseCode)
│       
│       students.add(student)
│       courses.add(course)
│       totalRecords++
│       
│     } catch (Exception e) {
│       failedRecords++
│       errors.add("Row " + row.getRowNum() + ": " + e.getMessage())
│     }
│   }
│
├── Step 6: Batch write to DynamoDB
│   // Batch write students (25 items per batch)
│   for (batch : students.chunk(25)) {
│     DynamoDB.batchWriteItem(StudentsTable, batch)
│   }
│   
│   // Batch write courses (25 items per batch)
│   for (batch : courses.chunk(25)) {
│     DynamoDB.batchWriteItem(CoursesTable, batch)
│   }
│
├── Step 7: Update job status → COMPLETED hoặc FAILED
│   DynamoDB.updateItem(ImportJobsTable, {
│     jobId: jobId,
│     status: failedRecords == totalRecords ? "FAILED" : "COMPLETED",
│     totalRecords: totalRecords,
│     processedRecords: totalRecords - failedRecords,
│     failedRecords: failedRecords,
│     errorMessage: errors.join("; "),
│     completedAt: now(),
│     updatedAt: now()
│   })
│
└── Step 8: Return success
    Return { statusCode: 200, body: "Processing completed" }
```

---

#### **Luồng 4: Track Import Job Status**

```
1. User trên ImportPage thấy job list
   ↓
2. Frontend polling GET /import/jobs mỗi 5 giây
   ↓
3. ListImportJobsFunction query ImportJobsTable
   ↓
4. Return danh sách jobs với status
   ↓
5. Frontend render:
   - PENDING: 🔵 (xanh dương, chờ xử lý)
   - PROCESSING: 🟡 (vàng, đang xử lý)
   - COMPLETED: 🟢 (xanh lá, thành công)
   - FAILED: 🔴 (đỏ, thất bại)
   ↓
6. User click vào job để xem chi tiết
   ↓
7. Frontend gọi GET /jobs/{jobId}
   ↓
8. GetJobStatusFunction query ImportJobsTable
   ↓
9. Return full job details + statistics
   ↓
10. Frontend hiển thị:
    - File name, size
    - Status, timestamps
    - Total/Processed/Failed records
    - Error messages (nếu có)
```

---

#### 🔐 Security Best Practices

**1. Authentication & Authorization:**
- ✅ JWT tokens với short expiration (1 hour)
- ✅ Cognito Authorizer protect sensitive endpoints
- ✅ Refresh token rotation

**2. Data Protection:**
- ✅ HTTPS only (API Gateway enforced)
- ✅ Pre-signed URLs với expiration ngắn (15 min)
- ✅ S3 bucket không public

**3. Access Control:**
- ✅ Lambda execution roles với least privilege
- ✅ DynamoDB fine-grained access control
- ✅ VPC endpoints (optional, không dùng trong workshop này)

**4. Input Validation:**
- ✅ Frontend validate file size/type
- ✅ Lambda validate email format, required fields
- ✅ DynamoDB schema validation

---

#### 📊 Scalability & Performance

**Auto-scaling:**
- Lambda: Auto-scale đến 1000 concurrent executions (default)
- DynamoDB: On-Demand mode tự động scale
- API Gateway: Handle millions requests/second

**Performance Optimization:**
- Batch write DynamoDB (25 items/request)
- Stream processing cho large files (nếu cần)
- CloudWatch metrics cho monitoring

**Cost Optimization:**
- S3 lifecycle policy: auto-delete sau 7 ngày
- Lambda timeout: 15 min (chỉ charge khi chạy)
- DynamoDB on-demand: chỉ trả theo usage thực tế

---

#### 🎯 Key Takeaways

1. **Event-Driven Architecture:** S3 Events trigger processing tự động
2. **Serverless Benefits:** No server management, auto-scaling, pay-per-use
3. **Managed Services:** Giảm operational overhead
4. **Security First:** Cognito + JWT + IAM roles
5. **Cost-Effective:** Free Tier coverage cho workshop này

#### 🚀 Tiếp Theo

Đã hiểu kiến trúc? Bắt đầu deploy backend!

[➡️ Tiếp theo: Deploy Backend](../6.4-deploy-backend/)
