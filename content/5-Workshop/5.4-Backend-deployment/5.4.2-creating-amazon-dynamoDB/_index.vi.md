---
title : "Tạo Amazon DynamoDB"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

### 1. Mô hình lưu trữ dữ liệu hệ thống SmartDocAI

Hệ thống SmartDocAI hiện tại tối ưu hóa kiến trúc lưu trữ bằng cách kết hợp giữa **Amazon DynamoDB** và **Amazon S3**:

- **Bảng DynamoDB (`smartdocai-user-profiles`)**: Dùng để quản lý thông tin hồ sơ người dùng mở rộng, cài đặt hệ thống và ảnh đại diện Avatar URL.
- **Quản lý Metadata trên Amazon S3 (Per-User S3 Isolation)**: Thay vì lưu trữ danh sách tài liệu và lịch sử hội thoại vào các bảng DynamoDB riêng lẻ (gây tốn kém chi phí Read/Write Capacity Unit và tăng độ phức tạp query), hệ thống quản lý trực tiếp dưới dạng các tệp JSON Metadata trên Amazon S3:
  - Danh sách tài liệu: `processed_files/{user_id}.json`
  - Lịch sử hội thoại RAG: `chat_history/{user_id}.json`

---

#### 1.1. Chi tiết Bảng DynamoDB : `smartdocai-user-profiles`

- **Partition Key (PK): `user_id` (String)** - Tương ứng với `sub` (UUID) do AWS Cognito cấp phát khi người dùng đăng ký hoặc đăng nhập Google.

- **Thuộc tính (Attributes):**
  - `email` (String): Địa chỉ email người dùng.
  - `fullname` (String): Họ và tên đầy đủ.
  - `phone` (String): Số điện thoại (chuẩn E.164: `+84...`).
  - `dob` (String): Ngày sinh (`YYYY-MM-DD`).
  - `avatar` (String, Optional): Chuỗi Base64 (nếu ảnh < 300KB).
  - `avatar_url` (String, Optional): Đường dẫn S3 URL công khai (nếu ảnh > 300KB).
  - `subscription_plan` (String): Gói dịch vụ (`free` / `pro`).
  - `document_quota` (Number): Giới hạn số tài liệu tối đa (Mặc định: 50).
  - `documents_used` (Number): Số lượng tài liệu đã tải lên hiện tại.
  - `user_preferences` (Map): Cấu hình ngôn ngữ & giao diện (`{ language: "vi", theme: "dark" }`).
  - `created_at` (Number): Timestamp khởi tạo bản ghi (Unix timestamp).
  - `updated_at` (Number): Timestamp cập nhật bản ghi gần nhất.

---

### 2. Khởi tạo tài nguyên DynamoDB trên AWS (thông qua AWS CLI)

#### 2.1. Mở Terminal / PowerShell đã cấu hình AWS CLI

![image23.png](/images/5-Workshop/5.4-Backend-deployment/5.4.2-creating-amazon-dynamoDB/image23.png)

#### 2.2. Tạo bảng `smartdocai-user-profiles`

Chạy lệnh AWS CLI tạo bảng DynamoDB với chế độ tính phí theo yêu cầu (`PAY_PER_REQUEST` / On-Demand):

```bash
aws dynamodb create-table \
    --table-name smartdocai-user-profiles \
    --attribute-definitions AttributeName=user_id,AttributeType=S \
    --key-schema AttributeName=user_id,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region us-east-1
```


**Kết quả khởi tạo bảng:**

![image27_tables1.png](/images/5-Workshop/5.4-Backend-deployment/5.4.2-creating-amazon-dynamoDB/image27_tables1.png)

---

### 3. Cấu hình phân quyền IAM Role cho AWS Lambda

Đảm bảo IAM Role của AWS Lambda (`smartdocai-lambda-role`) được đính kèm quyền `AmazonDynamoDBFullAccess` hoặc Inline Policy cho phép thao tác `GetItem`, `PutItem`, `UpdateItem` trên bảng `smartdocai-user-profiles`.

![image28.png](/images/5-Workshop/5.4-Backend-deployment/5.4.2-creating-amazon-dynamoDB/image28.png)
