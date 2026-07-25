---
title: "Bản đề xuất"
date: 2026-07-23
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# SMARTDOCAI - NỀN TẢNG HỎI ĐÁP & TRÍCH XUẤT TRI THỨC TÀI LIỆU THÔNG MINH TRÊN AWS SERVERLESS

---

### 1. TỔNG QUAN DỰ ÁN

**SmartDocAI** là giải pháp nền tảng web thông minh tích hợp Trợ lý AI (AI Assistant) dựa trên kiến trúc **AWS Serverless Container Architecture** và các mô hình Trí tuệ Nhân tạo Tạo hình (**Generative AI / Large Language Models**) tiên tiến trên **AWS Bedrock**.

Dự án được xây dựng nhằm giải quyết nhu cầu tìm kiếm, trích xuất và tổng hợp thông tin tự động từ các tài liệu doanh nghiệp/cá nhân dạng văn bản (PDF, DOCX). Hệ thống áp dụng kỹ thuật **Retrieval-Augmented Generation (RAG)** với 3 chế độ xử lý linh hoạt (Standard RAG, Self-RAG, Co-RAG Multi-Agent), đảm bảo câu trả lời có độ chính xác cao, trích dẫn rõ ràng và loại bỏ hiện tượng ảo giác (hallucination) từ LLM.

- **Tên dự án:** SmartDocAI (AWS Deployment)
- **Kiến trúc chính:** Serverless Container Architecture (FastAPI + React SPA + AWS Services)
- **Repository:** https://github.com/TakunKenjo/SmartdocAI-AWS
- **Môi trường triển khai:** AWS Region `us-east-1` (Account ID: `623035187993`)
- **Domain Production:** 
  - Frontend CDN (CloudFront): `https://dutf3c70nnjzl.cloudfront.net`
  - Backend API Gateway: `https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod`

---

### 2. MỤC TIÊU DỰ ÁN

1. **Xây dựng Hệ thống RAG Serverless Hoàn chỉnh (100% Serverless):**
   - Vận hành hoàn toàn trên hạ tầng Serverless (AWS Lambda, S3, DynamoDB, Bedrock, API Gateway, CloudFront) giúp hệ thống tự động co giãn theo lưu lượng (Auto-scaling) và đạt mức chi phí tối ưu nhất (Pay-as-you-go).

2. **Tối ưu hóa Khả năng Hỏi đáp Tri thức (Advanced RAG):**
   - Hỗ trợ 3 chế độ hỏi đáp nâng cao:
     - **Standard RAG:** Phân tích ngữ nghĩa nhanh chóng qua FAISS vector store.
     - **Self-RAG:** Tự động tái cấu trúc câu hỏi, lọc nhiễu văn bản và tự chấm điểm độ chính xác (grounding check) trước khi phản hồi.
     - **Co-RAG (Multi-Agent RAG):** Phối hợp song song 3 Agent độc lập (Semantic, Keyword, Conceptual) và hợp nhất kết quả theo cơ chế bỏ phiếu (voting), nâng cao chất lượng câu trả lời đối với câu hỏi phức tạp.

3. **Bảo mật & Phân lập Dữ liệu Tuyệt đối (Per-User Isolation):**
   - Đảm bảo mỗi người dùng có không gian lưu trữ tài liệu riêng (`uploads/{user_id}/`), bộ chỉ mục vector riêng (`vectorstore/{user_id}/`) và lịch sử hội thoại riêng.
   - Chuẩn hóa xác thực với **AWS Cognito User Pool**, hỗ trợ đăng nhập Email/Password native và Google OAuth 2.0. Tích hợp PreSignUp Lambda trigger tự động hợp nhất tài khoản trùng email (`AdminLinkProviderForUser`) và bảo vệ chống tấn công CSRF OAuth với `state` UUID parameter.

4. **Tự động hóa CI/CD & Production Hardening:**
   - Xây dựng pipeline tự động kiểm thử và triển khai với **AWS CodePipeline** & **AWS CodeBuild** (tích hợp pytest suite ~60 test cases với cơ chế hard-fail).
   - Tối ưu chi phí và giám sát vận hành: S3 Intelligent-Tiering, DynamoDB KMS Encryption at-rest, EventBridge Auto-Cleanup người dùng chưa xác thực, CloudWatch Alarms & SNS Alerting.

