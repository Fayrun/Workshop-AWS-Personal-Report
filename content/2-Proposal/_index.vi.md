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
- **Môi trường triển khai:** AWS Region `us-east-1`
- **Domain Production:** 
  - Frontend CDN (CloudFront): `https://dutf3c70nnjzl.cloudfront.net`
  - Backend API Gateway: `https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod`

---

### 2. MỤC TIÊU DỰ ÁN

1. **Xây dựng Hệ thống RAG Serverless Hoàn chỉnh (100% Serverless):**
   - Vận hành hoàn toàn trên hạ tầng Serverless (AWS Lambda, S3, DynamoDB, Bedrock, API Gateway, CloudFront) giúp hệ thống tự động co giãn theo lưu lượng (Auto-scaling) và đạt mức chi phí tối ưu nhất (Pay-as-you-go).

2. **Tối ưu hóa Khả năng Hỏi đáp Tri thức (Advanced RAG):**
   - Hỗ trợ 3 chế độ hỏi đáp nâng cao:
     - **Standard RAG:** Phân tích ngữ nghĩa nhanh chóng qua FAISS vector store, tìm kiếm từ khoá theo BM25, sử dụng thuật toán Hybrid Search kết hợp cả hai phương pháp để tối ưu hóa khả năng tìm kiếm.
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
- **Nghẽn tải payload khi upload file lớn:** Các dịch vụ API Gateway truyền thống thường bị giới hạn kích thước request (10 MB), gây thất bại khi người dùng tải lên tài liệu.
### 4. KIẾN TRÚC GIẢI PHÁP

#### Sơ đồ Kiến trúc Tổng quan Hệ thống (Overall AWS Architecture)

![Sơ đồ Kiến trúc Tổng quan SmartDocAI trên AWS](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/architecture-diagram.png)

---

#### Danh sách & Vai trò Chi tiết các Dịch vụ AWS Sử dụng

