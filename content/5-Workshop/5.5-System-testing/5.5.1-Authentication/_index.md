---
title : "Authentication Testing"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1 </b> "
---

This section tests the entire user authentication flow of SmartDocAI: account registration, email confirmation, login to obtain a JWT token, and the automatic cleanup mechanism for unconfirmed accounts via EventBridge. The test cases below were performed directly against the production environment using `curl` and the AWS CLI.

### 1. Test Case Summary Table

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

**Data validation rules:**
- **Email:** Standard format (user@domain.com)
- **Password:** Minimum 8 characters, uppercase, lowercase, number, special character
- **Phone:** Vietnam format (0901234567) or E.164 (+84901234567)
- **DOB:** YYYY-MM-DD, range 1900-2026
- **Fullname:** 2-100 characters, no `<script>` tags

**API Endpoint:** `POST https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod/auth/{action}`

**Expected response:**
```json
{
  "error": "Invalid confirmation code or code has expired"
}
```

**Notes:**
- Confirmation code expires after 24 hours
- If not confirmed within 5 minutes, the EventBridge rule automatically deletes the user

---

### 2. Login & JWT Token

**Endpoint:** `POST https://.../prod/auth/login`

**Test case: Successful login**
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

**Test case: Wrong password**
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

**Checking the JWT:**
```bash
# Decode the ID token to view claims
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

**Calling the API with a JWT:**
```bash
# Get user profile with JWT token
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

### 3. Automatic Cleanup Testing via EventBridge

**Purpose:** Verify that the EventBridge rule automatically deletes UNCONFIRMED accounts after 5 minutes, preventing junk accounts from accumulating in Cognito.

**Test procedure:**
1. Create a new user (do not confirm the email)
2. Wait 6 minutes
3. Check the CloudWatch Logs `/aws/lambda/smartdocai`
4. Try to log in again → should fail (the user has been deleted)

**Checking that the EventBridge rule is running:**
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

**Checking CloudWatch Logs:**
```bash
aws logs filter-log-events \
  --log-group-name /aws/lambda/smartdocai \
  --filter-pattern "Deleted.*users" \
  --start-time $(date -u -d '10 minutes ago' +%s)000 \
  --region us-east-1
```

**Expected log result:**
```
[Cleanup] Deleted 3 users: ['user1@example.com', 'user2@example.com', 'user3@example.com']
```

**Verifying the user was deleted in Cognito:**
```bash
aws cognito-idp list-users \
  --user-pool-id us-east-1_3oq5wIiuu \
  --filter "email = \"test@example.com\"" \
  --region us-east-1

# Expected: Empty list (user not found)
```

---