---

### 3. VẤN ĐỀ CẦN GIẢI QUYẾT

#### 3.1. Thực trạng & Thách thức

- **Tìm kiếm tri thức thủ công tốn thời gian:** Trong các doanh nghiệp và tổ chức nghiên cứu, việc tra cứu thủ công qua hàng ngàn trang tài liệu PDF/Word ngốn nhiều thời gian và dễ bỏ sót thông tin quan trọng.
- **Giới hạn của các mô hình LLM truyền thống:** Các công cụ chat AI thông thường không có quyền truy cập vào kho tài liệu nội bộ, đồng thời thường xuyên gặp tình trạng câu trả lời bị sai lệch (hallucination) do không có nguồn đối chiếu.
- **Rủi ro rò rỉ dữ liệu (Data Privacy & Isolation):** Nhiều dịch vụ SaaS bên thứ ba không cam kết phân lập dữ liệu người dùng, gây nguy cơ rò rỉ thông tin nhạy cảm.
- **Chi phí hạ tầng máy chủ 24/7 đắt đỏ:** Triển khai các cụm server GPU/EC2 cố định tốn kém chi phí duy trì lớn ngay cả khi không có lượt truy cập.
- **Nghẽn tải payload khi upload file lớn:** Các dịch vụ API Gateway truyền thống thường bị giới hạn kích thước request (10 MB), gây thất bại khi người dùng tải lên tài liệu dung lượng lớn.

#### 3.2. Giải pháp của SmartDocAI

- **Sử dụng RAG trên nền tảng AWS Bedrock:** Kết hợp **Amazon Titan Embeddings V2** (1024 chiều) và **Qwen 3 Next 80B A3B (`qwen.qwen3-next-80b-a3b`)** để tổng hợp tri thức chính xác kèm dẫn chứng vị trí trang/dòng cụ thể.
- **Cơ chế Upload S3 Presigned URL 3 bước:** Trình duyệt tải file trực tiếp lên S3 bucket bỏ qua API Gateway, hỗ trợ file tài liệu dung lượng lên tới **5 GB**.
- **Mô hình Phân lập Per-User Isolation:** Sử dụng Cognito `sub` (User ID) làm chìa khóa phân lập đường dẫn S3 và partition key DynamoDB, ngăn chặn triệt để nguy cơ truy cập chéo dữ liệu.
- **Kiến trúc Serverless tối ưu chi phí:** Loại bỏ chi phí duy trì máy chủ rảnh rỗi, chi phí vận hành hàng tháng chỉ phát sinh dựa trên lưu lượng thực tế.

---

### 4. KIẾN TRÚC GIẢI PHÁP

Hệ thống được thiết kế theo mô hình **Serverless Container Architecture** trên AWS với sơ đồ luồng hoạt động tổng quan như sau:

```text
+-----------------------------------------------------------------------------------------+
|                                TRÌNH DUYỆT NGƯỜI DÙNG                                    |
|                         (React + Vite SPA trên AWS CloudFront)                          |
+--------------------------------------------+--------------------------------------------+
                                             |
                                             v
+-----------------------------------------------------------------------------------------+
|                              XÁC THỰC & ĐĂNG NHẬP (AUTH)                                 |
|            AWS Cognito User Pool <---> Hosted UI (Google OAuth 2.0)                     |
|                                            |                                            |
|                  PreSignUp Lambda Trigger (Liên kết tài khoản Google & Native)          |
+--------------------------------------------+--------------------------------------------+
                                             |
                              Xác thực JWT Token thành công
                                             |
                                             v
+-----------------------------------------------------------------------------------------+
|                               API & COMPUTE LAYER (BACKEND)                             |
|                           Amazon API Gateway (HTTP REST Endpoint)                       |
|                                            |                                            |
|                     AWS Lambda Function: smartdocai (FastAPI Container)                 |
+------+---------------------+-----------------------+----------------------+-------------+
       |                     |                       |                      |
       v                     v                       v                      v
+--------------+     +---------------+     +------------------+     +-------------------+
|  AWS BEDROCK |     |   AMAZON S3   |     |  AMAZON DYNAMODB |     |   DEVOPS & ALERT  |
|  - Titan V2  |     | - Upload File |     | - User Profiles  |     | - CodePipeline    |
|  - Qwen3 80B |     | - FAISS Index |     | - KMS Encrypted  |     | - EventBridge     |
|  - FAISS RAG |     | - Chat Log    |     |   At-Rest        |     | - CloudWatch Alarms|
+--------------+     +---------------+     +------------------+     +-------------------+
```