| Dịch vụ AWS | Loại hình Dịch vụ | Vai trò & Chức năng trong Hệ thống SmartDocAI |
|---|---|---|
| **AWS CloudFront** | Content Delivery Network (CDN) | Phân phối ứng dụng React + Vite Frontend toàn cầu qua giao thức HTTPS (`https://dutf3c70nnjzl.cloudfront.net`), tối ưu tốc độ tải trang và giảm độ trễ cho người dùng. |
| **AWS Cognito** | Identity & Access Management | Quản lý đăng ký, đăng nhập (User Pool `us-east-1_3oq5wIiuu`), cấp phát JWT Token, tích hợp Hosted UI **Google OAuth 2.0** và PreSignUp Trigger tự động ghép tài khoản trùng email (`AdminLinkProviderForUser`). |
| **Amazon API Gateway** | RESTful API Gateway | Cổng REST API tập trung (`https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod`), tiếp nhận HTTP Request từ Frontend, xác thực với Cognito Authorizer và chuyển tiếp an toàn (AWS_PROXY) tới AWS Lambda. Hỗ trợ xử lý CORS preflight (`OPTIONS`). |
| **AWS Lambda** | Serverless Compute | Môi trường thực thi serverless chạy FastAPI backend đóng gói dạng Docker Container (Memory: 3008 MB, Timeout: 300s). Xử lý toàn bộ logic RAG, embedding, trao đổi Bedrock, đọc/ghi S3 và DynamoDB. |
| **Amazon ECR** | Container Registry | Kho lưu trữ các Docker Container Images của Backend FastAPI. Cung cấp image cho AWS Lambda cập nhật tự động khi có bản build mới từ CI/CD. |
| **Amazon S3** | Object Storage | Lưu trữ file tài liệu gốc (`uploads/{user_id}/`), bộ chỉ mục FAISS Vector Store (`vectorstore/{user_id}/`), lịch sử chat (`chat_history/`) và avatar. Cấu hình **S3 Intelligent-Tiering** tự động tối ưu chi phí lưu trữ lâu dài và cấp **Presigned URLs** để upload an toàn. |
| **Amazon DynamoDB** | NoSQL Database | Lưu trữ hồ sơ chi tiết người dùng (`smartdocai-user-profiles`) bao gồm: Họ tên, Email, Số điện thoại, Ngày sinh, gói dịch vụ, hạn mức tài liệu. Đảm bảo tốc độ truy vấn miligiây với mô hình On-Demand. |
| **AWS KMS** | Key Management Service | Mã hóa dữ liệu at-rest trên Amazon DynamoDB bằng **AWS Managed KMS Key** (`alias/aws/dynamodb`), bảo vệ an toàn thông tin cá nhân của người dùng. |
| **Amazon Bedrock** | Generative AI Platform | Phân tích và sinh phản hồi tri thức AI: <br>- **Titan Embeddings V2 (`amazon.titan-embed-text-v2:0`):** Tạo vector nhúng 1024 chiều (xử lý đa luồng 12 threads).<br>- **Qwen 3 Next 80B A3B (`qwen.qwen3-next-80b-a3b`):** Mô hình LLM suy luận RAG và sinh câu trả lời kèm trích dẫn nguồn. |
| **AWS CodePipeline** | CI/CD Orchestration | Tự động hóa luồng triển khai liên tục. Tự động kích hoạt khi push code lên nhánh GitHub `main`, phối hợp cùng CodeBuild để kiểm thử và deploy. |
| **AWS CodeBuild** | Automated Build & Test | Môi trường thực thi build container. Tự động chạy linter (`flake8`) và bộ unit test (~60 test cases với `pytest`) theo chính sách **Hard Fail** trước khi build Docker Image và cập nhật Lambda. |
| **Amazon EventBridge** | Event Bus & Scheduling | Cron job định thời tự động gọi Lambda mỗi 5 phút để dọn dẹp các tài khoản đăng ký rác chưa xác thực email quá 5 phút (`smartdocai-cleanup-unconfirmed`). |
| **Amazon CloudWatch** | Monitoring & Observability | Thu thập log toàn hệ thống (CloudWatch Logs) và quản lý 4 Alarms giám sát chỉ số sinh tử: Lambda Errors > 0, Duration > 25s, Throttles > 0, API Gateway 5xx Errors > 0. |
| **Amazon SNS** | Push Notification Service | Kênh phát thông báo cảnh báo sự cố khẩn cấp (`smartdocai-alerts`). Khi CloudWatch Alarm bị vi phạm, SNS tự động gửi email cảnh báo tức thì tới quản trị viên. |

---

### 5. TIMELINE

Dự án được triển khai thành công qua 6 giai đoạn chính từ Tháng 6/2026 đến Tháng 8/2026:

