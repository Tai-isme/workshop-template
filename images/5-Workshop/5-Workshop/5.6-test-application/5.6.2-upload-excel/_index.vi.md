---
title: "Upload File Excel"
date: "2025-12-01"
weight: 2
chapter: false
pre: " <b> 5.6.2. </b> "
---

#### Test File Upload Flow

Hãy test upload file Excel và trigger Lambda processing.

#### Chuẩn Bị File Excel Mẫu

**Option 1: Tạo file Excel thủ công**

Tạo file `students.xlsx` với cấu trúc sau:

| studentCode | fullName | email | courseCode |
|-------------|----------|-------|------------|
| ST001 | Nguyen Van A | nguyenvana@test.com | CS101 |
| ST002 | Tran Thi B | tranthib@test.com | CS101 |
| ST003 | Le Van C | levanc@test.com | CS102 |
| ST004 | Pham Thi D | phamthid@test.com | CS102 |
| ST005 | Hoang Van E | hoangvane@test.com | CS103 |

**Header row (row 1):**
- Column A: `studentCode`
- Column B: `fullName`
- Column C: `email`
- Column D: `courseCode`

**Data rows (row 2-6):**
- Điền theo bảng trên

> 💡 **Tip:** Lưu file dưới format `.xlsx` (Excel 2007+), không phải `.xls` (Excel 97-2003).

---

**Option 2: Download file mẫu (nếu có)**

Nếu repository có file mẫu:

```powershell
# File mẫu thường nằm trong examples/
Get-ChildItem ..\examples\*.xlsx
```

Copy file vào Desktop hoặc thư mục dễ truy cập.

---

**Option 3: Tạo bằng PowerShell (Advanced)**

```powershell
# Tạo CSV rồi convert sang XLSX bằng Excel COM
$data = @"
studentCode,fullName,email,courseCode
ST001,Nguyen Van A,nguyenvana@test.com,CS101
ST002,Tran Thi B,tranthib@test.com,CS101
ST003,Le Van C,levanc@test.com,CS102
ST004,Pham Thi D,phamthid@test.com,CS102
ST005,Hoang Van E,hoangvane@test.com,CS103
"@

$data | Out-File students.csv -Encoding UTF8

# Mở Excel và save as .xlsx manually
# Hoặc dùng script với Excel COM object
```

---

#### Upload File Trên Frontend

**Bước 1: Verify đã login**

- Đảm bảo bạn đang ở Import Page
- Thấy email user ở header
- Thấy "Upload Excel File" section

**Bước 2: Chọn file**

1. Click button **"Choose File"** hoặc **"Browse"**
2. Navigate đến file `students.xlsx`
3. Select và Open
4. File name hiển thị bên cạnh button

![Choose File](/images/6-Excel-Workshop/choose-file.png)

**Bước 3: Click "Upload"**

Process flow:
```
1. Frontend validate file:
   - Extension phải là .xlsx hoặc .xls
   - Size không quá 10MB (configurable)
   
2. Frontend gọi API: POST /upload-url
   Headers: { Authorization: Bearer {IdToken} }
   Body: { fileName: "students.xlsx", fileSize: 12345 }
   
3. GenerateUploadUrlFunction:
   a. Create import job record trong DynamoDB
   b. Generate S3 pre-signed URL (PUT)
   c. Return: { jobId, uploadUrl, expiresIn: 900 }
   
4. Frontend upload file:
   PUT {uploadUrl}
   Body: file binary (không qua server!)
   
5. S3 lưu file thành công
   
6. S3 Event trigger ImportS3TriggerFunction
   
7. Lambda parse Excel và import vào DynamoDB
```

**Expected UI feedback:**

```
Uploading... [Progress bar: 0%]
         ↓
Uploading... [Progress bar: 50%]
         ↓
Upload successful! [Progress bar: 100%]
         ↓
Job created: abc-123-def-456
Processing...
```

![Upload Progress](/images/6-Excel-Workshop/upload-progress.png)

> ⏱ Upload thường mất 2-5 giây tùy file size.

---

#### Theo Dõi Import Job

**Sau khi upload xong:**

1. Frontend tự động redirect đến **Jobs List** (hoặc refresh page)
2. Bạn sẽ thấy job vừa tạo trong danh sách

**Job List hiển thị:**

| Job ID | File Name | Status | Created | Actions |
|--------|-----------|--------|---------|---------|
| abc-123... | students.xlsx | 🟡 PROCESSING | 10:30 AM | View Details |

**Job Status Colors:**

- 🔵 **PENDING** - Đang chờ xử lý (vừa upload xong)
- 🟡 **PROCESSING** - Lambda đang parse file
- 🟢 **COMPLETED** - Hoàn thành thành công
- 🔴 **FAILED** - Có lỗi xảy ra

![Jobs List](/images/6-Excel-Workshop/jobs-list.png)

**Bước 4: Wait for processing**

- Lambda có thể mất 5-30 giây để parse file (tùy số records)
- Frontend tự động refresh status mỗi 3-5 giây (polling)
- Quan sát status chuyển từ PENDING → PROCESSING → COMPLETED

```
Initial:     🔵 PENDING
After 2s:    🟡 PROCESSING
After 10s:   🟢 COMPLETED ✅
```

> 💡 **Tip:** Nếu status không tự động update, refresh browser (F5).

---

#### Verify Upload Trên S3

**Check file đã lên S3:**

