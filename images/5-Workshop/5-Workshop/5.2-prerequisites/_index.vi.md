---
title : "Các bước chuẩn bị"
date :  "2025-09-15" 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### Yêu Cầu Môi Trường

Để hoàn thành workshop này, bạn cần chuẩn bị các công cụ và quyền truy cập sau.

#### AWS Account & Permissions

*AWS Account:*
- Tài khoản AWS có thể sử dụng Free Tier
- IAM user có quyền với các services:
  - AWS CloudFormation
  - AWS Lambda
  - Amazon API Gateway
  - Amazon S3
  - Amazon DynamoDB
  - Amazon Cognito
  - IAM
  - CloudWatch Logs


**AWS CLI:**

```
![alt text](image.png)
```

---

**AWS SAM CLI:**

```
SAM CLI, version 1.x.x
```
---

#### Cấu Hình AWS CLI

**Bước 1: Cấu hình credentials**

```powershell
aws configure
```

Nhập thông tin:
```
AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]: json
```

> ⚠️ **Lưu ý:** 
> - Region khuyến nghị: `us-east-1` (N. Virginia) hoặc `ap-southeast-1` (Singapore)
> - Không share Access Key/Secret Key lên GitHub hoặc public repositories

**Bước 2: Verify configuration**

```powershell
aws sts get-caller-identity
```

Expected output:
```json
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-username"
}
```

Nếu thấy output như trên → Cấu hình thành công! ✅

#### 5️⃣ Clone Workshop Repository

**Option A: Clone từ Git (nếu có repo)**

```powershell
git clone <your-repository-url>
cd workshop
```

**Option B: Download ZIP**

1. Download source code từ repository
2. Extract vào thư mục làm việc
3. Mở terminal tại thư mục đó

**Cấu trúc thư mục sau khi clone:**

```
workshop/
├── excel-import-workshop/       # Backend (Java + SAM)
│   ├── src/
│   ├── template.yaml
│   ├── pom.xml
│   └── README.md
└── excel-import-frontend/       # Frontend (React)
    ├── src/
    ├── package.json
    └── vite.config.js
```

#### 6️⃣ Kiểm Tra Tổng Hợp

Chạy script kiểm tra nhanh:

```powershell
# Check Java
java -version

# Check Maven
mvn -version

# Check AWS CLI
aws --version

# Check SAM CLI
sam --version

# Check Node.js
node --version

# Check npm
npm --version

# Check AWS credentials
aws sts get-caller-identity
```

Nếu tất cả commands trên đều chạy thành công → Bạn đã sẵn sàng! 🎉

#### 7️⃣ Troubleshooting

**Lỗi: "java is not recognized"**
- Cài đặt JDK và thêm `JAVA_HOME` vào Environment Variables
- Thêm `%JAVA_HOME%\bin` vào PATH

**Lỗi: "mvn is not recognized"**
- Cài đặt Maven và thêm `MAVEN_HOME` vào Environment Variables
- Thêm `%MAVEN_HOME%\bin` vào PATH

**Lỗi: "Unable to locate credentials"**
- Chạy lại `aws configure`
- Kiểm tra file `~\.aws\credentials` có tồn tại không

**Lỗi: "SAM CLI not found"**
- Reinstall SAM CLI
- Restart terminal sau khi cài đặt

#### ✅ Checklist

Đánh dấu các items sau trước khi chuyển sang bước tiếp theo:

- [ ] AWS Account đã có và credentials đã cấu hình
- [ ] Java 11+ đã cài và `java -version` chạy được
- [ ] Maven đã cài và `mvn -version` chạy được
- [ ] AWS CLI đã cài và `aws sts get-caller-identity` chạy được
- [ ] SAM CLI đã cài và `sam --version` chạy được
- [ ] Node.js 16+ đã cài và `node --version` chạy được
- [ ] Source code đã download/clone về máy
- [ ] Code editor đã cài đặt (VS Code recommended)

#### 🚀 Tiếp Theo

Môi trường đã sẵn sàng! Hãy tìm hiểu kiến trúc hệ thống trước khi bắt đầu deploy.

[➡️ Tiếp theo: Kiến Trúc Hệ Thống](../6.3-architecture/)
