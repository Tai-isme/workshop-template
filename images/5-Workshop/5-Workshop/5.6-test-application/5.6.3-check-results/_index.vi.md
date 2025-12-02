---
title: "Kiểm Tra Kết Quả"
date: "2025-12-01"
weight: 3
chapter: false
pre: " <b> 5.6.3. </b> "
---

#### Verify Dữ Liệu Trong DynamoDB

Sau khi import job hoàn thành, hãy verify dữ liệu đã được lưu đúng vào DynamoDB.

#### Xem Job Details Trên Frontend

**Bước 1: Click "View Details"**

Từ Jobs List, click vào job vừa import:

![Job Details Button](/images/6-Excel-Workshop/job-details-button.png)

**Bước 2: Xem Job Information**

Job Details page hiển thị:

```
Job ID: abc-123-def-456-ghi-789
File Name: students.xlsx
File Size: 12.3 KB
Status: ✅ COMPLETED
Created At: 2025-12-01 10:30:15
Completed At: 2025-12-01 10:30:28
Duration: 13 seconds

Statistics:
├── Total Records: 5
├── Processed Successfully: 5
├── Failed: 0
└── Error Messages: (none)
```

![Job Details Page](/images/6-Excel-Workshop/job-details-page.png)

> ✅ Nếu "Processed Successfully = Total Records" → Import thành công hoàn toàn!

**Nếu có errors:**

```
Statistics:
├── Total Records: 10
├── Processed Successfully: 8
├── Failed: 2
└── Error Messages:
    - Row 3: Invalid email format (not-an-email)
    - Row 7: Missing required field: studentCode
```

> 💡 Error messages giúp bạn fix file và re-upload.

---

#### Verify Trong DynamoDB Console

**Check Students Table:**

**Bước 1: Mở DynamoDB Console**

- URL: https://console.aws.amazon.com/dynamodbv2
- Hoặc: AWS Console → Services → DynamoDB

**Bước 2: Navigate đến Students Table**

1. Click **"Tables"** (sidebar)
2. Click table: **workshop-students**
3. Tab **"Explore table items"**

**Bước 3: Scan table**

Click **"Scan/Query items"** → **"Run"**

Expected results (5 items nếu upload file mẫu):

| studentId (PK) | studentCode | fullName | email | createdAt |
|----------------|-------------|----------|-------|-----------|
| uuid-1 | ST001 | Nguyen Van A | nguyenvana@test.com | 1733036428000 |
| uuid-2 | ST002 | Tran Thi B | tranthib@test.com | 1733036428000 |
| uuid-3 | ST003 | Le Van C | levanc@test.com | 1733036428000 |
| uuid-4 | ST004 | Pham Thi D | phamthid@test.com | 1733036428000 |
| uuid-5 | ST005 | Hoang Van E | hoangvane@test.com | 1733036428000 |

![DynamoDB Students Data](/images/6-Excel-Workshop/dynamodb-students-data.png)

> ✅ Dữ liệu đã được import!

**Bước 4: Verify item details**

Click vào một item để xem chi tiết:

```json
{
  "studentId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "studentCode": "ST001",
  "fullName": "Nguyen Van A",
  "email": "nguyenvana@test.com",
  "createdAt": 1733036428000
}
```

---

**Check Courses Table:**

Repeat tương tự với table **workshop-courses**:

1. Click table: **workshop-courses**
2. Tab **"Explore table items"** → Scan

Expected results (3 courses từ file mẫu):

| courseId (PK) | courseCode | courseName | createdAt |
|---------------|------------|------------|-----------|
| uuid-a | CS101 | (null) | 1733036428000 |
| uuid-b | CS102 | (null) | 1733036428000 |
| uuid-c | CS103 | (null) | 1733036428000 |

> 💡 **Note:** `courseName` có thể null vì file mẫu chỉ có `courseCode`. Lambda tự động tạo course record nếu chưa tồn tại.

![DynamoDB Courses Data](/images/6-Excel-Workshop/dynamodb-courses-data.png)

---

**Check Import Jobs Table:**

Table **workshop-import-jobs** lưu tracking information:

1. Click table: **workshop-import-jobs**
2. Scan items

Expected item:

```json
{
  "jobId": "abc-123-def-456-ghi-789",
  "userId": "cognito-user-uuid",
  "fileName": "students.xlsx",
  "fileSize": 12345,
  "s3Key": "imports/abc-123-students.xlsx",
  "status": "COMPLETED",
  "totalRecords": 5,
  "processedRecords": 5,
  "failedRecords": 0,
  "errorMessage": null,
  "createdAt": 1733036415000,
  "updatedAt": 1733036428000,
  "completedAt": 1733036428000
}
```

![DynamoDB Import Job](/images/6-Excel-Workshop/dynamodb-import-job.png)

---

#### Verify Bằng AWS CLI

**Query Students by email (GSI):**

```powershell
aws dynamodb query `
  --table-name workshop-students `
  --index-name email-index `
  --key-condition-expression "email = :email" `
  --expression-attribute-values '{":email":{"S":"nguyenvana@test.com"}}' `
  --region us-east-1
```

Expected output:
```json
{
  "Items": [
    {
      "studentId": {"S": "uuid-1"},
      "studentCode": {"S": "ST001"},
      "fullName": {"S": "Nguyen Van A"},
      "email": {"S": "nguyenvana@test.com"},
      "createdAt": {"N": "1733036428000"}
    }
  ],
  "Count": 1
}
```

---

**Query Students by studentCode (GSI):**

```powershell
aws dynamodb query `
  --table-name workshop-students `
  --index-name studentCode-index `
  --key-condition-expression "studentCode = :code" `
  --expression-attribute-values '{":code":{"S":"ST002"}}' `
  --region us-east-1
```

