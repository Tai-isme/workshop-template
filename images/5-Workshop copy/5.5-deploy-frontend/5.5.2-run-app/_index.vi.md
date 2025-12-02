---
title: "Chạy Ứng Dụng"
date: "2025-12-01"
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

#### Chạy React Application

Sau khi cấu hình xong, hãy cài đặt dependencies và chạy development server.

#### Cài Đặt Dependencies

**Bước 1: Verify Node.js và npm**

```powershell
node --version
npm --version
```

Expected output:
```
v18.x.x (hoặc v16+)
9.x.x (hoặc 8+)
```

**Bước 2: Install packages**

```powershell
npm install
```

Quá trình này sẽ:
1. Đọc `package.json`
2. Download tất cả dependencies (React, Vite, AWS SDK, Axios, etc.)
3. Tạo thư mục `node_modules/`
4. Tạo file `package-lock.json`

Expected output (cuối cùng):
```
added 234 packages, and audited 235 packages in 45s

89 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

> ⏱ **Thời gian:** Lần đầu install mất ~3-5 phút tùy tốc độ mạng.

**Kiểm tra node_modules:**

```powershell
Test-Path .\node_modules
```

Output: `True` ✅

---

#### Chạy Development Server

```powershell
npm run dev
```

Expected output:
```
> excel-import-frontend@1.0.0 dev
> vite


  VITE v4.5.0  ready in 523 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h to show help
```

> ✅ Development server đang chạy!

**Các thông tin quan trọng:**
- **Local URL:** http://localhost:5173
- **Hot reload:** Enabled (tự động reload khi code thay đổi)
- **Port:** 5173 (default của Vite)

---

#### Truy Cập Ứng Dụng

**Bước 1: Mở trình duyệt**

Mở trình duyệt và truy cập:
```
http://localhost:5173
```

**Bước 2: Xem trang Login**

Bạn sẽ thấy trang đăng nhập với:
- 📧 Email input field
- 🔒 Password input field
- 🔘 "Login" button
- 🔗 "Don't have an account? Register" link

![Frontend Login Page](/images/6-Excel-Workshop/frontend-login.png)

> ✅ Frontend đã chạy thành công!

---

#### Verify Frontend Hoạt Động

**Test 1: Check Console**

1. Mở Developer Tools (F12)
2. Tab **Console**
3. Không có error màu đỏ
4. Có thể thấy log: "App initialized" hoặc tương tự

![Browser Console](/images/6-Excel-Workshop/browser-console.png)

**Test 2: Check Network**

1. Developer Tools → Tab **Network**
2. Không có failed requests (status code 4xx, 5xx)
3. Có thể thấy requests đến `amazonaws.com` (Cognito)

**Test 3: Navigate pages**

- Click "Register" link → Chuyển sang trang đăng ký
- Quay lại Login page
- UI responsive và không có error

---

#### (Optional) Build Production

Nếu muốn build production version:

```powershell
# Build for production
npm run build
```

Output:
```
vite v4.5.0 building for production...
✓ 234 modules transformed.
dist/index.html                   0.45 kB │ gzip:  0.30 kB
dist/assets/index-abc123.css      5.23 kB │ gzip:  1.67 kB
dist/assets/index-def456.js     156.78 kB │ gzip: 52.34 kB
✓ built in 3.45s
```

Files sẽ được tạo trong thư mục `dist/`.

**Preview production build:**

```powershell
npm run preview
```

Access: http://localhost:4173

---

#### Troubleshooting

**Lỗi: "Cannot find module 'vite'"**

Nguyên nhân: `npm install` chưa chạy hoặc thất bại.

Solution:
```powershell
# Clean node_modules
Remove-Item -Recurse -Force .\node_modules
Remove-Item package-lock.json

# Reinstall
npm install
```

---

**Lỗi: "Port 5173 already in use"**

Nguyên nhân: Port đang được sử dụng bởi process khác.

Solution 1: Kill process trên port 5173
```powershell
# Find process
netstat -ano | findstr :5173

# Kill process (thay PID)
taskkill /PID <PID> /F
```

Solution 2: Dùng port khác
```powershell
# Run on different port
npm run dev -- --port 3000
```

---

**Lỗi: "Failed to fetch" khi login**

Nguyên nhân: Frontend không kết nối được backend.

Solution:
```powershell
# Check config.js
Get-Content .\src\config.js

# Verify apiUrl đúng (phải là HTTPS AWS URL)
# Verify CORS enabled trên API Gateway
```

Debug steps:
1. F12 → Network tab
2. Try login
3. Xem request đến API Gateway
4. Check response error message

---

**Lỗi: CORS policy blocked**

Error message:
```
Access to fetch at 'https://...' from origin 'http://localhost:5173' 
has been blocked by CORS policy
```

Solution:
1. Mở API Gateway Console
2. Chọn API → Resources
3. Actions → Enable CORS
4. Confirm và Deploy API lại

---

**Lỗi: "Cannot read property 'apiUrl' of undefined"**

Nguyên nhân: config.js export sai format.

Solution:
```javascript
// File config.js phải có cấu trúc:
export const config = {
  apiUrl: '...',
  cognito: { ... }
};

// KHÔNG phải:
export default { ... }  // ← SAI
```

---

#### Tips Phát Triển

**Hot Reload:**
- Mỗi khi sửa code trong `src/`, browser tự động reload
- Không cần restart `npm run dev`

**View Logs:**
Terminal đang chạy `npm run dev` sẽ hiển thị:
- Build status
- HMR (Hot Module Replacement) updates
- Errors nếu có

**Stop Server:**
```powershell
# Trong terminal đang chạy npm run dev
Ctrl + C

# Confirm: Y
```

**Restart Server:**
```powershell
# Stop trước (Ctrl+C)
# Sau đó chạy lại
npm run dev
```

---

#### Environment Variables (Nâng cao)

Nếu muốn dùng `.env` thay vì hardcode trong `config.js`:

**Tạo file `.env`:**

```powershell
@"
VITE_API_URL=https://abc123.execute-api.us-east-1.amazonaws.com/dev
VITE_USER_POOL_ID=us-east-1_xYzAbC123
VITE_USER_POOL_CLIENT_ID=1a2b3c4d5e6f7g8h9i0j1k2l3m
VITE_REGION=us-east-1
"@ | Out-File .env
```

**Update `config.js`:**

```javascript
export const config = {
  apiUrl: import.meta.env.VITE_API_URL,
  cognito: {
    userPoolId: import.meta.env.VITE_USER_POOL_ID,
    userPoolClientId: import.meta.env.VITE_USER_POOL_CLIENT_ID,
    region: import.meta.env.VITE_REGION
  }
};
```

> 💡 **Tip:** Nhớ thêm `.env` vào `.gitignore` để không commit lên Git!

---

#### ✅ Checklist

Đảm bảo:

- [ ] `npm install` chạy thành công
- [ ] `npm run dev` chạy không có error
- [ ] Truy cập http://localhost:5173 thấy Login page
- [ ] Browser console không có error màu đỏ
- [ ] UI hiển thị đúng (form login, register link)

#### 🎉 Frontend Đã Sẵn Sàng!

Application đang chạy! Bây giờ hãy test toàn bộ flow.

[➡️ Tiếp theo: Test Ứng Dụng](../../6.6-test-application/)
