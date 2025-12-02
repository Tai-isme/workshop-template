---
title : "Kiểm tra Interface Endpoint"
date : "2025-09-15"
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

#### Deploy Backend với AWS SAM

Trong phần này, bạn sẽ deploy toàn bộ backend infrastructure lên AWS sử dụng AWS SAM (Serverless Application Model).

#### 📋 Tổng Quan Quy Trình

```
Clone Source Code
      ↓
Build với Maven (compile Java)
      ↓
Build với SAM (package Lambda)
      ↓
Deploy với SAM (tạo CloudFormation stack)
      ↓
Verify Resources (kiểm tra trên AWS Console)
```

#### 🎯 Kết Quả Sau Khi Deploy

Sau khi hoàn thành phần này, bạn sẽ có:

- ✅ **8 Lambda Functions** đã deploy và sẵn sàng xử lý requests
- ✅ **1 API Gateway** với REST endpoints và Cognito authorization
- ✅ **3 DynamoDB Tables** cho students, courses, import jobs
- ✅ **1 S3 Bucket** với event notifications configured
- ✅ **1 Cognito User Pool** để quản lý users
- ✅ **IAM Roles** cho Lambda execution với least privilege

#### ⏱ Thời Gian Ước Tính

- Clone & Build: ~5 phút
- SAM Deploy: ~8-10 phút
- Verify: ~2 phút
- **Tổng:** ~15-17 phút

#### 💰 Chi Phí

Workshop này nằm trong **AWS Free Tier**. Chi phí thực tế < $0.50 nếu cleanup đúng cách.

#### 📚 Nội Dung

1. [Clone và Build](6.4.1-clone-and-build/)
2. [SAM Deploy](6.4.2-sam-deploy/)
3. [Verify Resources](6.4.3-verify-resources/)

---

#### 🚀 Bắt Đầu

Hãy chuyển sang bước đầu tiên để clone source code và build project!

[➡️ Bắt đầu: Clone và Build](6.4.1-clone-and-build/)