#### Các thành phần chính trong kiến trúc:

1. **Frontend Layer:**
   - **React + Vite SPA:** Giao diện người dùng hiện đại, phản hồi nhanh.
   - **AWS CloudFront (CDN):** Phân phối ứng dụng React toàn cầu qua HTTPS, tối ưu tốc độ tải trang (`https://dutf3c70nnjzl.cloudfront.net`).

2. **Identity Layer:**
   - **Amazon Cognito User Pool (`us-east-1_3oq5wIiuu`):** Quản lý đăng ký, đăng nhập, cấp phát JWT (ID & Access Tokens).
   - **Cognito Hosted UI & Google Identity Provider:** Cho phép đăng nhập bằng Google OAuth 2.0.
   - **PreSignUp Lambda Trigger (`smartdocai-presignup-check`):** Tự động liên kết tài khoản Google với tài khoản Email/Password nếu trùng email (`AdminLinkProviderForUser`).

3. **API & Compute Layer:**
   - **Amazon API Gateway:** Tiếp nhận HTTP REST requests từ Frontend, chuyển tiếp an toàn tới Lambda backend (`https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod`).
   - **AWS Lambda (`smartdocai`):** Chạy ứng dụng FastAPI thông qua Mangum adapter. Được đóng gói dưới dạng Docker Container lưu tại AWS ECR (Memory: 3008 MB, Timeout: 300s).

4. **Storage & Data Layer:**
   - **Amazon S3 (`smartdocai-storage-623035187993`):** Lưu trữ file tài liệu thô, các file chỉ mục FAISS vector store, lịch sử trò chuyện. Đã cấu hình **S3 Intelligent-Tiering** tự động tối ưu chi phí lưu trữ lâu dài.
   - **Amazon DynamoDB (`smartdocai-user-profiles`):** Cơ sở dữ liệu NoSQL lưu thông tin profile người dùng, mã hóa dữ liệu at-rest bằng **AWS Managed KMS Key** (`alias/aws/dynamodb`).

5. **AI Services Layer (AWS Bedrock):**
   - **Amazon Titan Embeddings V2 (`amazon.titan-embed-text-v2:0`):** Tạo vector 1024 chiều (xử lý song song 12 threads).
   - **Qwen 3 Next 80B A3B (`qwen.qwen3-next-80b-a3b`):** Mô hình ngôn ngữ lớn thực hiện suy luận RAG.
   - **FAISS Vector Store:** Cơ sở dữ liệu vector lưu trực tiếp trên S3 per-user, tải vào bộ nhớ tạm `/tmp` của Lambda để tìm kiếm tương đồng cực nhanh.

6. **DevOps, Automation & Hardening Layer:**
   - **AWS CodePipeline & CodeBuild:** Tự động kích hoạt khi push code lên nhánh `main`. Chạy unit test (`pytest` ~60 test cases) và linter (`flake8`) trước khi build Docker Image và cập nhật Lambda.
   - **AWS EventBridge (`smartdocai-cleanup-unconfirmed`):** Tự động gọi Lambda mỗi 5 phút để dọn dẹp các user đăng ký chưa confirm quá 5 phút.
   - **CloudWatch Alarms & SNS Topic (`smartdocai-alerts`):** Theo dõi 4 thông số trọng yếu (Lambda Errors, Duration >25s, Throttles, API Gateway 5xx) và gửi cảnh báo qua email.

---

### 5. TIMELINE

