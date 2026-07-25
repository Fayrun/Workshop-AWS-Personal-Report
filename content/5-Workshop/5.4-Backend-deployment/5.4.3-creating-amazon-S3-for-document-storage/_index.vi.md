---
title : "Tạo Amazon S3 lưu trữ tài liệu"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

### 1. Khai báo Tên Bucket và Region

![image5.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image5.png)

---

### 2. Chạy lệnh Tạo S3 Bucket

![image6.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image6.png)

---

### 3. Cấu hình CORS Security Hardening (Cho phép Frontend Tải/Tải lên tài liệu)

Để đảm bảo an toàn bảo mật dữ liệu và phòng chống tấn công Cross-Origin trái phép, chính sách S3 CORS không sử dụng ký tự đại diện `AllowedOrigins: ["*"]` mà giới hạn nghiêm ngặt theo danh sách tên miền tin cậy:
- CloudFront CDN Distribution: `https://dutf3c70nnjzl.cloudfront.net`
- Môi trường phát triển Local Dev: `http://localhost:5173` và `http://localhost:5174`

#### 3.1. Tệp cấu hình CORS Policy (`cors.json`):

```json
{
  "CORSRules": [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
      "AllowedOrigins": [
        "https://dutf3c70nnjzl.cloudfront.net",
        "http://localhost:5173",
        "http://localhost:5174"
      ],
      "ExposeHeaders": ["ETag"]
    }
  ]
}
```



#### 3.2. Chạy lệnh áp dụng CORS policy:

```bash
aws s3api put-bucket-cors --bucket smartdocai-storage-623035187993 --cors-configuration file://cors.json
```



### 4. Cấu hình Phân Quyền IAM Role cho AWS Lambda

Đảm bảo IAM Role chạy Lambda Function (ví dụ: `smartdocai-lambda-role`) được đính kèm Policy truy cập S3.

Chạy lệnh gán quyền `AmazonS3FullAccess` cho Lambda Role:

![image9.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image9.png)

**Kết quả:**

![image10.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image10.png)

---

### 5. Cấu Trúc Thư Mục Lưu Trữ Amazon S3 (Per-User Isolation)

Hệ thống lưu trữ tài liệu gốc, vector index và toàn bộ JSON metadata của người dùng trực tiếp trên S3 phân lập theo `user_id`:

![image11.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image11.png)

| STT | Thư mục (Prefix) | Loại tệp | Mô tả & Chức năng |
| --- | --- | --- | --- |
| 1 | `uploads/{user_id}/` | `.pdf`, `.docx` | Lưu trữ file tài liệu gốc mà người dùng tải lên từ Frontend qua S3 Presigned URL. |
| 2 | `vectorstore/{user_id}/smartdoc_index/` | `index.faiss`, `index.pkl` | Lưu chỉ mục Vector DB FAISS per-user đã chuyển đổi từ tài liệu, phục vụ RAG search. |
| 3 | `processed_files/` | `{user_id}.json` | Lưu danh sách tài liệu và metadata quản lý file per-user (thay thế DynamoDB) hiển thị ở Frontend. |
| 4 | `chat_history/` | `{user_id}.json` | Lưu toàn bộ lịch sử hỏi đáp RAG, câu trả lời AI và nguồn trích dẫn (`sources`) per-user (thay thế DynamoDB). |
| 5 | `avatars/{user_id}/` | `.jpg`, `.png` | Lưu ảnh đại diện của user khi dung lượng ảnh > 300KB (nếu < 300KB lưu base64 ở DynamoDB). |
| 6 | `metadata/` | `search_config.json` | Lưu cài đặt RAG chung (`chunk_size`, `chunk_overlap`, trạng thái Reranker/Self-RAG/Co-RAG). |
