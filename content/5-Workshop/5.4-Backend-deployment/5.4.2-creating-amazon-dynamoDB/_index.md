---
title : "Creating Amazon DynamoDB"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

### 1. SmartDocAI System Data Storage Model

The current SmartDocAI system optimizes storage architecture by combining **Amazon DynamoDB** and **Amazon S3**:

- **Single DynamoDB Table (`smartdocai-user-profiles`)**: Used to manage extended user profile data, system preferences, and avatar URLs.
- **Amazon S3 Metadata Management (Per-User S3 Isolation)**: Instead of storing document lists and chat histories in separate DynamoDB tables (which increases Read/Write Capacity Unit costs and query complexity), the system directly manages them as JSON Metadata files stored on Amazon S3:
  - Document List: `processed_files/{user_id}.json`
  - RAG Chat History: `chat_history/{user_id}.json`

---

#### 1.1. Single DynamoDB Table Details: `smartdocai-user-profiles`

- **Partition Key (PK): `user_id` (String)** - Corresponds to `sub` (UUID) issued by AWS Cognito upon user registration or Google OAuth sign-in.

- **Attributes:**
  - `email` (String): User email address.
  - `fullname` (String): Full name.
  - `phone` (String): Phone number (E.164 format: `+84...`).
  - `dob` (String): Date of birth (`YYYY-MM-DD`).
  - `avatar` (String, Optional): Base64 encoded string (if image size < 300KB).
  - `avatar_url` (String, Optional): Public S3 URL path (if image size > 300KB).
  - `subscription_plan` (String): Service tier (`free` / `pro`).
  - `document_quota` (Number): Maximum document upload limit (Default: 50).
  - `documents_used` (Number): Current count of uploaded documents.
  - `user_preferences` (Map): Language & theme settings (`{ language: "vi", theme: "dark" }`).
  - `created_at` (Number): Record creation Unix timestamp.
  - `updated_at` (Number): Record last updated Unix timestamp.

---

### 2. Initialize DynamoDB Resources on AWS (via AWS CLI)

#### 2.1. Open Terminal / PowerShell with configured AWS CLI

![image23.png](/images/5-Workshop/5.4-Backend-deployment/5.4.2-creating-amazon-dynamoDB/image23.png)

#### 2.2. Create table `smartdocai-user-profiles`

Run AWS CLI command to create the DynamoDB table with On-Demand billing mode (`PAY_PER_REQUEST`):

```bash
aws dynamodb create-table \
    --table-name smartdocai-user-profiles \
    --attribute-definitions AttributeName=user_id,AttributeType=S \
    --key-schema AttributeName=user_id,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region us-east-1
```

**Table Creation Result:**

![image27_tables1.png](/images/5-Workshop/5.4-Backend-deployment/5.4.2-creating-amazon-dynamoDB/image27_tables1.png)

---

### 3. Configure IAM Role Permissions for AWS Lambda

Ensure AWS Lambda's IAM Role (`smartdocai-lambda-role`) has attached `AmazonDynamoDBFullAccess` or inline policies permitting `GetItem`, `PutItem`, and `UpdateItem` operations on the `smartdocai-user-profiles` table.

![image28.png](/images/5-Workshop/5.4-Backend-deployment/5.4.2-creating-amazon-dynamoDB/image28.png)