Dự án được triển khai thành công qua 4 giai đoạn chính từ Tháng 6/2026 đến Tháng 7/2026:

| Giai đoạn | Thời gian | Hạng mục công việc chính | Trạng thái |
|---|---|---|---|
| **Giai đoạn 1: Nghiên cứu & Thiết kế** | 22/06/2026 - 27/06/2026 | - Phân tích yêu cầu bài toán RAG.<br>- Khảo sát LLM trên AWS Bedrock (Titan V2, qwen3-next-80b-a3b).<br>- Lập sơ đồ kiến trúc Serverless Container & thiết kế Per-User Isolation. | Hoàn thành |
| **Giai đoạn 2: Backend Core & Auth** | 29/06/2026 - 04/07/2026 | - Xây dựng Backend FastAPI, tích hợp FAISS vector store & Bedrock API.<br>- Cấu hình Cognito User Pool, Hosted UI & PreSignUp Lambda trigger ghép tài khoản.<br>- Triển khai luồng upload tài liệu 3 bước bằng S3 Presigned URL. | Hoàn thành |
| **Giai đoạn 3: Advanced RAG & Frontend** | 06/07/2026 - 11/07/2026 | - Hoàn thiện giao diện React + Vite SPA, kết nối CloudFront CDN.<br>- Triển khai quản lý Profile người dùng trên DynamoDB. | Hoàn thành |
| **Giai đoạn 4: Production Hardening** | 13/07/2026 - 16/07/2026 | - Xây dựng CodePipeline/CodeBuild CI/CD với bộ unit test ~60 test cases (hard-fail).<br>- Triển khai EventBridge dọn dẹp tài khoản chưa xác thực tự động.<br>- Production Hardening: DynamoDB KMS Encryption, OAuth CSRF State UUID, S3 Intelligent-Tiering, CloudWatch Alarms & SNS Topic Cảnh báo. | Hoàn thành |

---

### 6. NGÂN SÁCH

Hệ thống tận dụng tối đa mô hình **AWS Free Tier** và **Serverless Pay-As-You-Go** (chỉ trả tiền cho tài nguyên thực tế sử dụng), giúp tối ưu hóa chi phí vận hành ở mức thấp nhất.

#### Bảng Ước tính Chi phí Hạ tầng hàng tháng (Quy mô Thử nghiệm & Demo)

| Dịch vụ AWS | Mức sử dụng ước tính / tháng | Chi phí ước tính (USD) |
|---|---|---|
| **AWS Lambda** | 100,000 requests, 3008 MB RAM | **$0.00** (Thuộc Free Tier 1M requests & 400,000 GB-s) |
| **Amazon API Gateway** | 100,000 HTTP API calls | **$0.01 - $0.05** |
| **Amazon S3** | 10 GB lưu trữ (Standard + Intelligent-Tiering), 5,000 requests | **$0.15 - $0.30** |
| **Amazon DynamoDB** | < 1 GB data, On-Demand Read/Write | **$0.00** (Thuộc Free Tier 25 WCU/RCU) |
| **Amazon Cognito** | < 1,000 MAUs (Monthly Active Users) | **$0.00** (Free Tier cho 50,000 MAUs) |
| **AWS CloudFront** | < 50 GB Data Transfer Out | **$0.00** (Free Tier cho 1 TB) |
| **AWS CodePipeline & CodeBuild** | ~30 build minutes / tháng | **$0.05 - $0.15** |
| **Amazon Bedrock - Titan Embeddings V2** | ~5,000,000 tokens / tháng ($0.00002 / 1k tokens) | **$0.10 - $0.20** |
| **Amazon Bedrock - Qwen 3 Next 80B A3B (`qwen.qwen3-next-80b-a3b`)** | ~1,000,000 input & 1,000,000 output tokens | **$0.15 - $0.75** |
| **EventBridge & CloudWatch Alarms** | 4 Alarms, 1 Rule, SNS email alerts | **$0.10** |
| **TỔNG CHI PHÍ DỰ KIẾN** | **Vận hành hệ thống thực tế** | **~$0.56 - $1.65 USD / tháng** |

