---
title : "Kiểm thử xác thực"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1 </b> "
---

Phần này kiểm tra toàn bộ luồng xác thực người dùng của SmartDocAI: đăng ký tài khoản, xác nhận email, đăng nhập lấy JWT token, và cơ chế tự động dọn dẹp tài khoản chưa xác nhận qua EventBridge. Các test case dưới đây được thực hiện trực tiếp trên môi trường production qua `curl` và AWS CLI.

### 1. Bảng tổng hợp test case

| # | Test Case | Input | Expected Result | Status |
|---|-----------|-------|-----------------|--------|
| **REGISTRATION** |
| 1.1 | Valid registration | email: test@example.com<br/>password: SecurePass123!<br/>fullname: Nguyen Van Test<br/>phone: 0901234567<br/>dob: 1990-01-15 | 200 OK, confirmation email sent<br/>Return: user_id + email | PASS |
| 1.2 | Invalid phone format | phone: "123456" (wrong format) | 400 Bad Request<br/>Error: "Invalid phone format" | PASS |
| 1.3 | Invalid DOB | dob: "2050-01-01" (future date) | 400 Bad Request<br/>Error: "DOB must be between 1900-01-01 and 2026-12-31" | PASS |
| 1.4 | XSS attempt | fullname: `<script>alert(1)</script>` | 400 Bad Request<br/>Error: "Fullname contains invalid characters" | PASS |
| **EMAIL CONFIRMATION** |
| 2.1 | Valid confirmation | email + 6-digit code (from email) | 200 OK<br/>Message: "Email confirmed. You can now login" | PASS |
| 2.2 | Invalid code | email + wrong code "999999" | 400 Bad Request<br/>Error: "Invalid or expired code" | PASS |
| **LOGIN** |
| 3.1 | Successful login | email + correct password | 200 OK<br/>Return: JWT tokens (ID + Access + Refresh) | PASS |
| 3.2 | Wrong password | email + wrong password | 401 Unauthorized<br/>Error: "Incorrect username or password" | PASS |
| 3.3 | Unconfirmed user | email not confirmed yet | 401 Unauthorized<br/>Error: "User not confirmed" | PASS |
| **JWT TOKEN** |
| 4.1 | Valid JWT | Include Bearer token in Authorization header | API returns user data | PASS |
| 4.2 | Missing JWT | No Authorization header | 401 Unauthorized<br/>Error: "Missing Authorization header" | PASS |
| 4.3 | Invalid JWT | Bearer token with wrong signature | 401 Unauthorized<br/>Error: "Invalid JWT token" | PASS |
| 4.4 | Expired JWT | Token with exp < current time | 401 Unauthorized<br/>Error: "JWT token has expired" | PASS |
| **EVENTBRIDGE CLEANUP** |
| 5.1 | Auto-delete unconfirmed users | Wait >5 minutes after registration | User deleted from Cognito<br/>CloudWatch log: "Deleted user X" | PASS |

**Quy tắc validate dữ liệu:**
- **Email:** Định dạng chuẩn (user@domain.com)
- **Password:** Tối thiểu 8 ký tự, có chữ hoa, chữ thường, số, ký tự đặc biệt
- **Phone:** Định dạng Việt Nam (0901234567) hoặc E.164 (+84901234567)
- **DOB:** YYYY-MM-DD, trong khoảng 1900-2026
- **Fullname:** 2-100 ký tự, không chứa thẻ `<script>`

**API Endpoint:** `POST https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod/auth/{action}`

**Expected response:**
```json
{
  "error": "Invalid confirmation code or code has expired"
}
```

**Ghi chú:**
- Confirmation code expire sau 24 giờ
- Nếu không confirm trong 5 phút, EventBridge rule tự động xóa user

---

### 2. Đăng nhập & JWT Token

**Endpoint:** `POST https://.../prod/auth/login`

**Test case: Đăng nhập thành công**
```bash
curl -X POST https://.../prod/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "SecurePass123!"
  }'
```

**Expected response:**
```json
{
  "message": "Login successful",
  "id_token": "eyJraWQiOiJ...",
  "access_token": "eyJraWQiOiJ...",
  "refresh_token": "eyJjdHki...",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

**Test case: Sai mật khẩu**
```bash
curl -X POST .../auth/login \
  -d '{"email": "test@example.com", "password": "WrongPass"}'
```

**Expected response:**
```json
{
  "error": "Incorrect email or password"
}
```

**Kiểm tra JWT:**
```bash
# Decode ID token để xem claims
echo "eyJraWQiOiJ..." | base64 -d | jq .

# Expected payload:
{
  "sub": "uuid",
  "email": "test@example.com",
  "cognito:username": "test@example.com",
  "custom:fullname": "Nguyen Van Test",
  "custom:phone": "+84901234567",
  "custom:dob": "1990-01-15",
  "exp": 1234567890,
  "iat": 1234564290
}
```

**Gọi API kèm JWT:**
```bash
# Get user profile với JWT token
curl -X GET https://.../prod/profile \
  -H "Authorization: Bearer eyJraWQiOiJ..."
```

**Expected response:**
```json
{
  "user_id": "uuid",
  "email": "test@example.com",
  "fullname": "Nguyen Van Test",
  "phone": "+84901234567",
  "dob": "1990-01-15",
  "avatar_url": null,
  "created_at": 1234567890,
  "updated_at": 1234567890
}
```

---

### 3. Kiểm thử tự động dọn dẹp qua EventBridge

**Mục đích:** Xác minh rằng EventBridge rule tự động xóa các tài khoản UNCONFIRMED sau 5 phút, tránh tính trạng tài khoản rác tích tụ trong Cognito.

**Quy trình test:**
1. Tạo user mới (không xác nhận email)
2. Chờ 6 phút
3. Kiểm tra CloudWatch Logs `/aws/lambda/smartdocai`
4. Thử đăng nhập lại → phải thất bại (user đã bị xóa)

**Kiểm tra EventBridge đã trại qua chạy:**
```bash
aws events list-rules --region us-east-1 | grep smartdocai

# Expected output:
{
  "Name": "smartdocai-cleanup-unconfirmed",
  "Arn": "arn:aws:events:us-east-1:...",
  "State": "ENABLED",
  "ScheduleExpression": "rate(5 minutes)"
}
```

**Kiểm tra CloudWatch Logs:**
```bash
aws logs filter-log-events \
  --log-group-name /aws/lambda/smartdocai \
  --filter-pattern "Deleted.*users" \
  --start-time $(date -u -d '10 minutes ago' +%s)000 \
  --region us-east-1
```

**Kết quả log mong đợi:**
```
[Cleanup] Deleted 3 users: ['user1@example.com', 'user2@example.com', 'user3@example.com']
```

**Xác minh user đã bị xóa trong Cognito:**
```bash
aws cognito-idp list-users \
  --user-pool-id us-east-1_3oq5wIiuu \
  --filter "email = \"test@example.com\"" \
  --region us-east-1

# Expected: Empty list (user not found)
```

---