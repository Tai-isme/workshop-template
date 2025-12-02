---
title: "Test Ứng Dụng"
date: "2025-12-01"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Test End-to-End Application

Trong phần này, bạn sẽ test toàn bộ flow từ đăng ký user đến upload Excel và xem kết quả.

#### 📋 Test Scenarios

```
1. Đăng ký user mới → Xác nhận email
      ↓
2. Đăng nhập → Nhận JWT tokens
      ↓
3. Upload file Excel → Trigger Lambda processing
      ↓
4. Theo dõi import job status
      ↓
5. Verify dữ liệu trong DynamoDB
```

#### 🎯 Mục Tiêu

- ✅ Test Cognito authentication flow
- ✅ Test file upload với pre-signed URL
- ✅ Test S3 Event trigger Lambda
- ✅ Test Excel parsing và DynamoDB insert
- ✅ Verify import job tracking

#### ⏱ Thời Gian Ước Tính

- Register & Login: ~3 phút
- Upload Excel: ~2 phút
- Check Results: ~3 phút
- **Tổng:** ~8 phút

#### 📚 Nội Dung

1. [Đăng Ký & Đăng Nhập](6.6.1-register-login/)
2. [Upload File Excel](6.6.2-upload-excel/)
3. [Kiểm Tra Kết Quả](6.6.3-check-results/)

---

#### 🚀 Bắt Đầu Testing

Hãy bắt đầu với việc tạo tài khoản user!

[➡️ Bắt đầu: Đăng Ký & Đăng Nhập](6.6.1-register-login/)