> **Nhận xét:** So với chi phí thuê máy chủ cố định (EC2/VPS GPU tốn từ $30 - $100/tháng), giải pháp Serverless của SmartDocAI sử dụng mô hình Qwen 3 Next 80B A3B trên AWS Bedrock giúp **tiết kiệm hơn 95% chi phí**, hoàn toàn phù hợp cho môi trường thử nghiệm, nghiên cứu và sản xuất quy mô vừa.

---

### 7. RỦI RO & CHIẾN LƯỢC GIẢM THIỂU

#### Ma trận Rủi ro

| Rủi ro Kỹ thuật / Vận hành | Mức độ ảnh hưởng | Xác suất | Chiến lược giảm thiểu (Mitigation Strategy) |
|---|---|---|---|
| **1. Độ trễ phản hồi của LLM hoặc Lambda bị Timeout** | **Cao** | Trung bình | - Cấu hình Lambda Memory **3008 MB** và Timeout **300 giây**.<br>- Chạy song song 12 threads khi tạo Titan embeddings.<br>- Đã cấu hình CloudWatch Alarm `smartdocai-lambda-duration` (>25s) để cảnh báo sớm. |
| **2. Hiện tượng Ảo giác (Hallucination) từ mô hình AI** | **Trung bình** | Trung bình | - Triển khai chế độ **Self-RAG** (kiểm tra độ liên quan của ngữ cảnh và tự chấm điểm grounding câu trả lời).<br>- Triển khai chế độ **Co-RAG** hợp nhất kết quả từ 3 Agent theo cơ chế bỏ phiếu.<br>- Bắt buộc trích dẫn nguồn (citations) từ văn bản gốc. |
| **3. Truy cập trái phép / Rò rỉ dữ liệu giữa các User** | **Rất cao** | Thấp | - Bắt buộc xác thực JWT Token từ Cognito trên mọi API endpoint.<br>- Mã hóa dữ liệu DynamoDB at-rest bằng **AWS Managed KMS Key**.<br>- Phân lập tuyệt đối dữ liệu S3 theo cấu trúc key `uploads/{user_id}/` và `vectorstore/{user_id}/`. |
| **4. Tấn công CSRF OAuth trong quá trình Đăng nhập** | **Cao** | Thấp | - Sinh tham số `state` ngẫu nhiên dạng UUID lưu tại `sessionStorage`.<br>- Thực hiện xác minh 2 tầng độc lập tại Frontend Callback và Cognito OAuth Server token validation. |
| **5. Phát sinh Chi phí Đột biến (Cost Spikes)** | **Trung bình** | Thấp | - Áp dụng **S3 Intelligent-Tiering** tự động chuyển tài liệu cũ sang storage class giá rẻ.<br>- Sử dụng AWS Managed Key cho DynamoDB KMS (miễn phí quản lý key).<br>- Cấu hình CloudWatch Budget Alerts nhận thông báo khi chi phí vượt ngưỡng. |
| **6. Lỗi Code/Build hỏng môi trường Production** | **Cao** | Thấp | - Thiết lập pipeline CI/CD tự động chạy bộ unit test (~60 test cases) và linter trong bước `pre_build`.<br>- Áp dụng chính sách **Hard Fail**: Nếu có bất kỳ test case nào thất bại, pipeline lập tức hủy quá trình build và từ chối deploy. |

---

### 8. KẾT QUẢ KỲ VỌNG

1. **Hiệu năng & Trải nghiệm:** Người dùng có thể trích xuất thông tin từ tài liệu PDF/Word dài hàng trăm trang chỉ trong vài giây với độ chính xác cao và có trích dẫn nguồn minh bạch.
2. **Khả năng mở rộng:** Hệ thống Serverless có khả năng phục vụ đồng thời hàng ngàn người dùng mà không cần thay đổi kiến trúc hoặc can thiệp hạ tầng thủ công.
3. **Độ tin cậy & Bảo mật:** Đạt tiêu chuẩn bảo mật cho ứng dụng đám mây với cơ chế phân lập dữ liệu tuyệt đối, mã hóa dữ liệu KMS và chống tấn công CSRF.