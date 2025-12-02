---
title: "Đăng Ký & Đăng Nhập"
date: "2025-12-01"
weight: 1
chapter: false
pre: " <b> 5.6.1. </b> "
---

#### Test Authentication Flow

Hãy test Cognito authentication bằng cách đăng ký user mới và đăng nhập.

#### Đăng Ký User Mới

**Bước 1: Truy cập Register Page**

1. Mở trình duyệt: http://localhost:5173
2. Click link **"Don't have an account? Register"**
3. Bạn sẽ thấy Registration Form

![Register Page](/images/6-Excel-Workshop/register-page.png)

**Bước 2: Điền thông tin**

```
Email: your-email@example.com
Password: Workshop123
Confirm Password: Workshop123
```

> ⚠️ **Password requirements:**
> - Tối thiểu 8 ký tự
> - Có ít nhất 1 chữ hoa (A-Z)
> - Có ít nhất 1 chữ thường (a-z)
> - Có ít nhất 1 số (0-9)

**Bước 3: Click "Sign Up"**

Frontend sẽ:
1. Validate input
2. Call API: `POST /register`
3. Lambda gọi Cognito.signUp()
4. Cognito gửi OTP code qua email

Expected response:
```
Registration successful! Please check your email for verification code.
```

![Register Success](/images/6-Excel-Workshop/register-success.png)

> ⏱ Email có thể mất 1-2 phút để nhận được.

---

#### Xác Nhận Email (Verify)

**Bước 4: Check email**

1. Mở email inbox (hoặc spam folder)
2. Tìm email từ: `no-reply@verificationemail.com`
3. Subject: "Your verification code"
4. Copy **6-digit code** (ví dụ: `123456`)

![Verification Email](/images/6-Excel-Workshop/verification-email.png)

**Bước 5: Nhập verification code**

1. Frontend tự động chuyển đến Verify Page (hoặc click link trong notification)
2. Nhập 6-digit code
3. Click **"Confirm"**

![Verify Page](/images/6-Excel-Workshop/verify-page.png)

Expected response:
```
Email verified successfully! You can now login.
```

Frontend tự động redirect về Login Page.

---

#### Đăng Nhập

**Bước 6: Login với account vừa tạo**

1. Nhập email đã đăng ký
2. Nhập password
3. Click **"Login"**

![Login Page](/images/6-Excel-Workshop/login-page.png)

**Process flow:**

```
Frontend → POST /login
           ↓
        LoginFunction (Lambda)
           ↓
        Cognito.initiateAuth()
           ↓
        Return JWT tokens:
          - IdToken (for API authorization)
          - AccessToken (for user info)
          - RefreshToken (for renew)
           ↓
        Frontend lưu tokens vào localStorage
           ↓
        Redirect to Import Page
```

Expected result:
- ✅ Chuyển sang Import Page
- ✅ Thấy user email ở góc trên (header)
- ✅ Thấy "Upload Excel" form

![Import Page](/images/6-Excel-Workshop/import-page.png)

---

#### Verify Authentication State

**Check localStorage (Developer Tools):**

1. F12 → Tab **Application** (Chrome) hoặc **Storage** (Firefox)
2. Expand **Local Storage** → http://localhost:5173
3. Xem keys:

```
userTokens = {
  "IdToken": "eyJraWQiOiJ...",
  "AccessToken": "eyJraWQiOiJ...",
  "RefreshToken": "eyJjdHki..."
}
userEmail = your-email@example.com
```

![LocalStorage Tokens](/images/6-Excel-Workshop/localStorage-tokens.png)

> ✅ Nếu thấy tokens → Authentication thành công!

**Check Network Request:**

1. F12 → Tab **Network**
2. Find request: `login` (hoặc POST /login)
3. Response phải có:

```json
{
  "statusCode": 200,
  "body": {
    "tokens": {
      "IdToken": "...",
      "AccessToken": "...",
      "RefreshToken": "..."
    },
    "user": {
      "email": "your-email@example.com"
    }
  }
}
```

---

#### (Optional) Test Logout

**Test logout flow:**

1. Click **"Logout"** button (ở header)
2. Frontend gọi: `POST /logout`
3. Lambda gọi: `Cognito.globalSignOut()`
4. Clear localStorage
5. Redirect về Login Page

Expected result:
- ✅ Về trang Login
- ✅ localStorage.userTokens bị xóa
- ✅ Không thể access Import Page nếu chưa login

**Test lại bằng cách:**

```
1. Sau khi logout, try access: http://localhost:5173/import
2. Phải bị redirect về /login (vì không có token)
```

> ✅ Protected routes hoạt động!

---

#### Troubleshooting

**Lỗi: "Email already exists"**

Nguyên nhân: Email đã được đăng ký trước đó.

Solution:
- Dùng email khác
- Hoặc delete user từ Cognito Console:
  1. Mở Cognito Console → User Pool
  2. Tab **Users**
  3. Tìm và delete user cũ

---

**Lỗi: "Verification code expired"**

Nguyên nhân: Code chỉ valid trong 24 giờ.

Solution:
```powershell
# Resend code bằng AWS CLI
aws cognito-idp resend-confirmation-code `
  --client-id YOUR_CLIENT_ID `
  --username your-email@example.com
```

Hoặc đăng ký lại với email mới.

---

**Lỗi: "Invalid password"**

Nguyên nhân: Password không đáp ứng yêu cầu.

Solution:
- Check password policy (min 8 chars, uppercase, lowercase, number)
- Ví dụ valid: `Workshop123`, `Test@2024`
- Ví dụ invalid: `test123` (không có uppercase)

---

**Lỗi: "Incorrect username or password" khi login**

Nguyên nhân:
- Email hoặc password sai
- Hoặc account chưa verify email

Solution:
1. Verify lại email (check inbox/spam)
2. Nếu quên password:
```powershell
# Forgot password flow (qua CLI)
aws cognito-idp forgot-password `
  --client-id YOUR_CLIENT_ID `
  --username your-email@example.com
```

---

**Lỗi: "Network error" khi call API**

Nguyên nhân: Frontend không kết nối được backend.

Solution:
```powershell
# Check config.js
Get-Content .\src\config.js

# Test API Gateway manually
Invoke-RestMethod -Uri "https://YOUR_API_URL/dev/login" -Method POST -Body '{"email":"test@test.com","password":"Test123"}' -ContentType "application/json"
```

---

#### Test Với AWS CLI (Advanced)

**Register user qua CLI:**

```powershell
aws cognito-idp sign-up `
  --client-id YOUR_CLIENT_ID `
  --username test@example.com `
  --password Test1234 `
  --region us-east-1
```

**Confirm user qua CLI (skip email verification - admin only):**

```powershell
aws cognito-idp admin-confirm-sign-up `
  --user-pool-id YOUR_POOL_ID `
  --username test@example.com `
  --region us-east-1
```

> 💡 Useful cho testing nhanh!

---

#### ✅ Checklist

Đảm bảo:

- [ ] Đăng ký user thành công
- [ ] Nhận được verification email
- [ ] Confirm email thành công
- [ ] Login thành công
- [ ] JWT tokens lưu trong localStorage
- [ ] Redirect đến Import Page sau login
- [ ] Logout hoạt động đúng (optional)

#### 🎉 Authentication Hoạt Động!

Bây giờ hãy test upload file Excel!

[➡️ Tiếp theo: Upload File Excel](../6.6.2-upload-excel/)
