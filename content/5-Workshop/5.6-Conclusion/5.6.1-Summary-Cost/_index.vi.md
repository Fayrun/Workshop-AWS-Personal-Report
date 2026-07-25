---
title : "Tổng kết Workshop & Chi phí"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

Xin chúc mừng! Bạn đã hoàn thành việc triển khai và kiểm thử hệ thống **SmartDocAI** trên AWS với kiến trúc serverless đầy đủ.

### Những gì đã học được

#### 1. **Kiến trúc Serverless & Vi dịch vụ**
- Deploy FastAPI application lên Lambda với Docker container
- Sử dụng API Gateway làm HTTP endpoint public
- Tận dụng CloudFront CDN cho static assets và API caching
- Thiết kế kiến trúc 3 tầng (Presentation → Application → Data)

#### 2. **Xác thực & Phân quyền**
- Tích hợp Cognito User Pool để quản lý người dùng
- Triển khai xác thực dựa trên JWT
- Hỗ trợ nhiều phương thức đăng nhập (Email/Password + Google OAuth)
- Cô lập dữ liệu theo từng user với S3 prefix + DynamoDB partition key

#### 3. **Tích hợp AI & Học máy**
- Sử dụng Amazon Bedrock (Qwen3-Next 80B-A3B LLM + Titan Embeddings V2)
- Xây dựng quy trình RAG (Retrieval-Augmented Generation)
- Tìm kiếm vector bằng FAISS (cơ sở dữ liệu trong bộ nhớ)
- Triển khai 3 chế độ RAG: Standard, Self-RAG, Co-RAG

#### 4. **Quy trình CI/CD**
- Thiết lập CodePipeline tự động kích hoạt khi có push lên GitHub
- Tích hợp unit test pytest vào CodeBuild (hard fail)
- Triển khai dựa trên Docker với ECR registry
- Tự động cập nhật Lambda function

#### 5. **Tự động hóa & Giám sát**
- EventBridge rule định kỳ cho tác vụ dọn dẹp
- CloudWatch Logs + Insights để giám sát ứng dụng
- Chỉ số Lambda (invocations, duration, errors, throttles)
- Xem log thời gian thực (tailing) và truy vấn
- **CloudWatch Alarms (4 alarms) + SNS Topic Alerting** — chủ động phát hiện lỗi/hiệu năng bất thường (Lambda Errors, Duration, Throttles, API Gateway 5xx) và gửi email cảnh báo qua SNS, thay vì chờ user report

#### 6. **Thực hành bảo mật tốt nhất**
- Validate dữ liệu đầu vào (phone, DOB, fullname, chống XSS)
- Giới hạn CORS (không dùng wildcard `*`)
- Chỉ dùng HTTPS (TLS 1.2+)
- JWT hết hạn + kiểm tra chữ ký
- Đánh giá bảo mật (security audit) kèm giới hạn đã ghi chú

#### 7. **Tối ưu chi phí**
- Mô hình trả tiền theo mức dùng (pay-per-use) với dịch vụ serverless
- DynamoDB tính phí on-demand (không tốn phí khi rảnh)
- S3 presigned URL (bỏ qua Lambda khi upload)
- CloudFront Free Tier (1 TB/tháng)
- **Chi phí ước tính:** $7-15/tháng cho mức sử dụng vừa phải

---

## Tổng chi phí

### Chi phí thực tế đã phát sinh (trong suốt thời gian workshop)

Giả sử workshop chạy trong **7 ngày** với mức độ test vừa phải:

| Dịch vụ | Mức sử dụng | Chi phí (7 ngày) | Chi phí ước tính (theo tháng) |
|---------|-------|------------------|---------------------------|
| Lambda | 5K invocations, 2GB RAM, 5s avg | $0.20 | $0.85 |
| API Gateway | 5K requests | $0.02 | $0.08 |
| S3 (Frontend) | 2 GB storage, 5 GB transfer | $0.10 | $0.35 |
| S3 (Storage) | 10 GB storage, 10 GB transfer | $0.35 | $1.20 |
| DynamoDB | 2K reads, 1K writes | $0.05 | $0.15 |
| Cognito | 50 users (MAU) | Free | Free |
| CloudFront | 20 GB transfer (Free tier) | $0.00 | $0.00 |
| Bedrock (LLM) | 200K input tokens, 50K output tokens | $0.13 | $0.45 |
| Bedrock (Embeddings) | 1M tokens | $0.10 | $0.35 |
| CodePipeline | 1 active pipeline | Free | Free |
| CodeBuild | 5 builds, 5 min each | $0.02 | $0.08 |
| ECR | 1.5 GB storage | $0.15 | $0.20 |
| EventBridge | 2016 events (every 5min × 7 days) | Free | Free |
| CloudWatch Logs | 0.5 GB ingested | $0.25 | $1.00 |
| **TOTAL** | | **~$1.37** | **~$4.71** |

**Ghi chú:**
- Chi phí workshop: **Dưới $2** (rất thấp)
- Free Tier: Cognito, EventBridge, CodePipeline, CloudFront (1 TB) đều miễn phí
- Lambda provisioned concurrency (tùy chọn): +$4/tháng nếu bật
- Chi phí production với traffic cao hơn: $15-50/tháng

---