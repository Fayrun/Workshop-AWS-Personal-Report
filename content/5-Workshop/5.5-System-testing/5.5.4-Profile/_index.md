---
title : "Profile Testing"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.5.4 </b> "
---

This section tests all 4 CRUD operations (Create/Read/Update/Delete) on the user profile: viewing information, updating personal information, uploading an avatar via presigned URL, and deleting the account (including cleaning up all related data in Cognito, DynamoDB, and S3).

### 1. Test Case Summary Table

| # | Operation | Endpoint | Input | Expected Result | Verification | Status |
|---|-----------|----------|-------|-----------------|--------------|--------|
| **READ** |
| 4.1 | Get user profile | GET /profile | JWT token | 200 OK<br/>Return: user_id, email, fullname, phone, dob, avatar_url, timestamps | Profile data matches Cognito attributes | PASS |
| 4.2 | Get profile without JWT | GET /profile | No token | 401 Unauthorized<br/>Error: "Missing Authorization header" | - | PASS |
| **UPDATE** |
| 4.3 | Update fullname + phone | PUT /profile | JWT + fullname: "Nguyen Van Updated"<br/>+ phone: "0987654321" | 200 OK<br/>Return: updated_fields, new updated_at timestamp | DynamoDB record updated | PASS |
| 4.4 | Update with invalid phone | PUT /profile | phone: "123456" (invalid format) | 400 Bad Request<br/>Error: "Invalid phone format" | No changes to DynamoDB | PASS |
| 4.5 | Update DOB | PUT /profile | dob: "1995-06-20" | 200 OK | Cognito custom:dob + DynamoDB updated | PASS |
| **AVATAR UPLOAD** |
| 4.6 | Get avatar presigned URL | GET /profile/avatar-upload-url | JWT token | 200 OK<br/>Return: upload_url (1h expiry) to S3 path: users/{id}/avatar/ | - | PASS |
| 4.7 | Upload avatar to S3 | PUT to presigned URL | avatar.jpg + Content-Type: image/jpeg | 200 OK (direct S3 upload) | File appears in S3 bucket | PASS |
| 4.8 | Verify avatar_url updated | GET /profile | JWT token (after upload) | Profile contains avatar_url: "https://s3.../users/{id}/avatar/avatar.jpg" | - | PASS |
| **DELETE** |
| 4.9 | Delete account | DELETE /profile | JWT + confirm: true | 200 OK<br/>Return: deleted_items (cognito_user, dynamodb_profile, s3_documents count, s3_avatar) | All resources cleaned up | PASS |
| 4.10 | Verify Cognito user deleted | AWS CLI: list-users | Filter by email | Empty list (user not found) | - | PASS |
| 4.11 | Verify DynamoDB record deleted | AWS CLI: get-item | Key: user_id | Empty Item | - | PASS |
| 4.12 | Verify S3 objects deleted | AWS CLI: s3 ls | Path: uploads/{user_id}/, vectorstore/{user_id}/ | Empty (no objects) | - | PASS |
| 4.13 | Verify FAISS cleared | POST /api/chat after deletion | JWT token (new user) | No vectors from the old user remain | Per-user isolation works correctly | PASS |

**API endpoint:** `https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod/api/profile`

**Profile data schema (DynamoDB + Cognito):**
```json
{
  "user_id": "uuid (from Cognito sub)",
  "email": "test@example.com",
  "fullname": "Nguyen Van Test",
  "phone": "+84901234567",
  "dob": "1990-01-15",
  "avatar_url": "https://s3.../users/{id}/avatar/pic.jpg",
  "created_at": 1234567890,
  "updated_at": 1234567890
}
```

**Cleanup steps when deleting an account:**
1. Delete the user in Cognito (`admin_delete_user`)
2. Delete the profile record in DynamoDB
3. Delete all S3 objects under `uploads/{user_id}/`, `vectorstore/{user_id}/`, `avatars/{user_id}.jpg`
4. Delete the user's FAISS vector data (in-memory)

---