| Giai đoạn | Thời gian | Hạng mục công việc chính | Trạng thái |
|---|---|---|---|
| **Giai đoạn 1: Nghiên cứu & Thiết kế** | 22/06/2026 - 27/06/2026 | - Phân tích yêu cầu bài toán RAG.<br>- Khảo sát LLM trên AWS Bedrock (Titan V2, qwen3-next-80b-a3b).<br>- Lập sơ đồ kiến trúc Serverless Container & thiết kế Per-User Isolation.<br>- Thiết kế giao diện | Hoàn thành |
| **Giai đoạn 2: Backend Core & Auth** | 29/06/2026 - 04/07/2026 | - Xây dựng Backend FastAPI, tích hợp FAISS vector store & Bedrock API.<br>- Cấu hình Cognito User Pool, Hosted UI & PreSignUp Lambda trigger ghép tài khoản.<br>- Triển khai luồng upload tài liệu 3 bước bằng S3 Presigned URL. | Hoàn thành |
| **Giai đoạn 3: Advanced RAG & Frontend** | 06/07/2026 - 11/07/2026 | - Hoàn thiện giao diện React + Vite SPA, tạo static website hosting.<br>- Triển khai quản lý Profile người dùng trên DynamoDB. | Hoàn thành |
| **Giai đoạn 4: Production Hardening** | 13/07/2026 - 18/07/2026 | - Kết nối CloudFront CDN.<br>- Xây dựng CodePipeline/CodeBuild CI/CD với bộ unit test ~60 test cases (hard-fail).<br>- Triển khai EventBridge dọn dẹp tài khoản chưa xác thực tự động.<br>- Production Hardening: DynamoDB KMS Encryption, OAuth CSRF State UUID, S3 Intelligent-Tiering, CloudWatch Alarms & SNS Topic Cảnh báo. | Hoàn thành |
| **Giai đoạn 5: System Testing & Optimization** | 20/07/2026 - 25/07/2026 | - Kiểm thử toàn diện hệ thống (End-to-End System Testing), đo đạc latency và xử lý cold start AWS Lambda.<br>- Tối ưu hóa các chiến thuật RAG (Self-RAG, Co-RAG, Re-ranking) và khả năng trích dẫn nguồn.<br>- Xử lý ngoại lệ, đồng bộ dữ liệu per-user và hoàn thiện trải nghiệm UI/UX. | Hoàn thành |
| **Giai đoạn 6: Documentation & Workshop Report** | 27/07/2026 - 01/08/2026 | - Xây dựng bộ tài liệu báo cáo Workshop trên nền tảng Hugo (song ngữ Việt - Anh).<br>- Trích xuất đặc tả chi tiết kiến trúc Backend AWS, Cognito, DynamoDB, S3, Lambda, API Gateway và giao diện hệ thống.<br>- Tổng kết kết quả, nghiệm thu và hoàn thiện tài liệu dự án. | Hoàn thành |

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
| **Amazon ECR** | ~1-2 GB lưu trữ Docker image | **$0.10 - $0.20** |
| **Amazon Bedrock - Titan Embeddings V2** | ~5,000,000 tokens / tháng ($0.00002 / 1k tokens) | **$0.10 - $0.20** |
| **Amazon Bedrock - Qwen 3 Next 80B A3B (`qwen.qwen3-next-80b-a3b`)** | ~1,000,000 input & 1,000,000 output tokens | **$0.15 - $0.75** |
| **EventBridge & CloudWatch Alarms** | 4 Alarms, 1 Rule, SNS email alerts | **$0.10** |
| **TỔNG CHI PHÍ DỰ KIẾN** | **Vận hành hệ thống thực tế** | **~$0.66 - $1.85 USD / tháng** |


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

### 8. KẾT QUẢ KỲ VỌNG VỀ TÍNH NĂNG VẬN HÀNH

#### 1. Nhóm Chức năng Xác thực & Bảo mật Tài khoản (Authentication & Security)
- **Đăng ký 2 bước linh hoạt:** Tiếp nhận thông tin người dùng mới, tự động gửi mã OTP 6 chữ số qua Email để kích hoạt tài khoản trên AWS Cognito.
- **Đa phương thức đăng nhập:** Hỗ trợ đăng nhập truyền thống (Email/Password) và Đăng nhập nhanh SSO bằng tài khoản Google (OAuth 2.0 / OIDC qua Cognito Hosted UI).
- **Tự động liên kết tài khoản (Account Linking):** PreSignUp Lambda Trigger tự động hợp nhất tài khoản Google với tài khoản Email/Password nếu trùng địa chỉ email (`AdminLinkProviderForUser`).
- **Tự động xử lý Username (Automatic Username Resolution):** Tự động tra cứu username thật khi người dùng đăng nhập bằng Email thay cho Cognito internal username (`Google_...`), thực hiện tự động thử lại (retry) mượt mà.
- **Bảo mật phiên làm việc:** Lưu trữ JWT Token an toàn tại `sessionStorage`, bảo vệ chống tấn công CSRF OAuth với tham số `state` UUID và mã hóa dữ liệu DynamoDB at-rest bằng AWS KMS Key.

