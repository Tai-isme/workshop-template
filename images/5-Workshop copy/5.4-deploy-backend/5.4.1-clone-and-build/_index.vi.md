---
title : "Clone và Build"
date : "2025-09-15"
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### Clone Source Code

Nếu bạn chưa clone repository, hãy thực hiện bước này.

**Option A: Clone từ Git**

```powershell
# Clone repository
git clone <your-repository-url>

# Di chuyển vào thư mục workshop
cd workshop
```

**Option B: Download ZIP**

1. Tải source code từ repository
2. Giải nén vào thư mục làm việc
3. Mở PowerShell tại thư mục đó

**Kiểm tra cấu trúc:**

```powershell
# List directories
Get-ChildItem -Directory
```

Expected output:
```
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----          2025-12-01    10:00                excel-import-workshop
d----          2025-12-01    10:00                excel-import-frontend
```

---

#### Di Chuyển Vào Backend Directory

```powershell
cd excel-import-workshop
```

Kiểm tra files:
```powershell
Get-ChildItem
```

Expected files:
```
- template.yaml      # SAM template (infrastructure as code)
- pom.xml           # Maven configuration
- samconfig.toml    # SAM deploy configuration (nếu có)
- README.md         # Documentation
- src/              # Java source code
```

---

#### Build Project với Maven

Maven sẽ compile Java code và download tất cả dependencies cần thiết.

**Bước 1: Clean previous builds**

```powershell
mvn clean
```

Output mong đợi:
```
[INFO] Scanning for projects...
[INFO] Building excel-import-workshop 1.0.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] --- maven-clean-plugin:3.1.0:clean (default-clean) @ excel-import-workshop ---
[INFO] Deleting target
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

**Bước 2: Package application**

```powershell
mvn package
```

Quá trình này sẽ:
1. Download dependencies (AWS SDK, Apache POI, Jackson, etc.)
2. Compile Java source code
3. Run unit tests (nếu có)
4. Package thành JAR file

Expected output (cuối cùng):
```
[INFO] --- maven-jar-plugin:3.2.0:jar (default-jar) @ excel-import-workshop ---
[INFO] Building jar: D:\...\excel-import-workshop\target\excel-import-workshop-1.0.0.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  45.678 s
[INFO] Finished at: 2025-12-01T10:15:30+07:00
[INFO] ------------------------------------------------------------------------
```

> ⏱ **Thời gian:** Lần đầu build có thể mất 2-5 phút do phải download dependencies.

**Bước 3: Verify JAR file**

```powershell
Test-Path .\target\excel-import-workshop-1.0.0.jar
```

Output: `True` ✅

---

#### Build với AWS SAM

SAM CLI sẽ chuẩn bị Lambda deployment packages.

```powershell
sam build
```

Quá trình này sẽ:
1. Đọc `template.yaml`
2. Tìm tất cả Lambda functions
3. Copy compiled code từ `target/` vào `.aws-sam/build/`
4. Tạo deployment packages cho từng function

Expected output:
```
Building codeuri: . runtime: java11 metadata: {} architecture: x86_64 functions: RegisterFunction, ConfirmFunction, LoginFunction, LogoutFunction, GenerateUploadUrlFunction, ListImportJobsFunction, GetJobStatusFunction, ImportS3TriggerFunction

Running JavaMavenWorkflow:CopySource
Running JavaMavenWorkflow:MavenBuild
Running JavaMavenWorkflow:MavenCopyDependency
Running JavaMavenWorkflow:MavenCopyArtifacts

Build Succeeded

Built Artifacts  : .aws-sam\build
Built Template   : .aws-sam\build\template.yaml

Commands you can use next
=========================
[*] Validate SAM template: sam validate
[*] Invoke Function: sam local invoke
[*] Test Function in the Cloud: sam sync --stack-name {{stack-name}} --watch
[*] Deploy: sam deploy --guided
```

**Kiểm tra build output:**

```powershell
Get-ChildItem .\.aws-sam\build -Directory
```

Expected output (8 Lambda functions):
```
RegisterFunction
ConfirmFunction
LoginFunction
LogoutFunction
GenerateUploadUrlFunction
ListImportJobsFunction
GetJobStatusFunction
ImportS3TriggerFunction
```

---

#### (Optional) Test Local với SAM Local

Bạn có thể test Lambda functions local trước khi deploy.

**Test một function cụ thể:**

```powershell
# Test RegisterFunction với mock event
sam local invoke RegisterFunction -e events/register-event.json
```

**Start API Gateway local:**

```powershell
# Start local API Gateway (port 3000)
sam local start-api
```

Sau đó test endpoint:
```powershell
# Test register endpoint
Invoke-RestMethod -Uri "http://localhost:3000/register" -Method POST -Body '{"email":"test@example.com","password":"Test1234"}' -ContentType "application/json"
```

> 💡 **Tip:** Local testing yêu cầu Docker running. Nếu không có Docker, có thể skip bước này và deploy thẳng lên AWS.

---

#### Troubleshooting

**Lỗi: "mvn is not recognized"**

Solution:
```powershell
# Check Java installation
java -version

# Check Maven installation
mvn -version

# Nếu chưa có, quay lại phần Prerequisites để cài đặt
```

---

**Lỗi: "Failed to execute goal... compilation failure"**

Nguyên nhân: Java version không đúng hoặc source code có lỗi.

Solution:
```powershell
# Check Java version (cần 11+)
java -version

# Clean và rebuild
mvn clean
mvn package -X  # -X để xem detailed log
```

---

**Lỗi: "sam: command not found"**

Solution:
```powershell
# Kiểm tra SAM CLI
sam --version

# Nếu không có, cài đặt lại SAM CLI
# Download from: https://github.com/aws/aws-sam-cli/releases
```

---

**Lỗi: "Build failed" trong sam build**

Nguyên nhân: `mvn package` chưa chạy hoặc thất bại.

Solution:
```powershell
# Chạy lại Maven build
mvn clean package

# Sau đó chạy sam build
sam build
```

---

#### ✅ Checklist

Trước khi chuyển sang bước tiếp theo, đảm bảo:

- [ ] `mvn clean package` chạy thành công với "BUILD SUCCESS"
- [ ] File JAR tồn tại: `target/excel-import-workshop-1.0.0.jar`
- [ ] `sam build` chạy thành công
- [ ] Thư mục `.aws-sam/build/` chứa 8 Lambda function folders
- [ ] File `.aws-sam/build/template.yaml` tồn tại

#### 🚀 Tiếp Theo

Build thành công! Bây giờ deploy lên AWS.

[➡️ Tiếp theo: SAM Deploy](../6.4.2-sam-deploy/)
