---
title : "Kiến trúc tổng thể trên AWS"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.1.3 </b> "
---

Sau khi tìm hiểu riêng kiến trúc Frontend và Backend ở 2 phần trước, phần này tổng hợp lại thành **bức tranh toàn cảnh** cách các thành phần AWS phối hợp với nhau để tạo thành hệ thống SmartDocAI hoàn chỉnh, cùng với cấu trúc lưu trữ dữ liệu per-user.

### 1. Sơ đồ kiến trúc tổng thể

![Sơ đồ kiến trúc tổng thể](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/architecture-diagram.png)

**Chú thích luồng theo số thứ tự trong sơ đồ:**

| # | Luồng | # | Luồng |
|---|---|---|---|
| 1 | Users → CloudFront | 9 | GitHub Repository → CodePipeline (Backend) & (Frontend) |
| 2 | CloudFront → S3 Frontend Bucket | 10 | CodePipeline (Backend) → CodeBuild (pytest) |
| 3 | CloudFront → API Gateway (proxy `/api/*`) | 11 | CodeBuild (pytest) → Amazon ECR |
| 4 | API Gateway → Lambda | 12 | ECR → Lambda (deploy container mới) |
| 5 | Lambda → Cognito User Pool (validate JWT) | 13 | CodePipeline (Frontend) → AWS CodeBuild |
| 6 | Lambda → Data Storage (DynamoDB + S3) | 14 | AWS CodeBuild (Frontend) → S3 Frontend Bucket |
| 7 | Lambda → Amazon Bedrock (LLM + Embeddings) | 15 | Cognito ↔ Google Identity Provider (OAuth) |
| 8 | EventBridge → Lambda (cleanup định kỳ 5 phút) | 16 | Cognito ↔ Lambda presignup-check (merge account) |

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
| CI/CD | 2 CodePipeline riêng biệt | `smartdocai-be-pipeline` (Backend), `smartdocsai-fe-pipeline` (Frontend) |
| Tác vụ định kỳ | EventBridge | Rule dọn user chưa xác thực (rate 5 phút) |
| Giám sát | CloudWatch + SNS | Alarms cho Lambda Errors/Duration/Throttles + API Gateway 5xx, cảnh báo qua email (chi tiết xem mục 5.5.5) |
| Phân quyền & mã hóa | IAM + KMS | IAM Roles cho Lambda/CodeBuild, KMS key mã hóa DynamoDB |

### 2. Cấu trúc lưu trữ dữ liệu (Storage Structure)

![Cấu trúc lưu trữ dữ liệu](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/storage-structure.png)

Toàn bộ dữ liệu được thiết kế **cô lập theo từng user** (`user_id` = Cognito `sub`), tránh rò rỉ dữ liệu chéo giữa các tài khoản:

- `uploads/{user_id}/` — tài liệu gốc (PDF/DOCX) người dùng tải lên
- `vectorstore/{user_id}/` — FAISS index riêng (1024 chiều) để tìm kiếm ngữ nghĩa
- `chat_history/{user_id}.json` — lịch sử hội thoại RAG
- `processed_files/{user_id}.json` — danh sách tài liệu đã xử lý xong
- DynamoDB `smartdocai-user-profiles` (partition key `user_id`) — chỉ lưu `avatar_url` (các thông tin còn lại như họ tên, SĐT, ngày sinh đã lưu trực tiếp trong Cognito attributes để tránh 2 nơi lưu bị lệch dữ liệu)

### 3. Module hóa Backend (Lambda Modules)

![Module hóa Backend Lambda](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/lambda-modules.png)

`app_api.py` đóng vai trò entry point chính (FastAPI + Mangum adapter), điều hướng request tới các module chuyên biệt trong `modules/`: `auth_service.py` (xác thực), `document_processor.py` + `vector_store.py` (xử lý & lập chỉ mục tài liệu), `rag_chain.py` + `self_rag.py` + `co_rag.py` (3 chế độ RAG), `profile_service.py` (hồ sơ cá nhân).

### 4. CI/CD Pipeline

![CI/CD Pipeline](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/cicd-pipeline.png)

Hệ thống có **2 CodePipeline tách biệt**, cùng nhận code từ 1 GitHub repository:

- **CodePipeline (Backend)** `smartdocai-be-pipeline`: Mỗi lần push code lên nhánh `main`, tự động kích hoạt CodeBuild: cài dependencies → lint bằng flake8 → chạy pytest (hard-fail nếu test không qua) → build Docker image → đẩy lên ECR → cập nhật Lambda function. Cơ chế này đảm bảo code lỗi không thể lọt vào production.
- **CodePipeline (Frontend)** `smartdocsai-fe-pipeline`: gồm 3 stage **Source → Build → Deploy**. Stage Build dùng **AWS CodeBuild** (project `smartdocsai-fe-build`, chạy theo `buildspec.yml`) để cài dependencies và build ứng dụng React/Vite, đóng gói kết quả thành `smartdocsai-fe.zip`; stage Deploy sau đó đẩy thẳng file này lên S3 Frontend Bucket (tự động giải nén). Khi tạo pipeline, stage Test được chọn **Skip test stage** vì frontend chưa có bộ test tự động riêng, khác với Backend luôn bắt buộc pytest phải pass.
