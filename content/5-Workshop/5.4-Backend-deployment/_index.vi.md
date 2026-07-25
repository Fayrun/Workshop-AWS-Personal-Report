---
title : "Triển khai Backend"
date : 2024-01-01 
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---


Trong phần này, nhóm sẽ thực hiện triển khai toàn bộ hệ thống xử lý Backend và cơ sở hạ tầng dịch vụ phụ trợ (Microservices & Serverless architecture) cho ứng dụng SmartDocAI trên nền tảng AWS. Quá trình triển khai bao gồm khởi tạo các dịch vụ cơ sở dữ liệu, dịch vụ lưu trữ tài liệu, quản lý định danh người dùng, xây dựng REST API Gateway, đóng gói và vận hành AI RAG Engine trên AWS Lambda qua Container Image, định tuyến API từ Frontend qua Amazon CloudFront, và tự động hóa toàn bộ quy trình CI/CD với AWS CodePipeline.

### Các Dịch Vụ AWS Được Sử Dụng

- **Amazon Cognito**: Quản lý đăng ký/đăng nhập người dùng, cấp phát JWT Tokens (`ID/Access Token`) bảo mật API và liên kết Domain cho Hosted UI.
- **Amazon DynamoDB**: Lưu trữ thông tin hồ sơ người dùng mở rộng (`smartdocai-user-profiles`) theo mô hình NoSQL On-Demand (`PAY_PER_REQUEST`).
- **Amazon S3**: Lưu trữ tập trung tệp tài liệu gốc, chỉ mục FAISS Vector DB, ảnh đại diện Avatar và hệ thống tệp JSON Metadata phân lập theo từng người dùng (`uploads/`, `vectorstore/`, `processed_files/`, `chat_history/`).
- **AWS Lambda & Amazon ECR**: Xây dựng Engine xử lý backend tập trung (FastAPI + Mangum handler) đóng gói dạng Docker Container Image lưu trên ECR, kết hợp Lambda Trigger cho Cognito và EventBridge Cron Job tự động dọn dẹp tài khoản chưa xác minh.
- **Amazon API Gateway**: Cung cấp REST API Proxy (`{proxy+}`), tích hợp Cognito Authorizer để bảo vệ tài nguyên API và cấu hình CORS Preflight cho Frontend.
- **Amazon CloudFront Integration**: Định tuyến các yêu cầu API (`/api/*`) từ ứng dụng Frontend trên CloudFront CDN tới API Gateway backend thông qua Origin và Behavior tương ứng.
- **AWS CodePipeline & CodeBuild**: Tự động hóa quy trình CI/CD từ GitHub repository, thực thi kiểm thử Linting (`flake8`) & Unit Tests (`pytest`) cơ chế Hard Fail, đóng gói Docker Image và cập nhật mã nguồn hàm Lambda tự động.

---

### Nội dung thực hiện

1. [Tạo Amazon Cognito](5.4.1-creating-amazon-cognito/)
2. [Tạo Amazon DynamoDB](5.4.2-creating-amazon-dynamoDB/)
3. [Tạo Amazon S3 lưu trữ tài liệu](5.4.3-creating-amazon-S3-for-document-storage/)
4. [Triển khai AWS Lambda Engine](5.4.4-creating-AWS-lambda/)
5. [Tạo API Gateway](5.4.5-creating-API-gateway/)
6. [Viết Frontend gọi API Gateway](5.4.6-implementing-frontend-API-gateway-integration/)
7. [Tự động triển khai Lambda bằng CodePipeline](5.4.7-automating-lambda-deployment-with-codePipeline/)
