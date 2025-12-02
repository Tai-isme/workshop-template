---
title: "Cấu hình Frontend"
date: "2025-12-01"
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

#### Cấu Hình Frontend

Frontend cần kết nối với backend AWS thông qua các thông tin từ SAM deploy outputs.

#### Di Chuyển Vào Frontend Directory

```powershell
# Từ thư mục gốc workshop
cd ..\excel-import-frontend

# Hoặc nếu đang ở backend folder
cd ..\excel-import-frontend
```

Verify thư mục:
```powershell
Get-ChildItem
```

Expected files:
```
- package.json
- vite.config.js
- index.html
- src/
  - config.js    ← File cần cập nhật
  - App.jsx
  - main.jsx
  - components/
```

---

#### Lấy Backend Outputs

Nếu bạn chưa lưu outputs từ bước 6.4.2, hãy lấy lại:

**Option 1: SAM CLI**

```powershell
cd ..\excel-import-workshop
sam list stack-outputs --stack-name excel-import-workshop
```

**Option 2: AWS CLI**

```powershell
aws cloudformation describe-stacks --stack-name excel-import-workshop --query "Stacks[0].Outputs" --output table
```

**Option 3: Copy từ notepad** (nếu bạn đã lưu)

Bạn cần 4 giá trị:
```
ApiUrl           = https://abc123def4.execute-api.us-east-1.amazonaws.com/dev
BucketName       = workshop-excel-imports-123456789012
UserPoolId       = us-east-1_xYzAbC123
UserPoolClientId = 1a2b3c4d5e6f7g8h9i0j1k2l3m
```

---

#### Cập Nhật File config.js

**Bước 1: Mở file config**

```powershell
# Mở bằng notepad
notepad .\src\config.js

# Hoặc VS Code
code .\src\config.js
```

**Bước 2: Thay đổi cấu hình**

File gốc trông như này:

```javascript
export const config = {
  apiUrl: 'http://localhost:3000',
  cognito: {
    userPoolId: 'YOUR_USER_POOL_ID',
    userPoolClientId: 'YOUR_USER_POOL_CLIENT_ID',
    region: 'us-east-1'
  }
};
```

**Cập nhật thành:**

```javascript
export const config = {
  apiUrl: 'https://abc123def4.execute-api.us-east-1.amazonaws.com/dev',
  cognito: {
    userPoolId: 'us-east-1_xYzAbC123',
    userPoolClientId: '1a2b3c4d5e6f7g8h9i0j1k2l3m',
    region: 'us-east-1'
  }
};
```

> ⚠️ **Quan trọng:** 
> - Thay `apiUrl` bằng **ApiUrl** từ SAM outputs
> - Thay `userPoolId` bằng **UserPoolId**
> - Thay `userPoolClientId` bằng **UserPoolClientId**
> - `region` phải khớp với region bạn deploy backend

**Bước 3: Lưu file**

- Notepad: Ctrl+S
- VS Code: Ctrl+S

---

#### Verify Configuration

Kiểm tra lại file đã đúng:

```powershell
Get-Content .\src\config.js
```

Output phải có:
- ✅ `apiUrl` bắt đầu với `https://` (không phải `http://localhost`)
- ✅ `userPoolId` format: `{region}_{randomString}`
- ✅ `userPoolClientId` là một chuỗi alphanumeric dài
- ✅ `region` khớp với backend region

---

#### (Optional) Cấu Hình Thêm

**Nếu muốn thay đổi port (mặc định 5173):**

Mở `vite.config.js`:

```javascript
export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000, // Thay đổi port nếu muốn
    host: true
  }
})
```

**Nếu cần configure CORS (nâng cao):**

CORS đã được config sẵn trong API Gateway template, nhưng nếu gặp lỗi CORS:

1. Mở API Gateway Console
2. Chọn API → Resources
3. Enable CORS cho tất cả methods
4. Deploy lại API (Actions → Deploy API)

---

#### Troubleshooting

**Lỗi: "config.js not found"**

Solution:
```powershell
# Verify đang ở đúng directory
Get-Location

# Phải là: ...\excel-import-frontend
cd excel-import-frontend

# Check file tồn tại
Test-Path .\src\config.js
```

---

**Lỗi: "Invalid config format"**

Nguyên nhân: JavaScript syntax error.

Solution:
- Kiểm tra dấu ngoặc `{`, `}`, `,`, `'`
- Mỗi property phải có dấu `,` (trừ item cuối)
- String phải trong dấu `'` hoặc `"`

Valid format:
```javascript
export const config = {
  apiUrl: 'https://...',  // ← dấu phẩy
  cognito: {
    userPoolId: 'us-east-1_xxx',  // ← dấu phẩy
    userPoolClientId: 'xxx',      // ← dấu phẩy
    region: 'us-east-1'           // ← KHÔNG có dấu phẩy (item cuối)
  }
};
```

---

**Config nhưng không chắc đúng?**

Test bằng cách import:

```powershell
# Tạo test file
@"
import { config } from './src/config.js';
console.log('API URL:', config.apiUrl);
console.log('User Pool ID:', config.cognito.userPoolId);
"@ | Out-File test-config.mjs

# Run với Node.js
node test-config.mjs

# Xóa test file
Remove-Item test-config.mjs
```

---

#### ✅ Checklist

Trước khi chuyển sang bước tiếp theo:

- [ ] Đã lấy được 4 outputs từ backend deploy
- [ ] File `src/config.js` đã được cập nhật
- [ ] `apiUrl` là HTTPS URL từ API Gateway
- [ ] `userPoolId` và `userPoolClientId` đã điền đúng
- [ ] `region` khớp với backend deployment region
- [ ] File config không có syntax error (check dấu ngoặc, phẩy)

#### 🚀 Tiếp Theo

Cấu hình xong! Bây giờ cài đặt dependencies và chạy app.

[➡️ Tiếp theo: Chạy Ứng Dụng](../6.5.2-run-app/)