#### 2. Nhóm Chức năng Quản lý & Xử lý Tài liệu (Document Management & Ingestion)
- **Tải lên tài liệu trực tiếp:** Cho phép kéo & thả hoặc chọn các file tài liệu định dạng PDF, DOCX. Tải lên trực tiếp từ Frontend tới S3 qua cơ chế Presigned URL.
- **Tự động trích xuất & Đánh chỉ mục:** Hệ thống tự động phân tách văn bản (Chunking), tạo Vector nhúng với **Amazon Titan Embeddings V2** và xây dựng bộ chỉ mục FAISS Vector Store lưu trên S3 per-user.
- **Quản lý danh sách tài liệu:** Hiển thị danh sách file đã xử lý thành công (kèm định dạng, số trang, số chunk, dung lượng), cho phép xóa từng file hoặc xóa toàn bộ kho chỉ mục vector.

#### 3. Nhóm Chức năng Hỏi đáp & Tra cứu Tri thức AI (Advanced RAG Engine)
- **Tùy chỉnh 3 Chế độ RAG Nâng cao:**
  - **Standard RAG:** Phản hồi nhanh chóng với cơ chế **Hybrid Search** (kết hợp 60% FAISS Vector Search + 40% BM25 Keyword Search) và tái xếp hạng ngữ cảnh bằng Cross-Encoder Re-ranker (`ms-marco-MiniLM-L-12-v2`).
  - **Self-RAG:** Tự động sửa lại câu hỏi (Query Rewriting), lọc ngữ cảnh nhiễu và tự chấm điểm độ chính xác (grounding check) trước khi đưa ra câu trả lời.
  - **Co-RAG (Multi-Agent):** Phối hợp 3 Agent chuyên biệt (Semantic, Keyword, Conceptual) và hợp nhất kết quả theo cơ chế bỏ phiếu (voting) cho các truy vấn phức tạp.
- **Lọc tài liệu truy vấn (File Filter):** Giới hạn không gian tìm kiếm tri thức trong một hoặc nhiều tài liệu cụ thể do người dùng tick chọn.
- **Minh bạch Nguồn trích dẫn (Source Citations Viewer):** Đi kèm với câu trả lời từ LLM (Qwen 3 Next 80B A3B trên Bedrock), hiển thị chính xác tên file gốc, trang số bao nhiêu và đoạn văn bản trích dẫn tương ứng.
- **Hiển thị tiến trình suy luận (Thinking Process):** Cho phép xem các bước AI truy xuất thông tin (Retrieval & Reasoning steps).

#### 4. Nhóm Chức năng Lịch sử Hội thoại & Quản lý Profile (Chat History & User Profile)
- **Quản lý phiên hội thoại:** Tự động lưu trữ lịch sử hỏi đáp per-user (`chat_history/{user_id}.json`), hỗ trợ xem lại các cuộc hội thoại cũ hoặc xóa lịch sử chat.
- **Phản hồi phong phú (Rich Text Response):** Định dạng câu trả lời AI dưới dạng Markdown, Code Block có nút Copy mã nguồn, Công thức toán LaTeX và Bảng dữ liệu.
- **Quản lý Profile cá nhân:** Xem và cập nhật thông tin cá nhân (Họ tên, Số điện thoại, Ngày sinh), đổi mật khẩu và cập nhật Ảnh đại diện (Avatar) có hỗ trợ nén ảnh & lưu trữ linh hoạt (DynamoDB / S3).

#### 5. Nhóm Chức năng Tự động hóa & Giám sát Hệ thống (Automation & Monitoring)
- **Tự động dọn dẹp tài khoản rác:** EventBridge định thời 5 phút/lần gọi Lambda để dọn dẹp các user đăng ký chưa xác thực email Quá 5 phút.
- **Tự động triển khai CI/CD:** CodePipeline & CodeBuild tự động chạy bộ unit test (~60 test cases) và linter trước khi build Docker Image và deploy Lambda.
- **Giám sát & Cảnh báo sự cố:** CloudWatch Alarms theo dõi 4 chỉ số sinh tử của hệ thống và tự động gửi cảnh báo qua email thông qua AWS SNS Topic.