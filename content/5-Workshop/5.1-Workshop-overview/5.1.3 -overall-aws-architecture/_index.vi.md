---
title : "Kiến trúc tổng thể trên AWS"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.1.3 </b> "
---

Sau khi tìm hiểu riêng kiến trúc Frontend và Backend ở 2 phần trước, phần này tổng hợp lại thành **bức tranh toàn cảnh** cách các thành phần AWS phối hợp với nhau để tạo thành hệ thống SmartDocAI hoàn chỉnh, cùng với cấu trúc lưu trữ dữ liệu per-user.

### 1. Sơ đồ kiến trúc tổng thể

<img src="/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/architecture-diagram.png" width="100%" style="max-width:1100px">

**Chú thích luồng theo số thứ tự trong sơ đồ:**

| # | Luồng | # | Luồng |
|---|---|---|---|
| 1 | Users → CloudFront | 10 | CodePipeline → CodeBuild |
| 2 | CloudFront → S3 Frontend Bucket | 11 | CodeBuild → Amazon ECR |
| 3 | CloudFront → API Gateway (proxy `/api/*`) | 12 | ECR → Lambda (deploy container mới) |
| 4 | API Gateway → Lambda | 13 | Lambda → CloudWatch (ghi log) |
| 5 | Lambda → Cognito User Pool (validate JWT) | 14 | CloudWatch → SNS Topic |
| 6 | Lambda → Data Storage (DynamoDB + S3) | 15 | Cognito ↔ Google Identity Provider (OAuth) |
| 7 | Lambda → Amazon Bedrock (LLM + Embeddings) | 16 | Cognito ↔ Lambda presignup-check (merge account) |
| 8 | EventBridge → Lambda (cleanup định kỳ 5 phút) | 17 | API Gateway → CloudWatch (metric 5xxError) |
| 9 | GitHub Repository → CodePipeline | 18 | SNS Topic → Email Notification (Admin) |

> Đường 12 (tím), 13 (cam), 17 (xanh dương) đi theo 3 kênh riêng biệt trong ảnh để tránh chồng chéo — đây là các kết nối "xuyên băng" giữa luồng request runtime và phần vận hành nền (CI/CD + Monitoring).

SmartDocAI được xây dựng theo mô hình **Serverless Container Architecture** kết hợp **Managed Identity (Cognito)**, gồm các thành phần chính:

| Thành phần | Dịch vụ AWS | Giá trị cụ thể |
|---|---|---|
| Frontend hosting | S3 + CloudFront | `https://dutf3c70nnjzl.cloudfront.net` |
| Backend API | Lambda + API Gateway | `https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod` |
| Xác thực | Cognito User Pool | `us-east-1_3oq5wIiuu` |
| Cognito Hosted UI | Cognito | `https://smartdocai-fayrun2026.auth.us-east-1.amazoncognito.com` |
| PreSignUp trigger | Lambda | `smartdocai-presignup-check` |
| Profile database | DynamoDB | `smartdocai-user-profiles` (SSE-KMS) |
| File & Index storage | S3 | `smartdocai-storage-623035187993` (Intelligent-Tiering) |
| LLM | Bedrock | `qwen.qwen3-next-80b-a3b` |
| Embeddings | Bedrock | `amazon.titan-embed-text-v2:0` (1024 chiều) |
| CI/CD | CodePipeline | `smartdocai-be-pipeline`, `smartdocsai-fe-pipeline` |
| Tác vụ định kỳ | EventBridge | Rule dọn user chưa xác thực (rate 5 phút) |
| Giám sát | CloudWatch | Alarms cho Lambda Errors/Duration/Throttles + API Gateway 5xx |

### 2. Cấu trúc lưu trữ dữ liệu (Storage Structure)

<img src="/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/storage-structure.png" width="90%" style="max-width:900px">

Toàn bộ dữ liệu được thiết kế **cô lập theo từng user** (`user_id` = Cognito `sub`), tránh rò rỉ dữ liệu chéo giữa các tài khoản:

- `uploads/{user_id}/` — tài liệu gốc (PDF/DOCX) người dùng tải lên
- `vectorstore/{user_id}/` — FAISS index riêng (1024 chiều) để tìm kiếm ngữ nghĩa
- `chat_history/{user_id}.json` — lịch sử hội thoại RAG
- `processed_files/{user_id}.json` — danh sách tài liệu đã xử lý xong
- DynamoDB `smartdocai-user-profiles` (partition key `user_id`) — chỉ lưu `avatar_url` (các thông tin còn lại như họ tên, SĐT, ngày sinh đã lưu trực tiếp trong Cognito attributes để tránh 2 nơi lưu bị lệch dữ liệu)

### 3. Module hóa Backend (Lambda Modules)

<img src="/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/lambda-modules.png" width="90%" style="max-width:900px">

`app_api.py` đóng vai trò entry point chính (FastAPI + Mangum adapter), điều hướng request tới các module chuyên biệt trong `modules/`: `auth_service.py` (xác thực), `document_processor.py` + `vector_store.py` (xử lý & lập chỉ mục tài liệu), `rag_chain.py` + `self_rag.py` + `co_rag.py` (3 chế độ RAG), `profile_service.py` (hồ sơ cá nhân).

### 4. CI/CD Pipeline

<img src="/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/cicd-pipeline.png" width="100%" style="max-width:1100px">

Mỗi lần push code lên nhánh `main` trên GitHub, CodePipeline tự động kích hoạt CodeBuild: cài dependencies → lint bằng flake8 → chạy pytest (hard-fail nếu test không qua) → build Docker image → đẩy lên ECR → cập nhật Lambda function. Cơ chế này đảm bảo code lỗi không thể lọt vào production.