```powershell
# List files trong bucket
aws s3 ls s3://workshop-excel-imports-YOUR_ACCOUNT_ID/imports/

# Output:
# 2025-12-01 10:30:00    12345 abc-123-def-456-students.xlsx
```

Hoặc qua Console:

1. Mở [S3 Console](https://s3.console.aws.amazon.com)
2. Bucket: `workshop-excel-imports-{AccountId}`
3. Prefix: `imports/`
4. Thấy file: `{jobId}-students.xlsx`

![S3 File Uploaded](/images/6-Excel-Workshop/s3-file-uploaded.png)

---

#### Check Lambda Processing Logs

**Xem logs của ImportS3TriggerFunction:**

1. Mở [CloudWatch Logs Console](https://console.aws.amazon.com/cloudwatch/home#logsV2:log-groups)
2. Tìm log group: `/aws/lambda/ImportS3Trigger`
3. Click vào log group
4. Xem **Latest log stream**

Expected logs:

```
START RequestId: abc-123-def...
INFO: Processing file: imports/abc-123-students.xlsx
INFO: Parsing Excel file...
INFO: Found 5 rows (excluding header)
INFO: Processing row 1: ST001 - Nguyen Van A
INFO: Processing row 2: ST002 - Tran Thi B
...
INFO: Batch writing 5 students to DynamoDB
INFO: Batch writing 3 courses to DynamoDB
INFO: Updating job status to COMPLETED
INFO: Total: 5, Processed: 5, Failed: 0
END RequestId: abc-123-def...
REPORT Duration: 8432.12 ms Memory Used: 256 MB
```

![CloudWatch Logs](/images/6-Excel-Workshop/cloudwatch-processing-logs.png)

> ✅ Nếu thấy "COMPLETED" và "Failed: 0" → Success!

---

#### Troubleshooting

**Lỗi: "File type not supported"**

Nguyên nhân: File không phải .xlsx/.xls

Solution:
- Save lại file Excel dưới format .xlsx
- Không dùng .csv, .txt, hoặc format khác

---

**Lỗi: "Upload failed" - 403 Forbidden**

Nguyên nhân: Pre-signed URL expired hoặc CORS issue.

Solution:
```powershell
# Check S3 CORS configuration
aws s3api get-bucket-cors --bucket workshop-excel-imports-YOUR_ACCOUNT_ID

# Verify Lambda có quyền S3 PutObject
```

Debug:
1. F12 → Network tab
2. Find PUT request đến s3.amazonaws.com
3. Check response error

---

**Lỗi: Job status stuck ở "PENDING"**

Nguyên nhân: Lambda không được trigger, hoặc có lỗi.

Solution:
1. Check S3 Event Notifications configured:
   - Bucket → Properties → Event notifications
   - Verify Lambda target đúng

2. Check Lambda CloudWatch Logs:
   - Có log mới không?
   - Có error gì không?

3. Manually invoke Lambda:
```powershell
# Test Lambda với mock event
aws lambda invoke --function-name ImportS3Trigger --payload '{"Records":[{"s3":{"bucket":{"name":"YOUR_BUCKET"},"object":{"key":"imports/test.xlsx"}}}]}' response.json
```

---

**Lỗi: Job status = "FAILED"**

Nguyên nhân: Lambda parse file thất bại.

Debug steps:
1. Click "View Details" của job để xem error message
2. Check CloudWatch Logs để xem chi tiết lỗi
3. Common errors:
   - Invalid Excel format (corrupted file)
   - Missing required columns (studentCode, fullName, email)
   - Invalid email format
   - Duplicate studentCode/email

Solution:
- Fix file Excel theo đúng format
- Re-upload file

---

**Lỗi: "Authorization failed" khi gọi /upload-url**

Nguyên nhân: JWT token không valid hoặc expired.

Solution:
```powershell
# Check token trong localStorage
# F12 → Application → Local Storage → userTokens

# Nếu expired (1 hour), logout và login lại
```

---

#### Test Edge Cases

**Test 1: Upload file lớn**

Tạo file với 100+ rows:
```powershell
# Generate large CSV
1..100 | ForEach-Object {
  "ST{0:D3},Student {0},student{0}@test.com,CS101" -f $_
} | Out-File large-students.csv
```

Convert sang .xlsx và upload.

Expected: Lambda xử lý thành công (có thể mất 20-30s).

---

**Test 2: Upload file có lỗi**

Tạo file với email invalid:

| studentCode | fullName | email | courseCode |
|-------------|----------|-------|------------|
| ST001 | Test User | INVALID_EMAIL | CS101 |

Expected: 
- Job status = FAILED
- Error message: "Invalid email format at row 2"

---

**Test 3: Duplicate studentCode**

Upload file có 2 rows cùng studentCode `ST001`.

Expected:
- DynamoDB overwrite record cũ (nếu không có unique constraint)
- Hoặc Lambda detect duplicate và skip

---

#### ✅ Checklist

Đảm bảo:

- [ ] File Excel đã tạo với đúng format
- [ ] Upload file thành công từ frontend
- [ ] Job được tạo trong danh sách
- [ ] Job status chuyển từ PENDING → PROCESSING → COMPLETED
- [ ] File xuất hiện trong S3 bucket
- [ ] CloudWatch Logs hiển thị processing success
- [ ] Không có error trong logs

#### 🎉 Upload & Processing Hoạt Động!

Bây giờ hãy verify dữ liệu đã được lưu vào DynamoDB!

[➡️ Tiếp theo: Kiểm Tra Kết Quả](../6.6.3-check-results/)
