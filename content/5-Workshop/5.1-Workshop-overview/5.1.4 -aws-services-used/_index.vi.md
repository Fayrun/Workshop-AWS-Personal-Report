---
title : "Các dịch vụ AWS được sử dụng"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.1.4 </b> "
---

Phần này liệt kê chi tiết từng dịch vụ AWS mà SmartDocAI sử dụng, cấu hình cụ thể và lý do lựa chọn — giúp người đọc hiểu rõ vai trò của mỗi thành phần trong kiến trúc tổng thể đã trình bày ở phần trước.

### 1. Compute & API

| Dịch vụ | Cấu hình | Vai trò |
|---|---|---|
| **AWS Lambda** | Function `smartdocai`, Docker container image, region `us-east-1` | Chạy toàn bộ backend FastAPI (auth, upload, RAG, profile) theo mô hình serverless — chỉ tính phí khi có request, tự động scale |
| **Amazon API Gateway** | HTTP API, stage `/prod`, endpoint `d60866voq5.execute-api...` | Cổng vào duy nhất cho mọi request từ Frontend, proxy trực tiếp sang Lambda |
| **Amazon ECR** | Repository `smartdocai`, Lifecycle Policy (xóa image untagged sau 1 ngày, giữ tối đa 5 image tagged) | Lưu trữ Docker image của Lambda function, được CodePipeline tự động cập nhật mỗi lần deploy |

### 2. Storage

| Dịch vụ | Cấu hình | Vai trò |
|---|---|---|
| **Amazon S3 (Application Storage)** | Bucket `smartdocai-storage-623035187993`, Intelligent-Tiering, SSE-S3 mặc định | Lưu tài liệu người dùng, FAISS vector index, chat history, avatar — cô lập theo `user_id` |
| **Amazon S3 (Frontend Hosting)** | Bucket `aws-smartdocsai-cloud`, Static Website Hosting | Lưu file build của React SPA (`index.html`, `assets/`), làm origin cho CloudFront |
| **Amazon DynamoDB** | Table `smartdocai-user-profiles`, Partition Key `user_id`, PAY_PER_REQUEST, SSE-KMS | Lưu duy nhất field `avatar_url` — các thông tin cá nhân khác đã lưu trực tiếp trong Cognito attributes |

### 3. Identity & Security

| Dịch vụ | Cấu hình | Vai trò |
|---|---|---|
| **Amazon Cognito User Pool** | `us-east-1_3oq5wIiuu`, hỗ trợ Email/Password + Google OAuth | Quản lý danh tính, cấp JWT token, có sẵn cơ chế throttling/lockout chống brute-force |
| **Cognito Hosted UI** | Domain `smartdocai-fayrun2026.auth.us-east-1.amazoncognito.com` | Màn hình trung gian xử lý luồng OAuth với Google, có `state` parameter chống CSRF (tự triển khai thêm ở tầng ứng dụng) |
| **Lambda PreSignUp Trigger** | Function `smartdocai-presignup-check` | Tự động liên kết (`AdminLinkProviderForUser`) tài khoản Google với tài khoản email/password đã tồn tại cùng email, tránh tạo 2 profile trùng lặp |
| **AWS KMS** | Key `alias/aws/dynamodb` | Mã hóa dữ liệu trong DynamoDB, cho phép audit trail qua CloudTrail |

### 4. AI & Machine Learning

| Dịch vụ | Model cụ thể | Vai trò |
|---|---|---|
| **Amazon Bedrock (LLM)** | `qwen.qwen3-next-80b-a3b` | Sinh câu trả lời cho 3 chế độ RAG (Standard, Self-RAG, Co-RAG) |
| **Amazon Bedrock (Embeddings)** | `amazon.titan-embed-text-v2:0` (1024 chiều) | Chuyển văn bản thành vector để lập chỉ mục và tìm kiếm ngữ nghĩa, chạy song song 12 threads khi xử lý tài liệu lớn |

> Xem ảnh `system-status-live.png` ở mục 5.1.3 — response thực tế đã xác nhận cả 2 model đang hoạt động: `"provider":"Amazon Bedrock","model":"qwen.qwen3-next-80b-a3b","embedding_provider":"Amazon Titan"`.

### 5. CDN & Delivery

| Dịch vụ | Cấu hình | Vai trò |
|---|---|---|
| **Amazon CloudFront** | Distribution `dutf3c70nnjzl.cloudfront.net`, origin = S3 frontend bucket | Phân phối React SPA qua mạng CDN toàn cầu, giảm độ trễ, HTTPS enforce TLS 1.2+ |

### 6. CI/CD

| Dịch vụ | Cấu hình | Vai trò |
|---|---|---|
| **AWS CodePipeline** | `smartdocai-be-pipeline` (backend), `smartdocsai-fe-pipeline` (frontend) | Tự động trigger khi push code lên GitHub `main`, điều phối Source → Build → Deploy |
| **AWS CodeBuild** | Project `smartdocai-be-build` | Cài dependencies, lint (flake8), chạy pytest (hard-fail), build Docker image, push ECR |
| **S3 (Artifact Store)** | Bucket `codepipeline-us-east-1-...` (tự động tạo bởi AWS) | Lưu artifact trung gian giữa các stage của cả 2 pipeline |

### 7. Automation & Monitoring

| Dịch vụ | Cấu hình | Vai trò |
|---|---|---|
| **Amazon EventBridge** | Rule `smartdocai-cleanup-unconfirmed`, rate 5 phút | Tự động kích hoạt Lambda để xóa các tài khoản Cognito chưa xác thực email sau 5 phút, tránh chiếm giữ slot username |
| **Amazon CloudWatch Logs** | Log group `/aws/lambda/smartdocai` | Ghi log thực thi của Lambda, hỗ trợ debug qua Logs Insights query |
| **Amazon CloudWatch Alarms** | 4 alarms: Lambda Errors, Duration, Throttles, API Gateway 5xx | Chủ động phát hiện lỗi/hiệu năng bất thường, gửi cảnh báo qua SNS thay vì chờ user report |
| **Amazon SNS** | Topic `smartdocai-alerts` | Gửi email cảnh báo khi CloudWatch Alarm chuyển sang trạng thái ALARM |

### 8. Bảng tổng hợp theo tầng kiến trúc

| Tầng | Dịch vụ AWS sử dụng |
|---|---|
| **Presentation** | S3 (Frontend), CloudFront |
| **Application** | API Gateway, Lambda, ECR, EventBridge |
| **Data** | S3 (Storage), DynamoDB, Cognito |
| **AI** | Bedrock (LLM + Embeddings) |
| **CI/CD** | CodePipeline, CodeBuild, S3 (Artifact) |
| **Security & Monitoring** | KMS, CloudWatch (Logs + Alarms), SNS |
