---
title : "Tổng kết Workshop & Chi phí"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

### Những gì đã thực hiện

#### 1. **5.1 - Giới thiệu & Kiến trúc**
- Đặc tả kiến trúc Frontend, kiến trúc Backend & RAG Pipeline
- Vẽ sơ đồ kiến trúc tổng thể trên AWS
- Liệt kê đầy đủ các dịch vụ AWS sử dụng
- Mô tả giao diện và chức năng hệ thống

#### 2. **5.2 - Chuẩn bị**
- Chuẩn bị mã nguồn dự án
- Tạo và cấu hình tài khoản AWS
- Tạo IAM User riêng để quản lý quyền truy cập thay vì dùng root user

#### 3. **5.3 - Triển khai Frontend**
- Tạo S3 Bucket lưu trữ file tĩnh
- Bật Static Website Hosting
- Cấu hình Bucket Policy/Block Public Access
- Tăng tốc truy cập bằng CloudFront
- Tự động hóa deploy bằng CodePipeline (GitHub → S3)

#### 4. **5.4 - Triển khai Backend**
- Khởi tạo Cognito (xác thực người dùng)
- Khởi tạo DynamoDB (hồ sơ người dùng)
- Khởi tạo S3 (lưu tài liệu + FAISS index)
- Đóng gói RAG Engine chạy trên Lambda (Docker container qua ECR)
- Tạo API Gateway
- Tích hợp Frontend gọi API qua CloudFront
- Tự động hóa CI/CD backend bằng CodePipeline

#### 5. **5.5 - Kiểm thử hệ thống**
- Authentication (đăng ký/đăng nhập/JWT)
- Document Upload & RAG (3 chế độ Standard/Self-RAG/Co-RAG)
- Security (validate đầu vào, CORS/JWT, CSRF OAuth state)
- Profile (CRUD hồ sơ + avatar)
- Monitoring (CloudWatch Logs/Insights/Alarms + SNS)
- CI/CD (pytest tự động, cơ chế hard-fail)

### Những gì đã học được

#### 1. **Kiến trúc Serverless & Vi dịch vụ (Microservices)**
- Deploy FastAPI application lên Lambda với Docker container
- Sử dụng API Gateway làm HTTP endpoint public
- Tận dụng CloudFront CDN cho static assets và API caching

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
- **Chi phí thực tế:** ~$1.65 trong 30 ngày phát triển (27/06 - 26/07/2026) — xem chi tiết bên dưới

---

## Tổng chi phí

### Chi phí thực tế (theo AWS Cost Explorer, 27/06/2026 - 26/07/2026, 30 ngày)

Số liệu dưới đây lấy trực tiếp từ AWS Cost Explorer, tổng hợp đúng **30 ngày** trong giai đoạn phát triển + kiểm thử workshop:

| Dịch vụ | Chi phí thực tế (USD) | Tỷ trọng |
|---------|------------------------|----------|
| Amazon ECR (lưu trữ Docker image) | $0.5977 | 36.3% |
| AWS CodePipeline | $0.4440 | 26.9% |
| Amazon Bedrock (LLM + Embeddings) | $0.3453 | 20.9% |
| AWS CodeBuild | $0.2250 | 13.7% |
| Amazon S3 | $0.0197 | 1.2% |
| Amazon API Gateway | $0.0140 | 0.9% |
| Amazon DynamoDB | $0.0021 | 0.1% |
| Amazon SES | $0.0003 | <0.1% |
| AWS Secrets Manager | $0.00002 | <0.1% |
| Amazon CloudFront | $0.0000052 | ~0% |
| AWS Lambda | $0.00 (Free Tier) | 0% |
| Amazon Cognito | $0.00 (Free Tier) | 0% |
| Amazon EventBridge | $0.00 (Free Tier) | 0% |
| Amazon CloudWatch | $0.00 (Free Tier) | 0% |
| Amazon SNS | $0.00 (Free Tier) | 0% |
| **TỔNG (30 ngày)** | **~$1.6481** | 100% |

### So sánh với chi phí dự tính ở mục 2 - Đề xuất

Ở [mục 2 - Đề xuất](/2-proposal/), nhóm đã ước tính tổng chi phí hạ tầng hàng tháng vào khoảng **$0.66 - $1.85 USD/tháng**. Chi phí thực tế trong đúng 30 ngày là **~$1.65/tháng** — **nằm trong khoảng đã dự tính ban đầu**, cho thấy ước tính tổng thể khá chính xác.

Tuy nhiên, khi so sánh chi tiết theo từng dịch vụ, cơ cấu chi phí thực tế lệch khá nhiều so với giả định ban đầu:

| Dịch vụ | Ước tính (mục 2) | Thực tế | Nhận xét |
|---------|-------------------|---------|----------|
| **Amazon ECR** | $0.10 - $0.20 | $0.5977 | **Khoản chi lớn nhất thực tế**, cao hơn ước tính đáng kể do đang trong giai đoạn phát triển với tần suất build/push liên tục, chưa phải giai đoạn vận hành ổn định |
| **CodePipeline & CodeBuild** | $0.05 - $0.15 | $0.6690 | Cao hơn ước tính **4-13 lần**, do giai đoạn phát triển có rất nhiều lần push/sửa lỗi/build lại (không giống giả định "~30 phút build/tháng" của hệ thống đã ổn định) |
| **Amazon Bedrock** | $0.25 - $0.95 | $0.3453 | Nằm trong khoảng ước tính |
| **Amazon S3** | $0.15 - $0.30 | $0.0197 | Thấp hơn nhiều so với ước tính — dung lượng tài liệu test thực tế còn ít |
| **API Gateway** | $0.01 - $0.05 | $0.0140 | Nằm trong khoảng ước tính |
| **DynamoDB** | $0.00 (Free Tier) | $0.0021 | Gần như khớp, vẫn gần như miễn phí |
| **Lambda, Cognito, CloudFront, EventBridge, CloudWatch** | $0.00 - $0.10 | $0.00 | Free Tier phát huy đúng như kỳ vọng |
| **Amazon SES, Secrets Manager** | *(không có trong bảng ước tính)* | $0.0003 + $0.00004 | Phát sinh nhỏ, không đáng kể, nhưng cho thấy có dùng thêm 2 dịch vụ ngoài kiến trúc đã liệt kê ban đầu |

**Kết luận:** Tổng chi phí thực tế khớp khá sát với ước tính ban đầu, nhưng **CI/CD (CodePipeline + CodeBuild + ECR) mới là nhóm chi phí lớn nhất trong thực tế (~77% tổng chi phí)** thay vì Bedrock/S3 như dự đoán ban đầu — điều này hợp lý vì đây là giai đoạn phát triển tích cực (nhiều lần build/deploy), không phải giai đoạn vận hành ổn định.

---

**Ghi chú:**
- Chi phí thực tế trong giai đoạn phát triển/workshop: **dưới $1.7** (rất thấp)
- Free Tier: Lambda, Cognito, EventBridge, CloudWatch, SNS đều đạt $0.00 nhờ mức sử dụng nằm trong hạn mức miễn phí
- Nhóm chi phí lớn nhất thực tế: CI/CD pipeline (ECR + CodePipeline + CodeBuild), không phải AI/Bedrock như dự đoán ban đầu
- Chi phí production với traffic cao hơn và nhiều lần build/deploy hơn: có thể cao hơn đáng kể, cần theo dõi qua CloudWatch Budget Alerts


---