---

**Scan all students:**

```powershell
aws dynamodb scan --table-name workshop-students --region us-east-1
```

**Count total items:**

```powershell
aws dynamodb scan --table-name workshop-students --select COUNT --region us-east-1

# Output:
# {
#   "Count": 5,
#   "ScannedCount": 5
# }
```

---

#### Test GSI (Global Secondary Index)

**Test email-index:**

Frontend có thể search student by email:

```javascript
// Example API call (nếu implement search endpoint)
GET /students?email=nguyenvana@test.com

// Lambda query DynamoDB:
dynamoDB.query({
  TableName: 'workshop-students',
  IndexName: 'email-index',
  KeyConditionExpression: 'email = :email',
  ExpressionAttributeValues: { ':email': 'nguyenvana@test.com' }
})
```

**Test studentCode-index:**

```javascript
GET /students?studentCode=ST001

// Query với studentCode-index
```

> 💡 **Performance:** GSI query nhanh hơn scan rất nhiều (O(log n) vs O(n)).

---

#### Verify S3 Lifecycle

**Check file sẽ tự động xóa sau 7 ngày:**

```powershell
# Get lifecycle configuration
aws s3api get-bucket-lifecycle-configuration --bucket workshop-excel-imports-YOUR_ACCOUNT_ID
```

Expected output:
```json
{
  "Rules": [
    {
      "ID": "DeleteAfter7Days",
      "Status": "Enabled",
      "Expiration": {
        "Days": 7
      }
    }
  ]
}
```

> ✅ Files sẽ tự động xóa để tiết kiệm chi phí storage!

---

#### Advanced Verification

**Check Lambda metrics:**

```powershell
# Get ImportS3TriggerFunction invocation count (last 1 hour)
aws cloudwatch get-metric-statistics `
  --namespace AWS/Lambda `
  --metric-name Invocations `
  --dimensions Name=FunctionName,Value=ImportS3Trigger `
  --start-time (Get-Date).AddHours(-1).ToString("yyyy-MM-ddTHH:mm:ss") `
  --end-time (Get-Date).ToString("yyyy-MM-ddTHH:mm:ss") `
  --period 3600 `
  --statistics Sum `
  --region us-east-1
```

**Check API Gateway request count:**

```powershell
aws cloudwatch get-metric-statistics `
  --namespace AWS/ApiGateway `
  --metric-name Count `
  --dimensions Name=ApiName,Value=excel-import-workshop `
  --start-time (Get-Date).AddHours(-1).ToString("yyyy-MM-ddTHH:mm:ss") `
  --end-time (Get-Date).ToString("yyyy-MM-ddTHH:mm:ss") `
  --period 3600 `
  --statistics Sum `
  --region us-east-1
```

---

#### Compare Data: Excel vs DynamoDB

**Verify data integrity:**

1. **Row count match:**
   - Excel: 5 rows (excluding header)
   - DynamoDB: 5 items in students table ✅

2. **Data accuracy:**
   - ST001 | Nguyen Van A | nguyenvana@test.com ✅
   - ST002 | Tran Thi B | tranthib@test.com ✅
   - (verify tất cả rows)

3. **No data loss:**
   - Total records = Processed records
   - Failed records = 0

---

#### Troubleshooting

**Không thấy data trong DynamoDB**

Debug checklist:
1. ✅ Job status = COMPLETED?
2. ✅ CloudWatch Logs có "Batch writing to DynamoDB"?
3. ✅ Lambda có IAM permission DynamoDB:PutItem?
4. ✅ Table name đúng không?

Solution:
```powershell
# Check Lambda execution role
aws lambda get-function --function-name ImportS3Trigger --query 'Configuration.Role'

# Check role policies
aws iam list-attached-role-policies --role-name LAMBDA_ROLE_NAME
```

---

**Data không đúng format**

Nguyên nhân: Lambda parse sai hoặc mapping sai columns.

Debug:
1. Check CloudWatch Logs: "Processing row X: ..."
2. Verify Excel columns đúng thứ tự:
   - A: studentCode
   - B: fullName
   - C: email
   - D: courseCode

---

**Duplicate data khi re-upload**

Nguyên nhân: DynamoDB overwrite item với cùng primary key.

Expected behavior:
- Nếu studentCode đã tồn tại → overwrite (vì Lambda tạo new UUID cho studentId)
- Nếu muốn prevent duplicate → cần check GSI trước khi insert

---

#### ✅ Final Checklist

Đảm bảo:

- [ ] Job details hiển thị COMPLETED status
- [ ] Processed records = Total records
- [ ] Failed records = 0 (hoặc có explanation)
- [ ] Data xuất hiện trong workshop-students table
- [ ] Data xuất hiện trong workshop-courses table (nếu có)
- [ ] Import job record trong workshop-import-jobs table
- [ ] GSI queries hoạt động (email-index, studentCode-index)
- [ ] CloudWatch Logs không có errors
- [ ] S3 lifecycle policy configured đúng

#### 🎉 Workshop Hoàn Thành!

Chúc mừng! Bạn đã test thành công toàn bộ flow:
- ✅ Authentication (Cognito)
- ✅ File upload (S3 pre-signed URL)
- ✅ Event-driven processing (S3 Events → Lambda)
- ✅ Excel parsing (Apache POI)
- ✅ Data persistence (DynamoDB)
- ✅ Job tracking

Bây giờ hãy dọn dẹp resources để tránh chi phí!

[➡️ Tiếp theo: Dọn Dẹp Tài Nguyên](../../6.7-cleanup/)
