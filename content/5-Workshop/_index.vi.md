---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

### Tổng quan

Chương này trình bày chi tiết từng bước (step-by-step) xây dựng, cấu hình, triển khai, kiểm thử và tự động hóa toàn bộ ứng dụng SmartDocAI trên hạ tầng AWS Serverless.

Hệ thống kết hợp mô hình Serverless Container Architecture sử dụng AWS Lambda (FastAPI Container), Amazon API Gateway, AWS Cognito, Amazon DynamoDB, Amazon S3, AWS CloudFront và mô hình Generative AI tiên tiến Amazon Bedrock (Titan Embeddings V2 & Qwen 3 Next 80B A3B).

Người học sẽ được thực hành từ khâu chuẩn bị tài nguyên, đóng gói Docker Container, thiết lập các dịch vụ đám mây, cấu hình phân quyền bảo mật IAM, tích hợp quy trình tự động hóa CI/CD qua AWS CodePipeline/CodeBuild cho đến kiểm thử tải và dọn dẹp tài nguyên.

---

#### Nội dung chi tiết Hướng dẫn Thực hành

1. **[Tổng quan về Workshop (Workshop Overview)](5.1-Workshop-overview/)**
   - 5.1.1 [Đặc tả Kiến trúc Frontend](5.1-Workshop-overview/5.1.1%20-frontend-architecture/)
   - 5.1.2 [Đặc tả Kiến trúc Backend & RAG Pipeline](5.1-Workshop-overview/5.1.2%20-backend-architecture/)
   - 5.1.3 [Sơ đồ Kiến trúc Tổng quan trên AWS](5.1-Workshop-overview/5.1.3%20-overall-aws-architecture/)
   - 5.1.4 [Danh sách các Dịch vụ AWS Sử dụng](5.1-Workshop-overview/5.1.4%20-aws-services-used/)
   - 5.1.5 [Đặc tả Giao diện và Chức năng Hệ thống](5.1-Workshop-overview/5.1.5%20-ui-function/)

2. **[Chuẩn bị Môi trường (Prerequisites)](5.2-Prerequiste/)**
   - 5.2.1 [Chuẩn bị Mã nguồn Dự án](5.2-Prerequiste/5.2.1-source-code-preparation/)
   - 5.2.2 [Chuẩn bị Tài khoản AWS & Quyền truy cập Amazon Bedrock](5.2-Prerequiste/5.2.2-aws-account-preparation/)
   - 5.2.3 [Tạo IAM User & Cấu hình AWS CLI](5.2-Prerequiste/5.2.3-creating-an-IAM-user/)

3. **[Triển khai Frontend SPA (Frontend Deployment)](5.3-Frontend-deployment/)**
   - 5.3.1 [Tạo S3 Bucket lưu trữ Static Website](5.3-Frontend-deployment/5.3.1-create-S3-bucket/)
   - 5.3.2 [Kích hoạt tính năng Static Website Hosting](5.3-Frontend-deployment/5.3.2-enable-static-web-hosting/)
   - 5.3.3 [Cấu hình Block Public Access Settings](5.3-Frontend-deployment/5.3.3-config-block-public-access-settings/)
   - 5.3.4 [Cấu hình S3 Bucket Policy cấp quyền Đọc công khai](5.3-Frontend-deployment/5.3.4-config-bucket-policy-public-read/)
   - 5.3.5 [Xác minh và Kiểm tra Trang web Static](5.3-Frontend-deployment/5.3.5-website-verification/)
   - 5.3.6 [Tăng tốc Phân phối & Bảo mật với AWS CloudFront (CDN)](5.3-Frontend-deployment/5.3.6-accelerating-website-performance-with-cloudfront/)
   - 5.3.7 [Tự động hóa Triển khai Frontend với AWS CodePipeline](5.3-Frontend-deployment/5.3.7-codepipeline-to-S3-frontend/)

4. **[Triển khai Hạ tầng Backend AWS (Backend Deployment)](5.4-Backend-deployment/)**
   - 5.4.1 [Khởi tạo Amazon Cognito User Pool & Binding Domain Hosted UI](5.4-Backend-deployment/5.4.1-creating-amazon-cognito/)
   - 5.4.2 [Khởi tạo các Bảng NoSQL trên Amazon DynamoDB](5.4-Backend-deployment/5.4.2-creating-amazon-dynamoDB/)
   - 5.4.3 [Tạo Amazon S3 Bucket lưu trữ Tài liệu & Cấu hình CORS](5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/)
   - 5.4.4 [Đóng gói Docker Container & Triển khai AWS Lambda Engine](5.4-Backend-deployment/5.4.4-creating-AWS-lambda/)
   - 5.4.5 [Tạo Amazon API Gateway REST API & Cognito Authorizer](5.4-Backend-deployment/5.4.5-creating-API-gateway/)
   - 5.4.6 [Định tuyến & Tích hợp Frontend CloudFront sang API Gateway](5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/)
   - 5.4.7 [Tự động hóa Triển khai Backend Lambda với AWS CodePipeline & CodeBuild](5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/)

5. **[Kiểm thử hệ thống (System Testing)](5.5-System-testing/)**
   - 5.5.1 [Kiểm thử xác thực](5.5-System-testing/5.5.1-Authentication/)
   - 5.5.2 [Kiểm thử tải lên tài liệu & RAG](5.5-System-testing/5.5.2-Document-RAG/)
   - 5.5.3 [Kiểm thử bảo mật](5.5-System-testing/5.5.3-Security/)
   - 5.5.4 [Kiểm thử hồ sơ cá nhân](5.5-System-testing/5.5.4-Profile/)
   - 5.5.5 [Giám sát & Nhật ký hệ thống](5.5-System-testing/5.5.5-Monitoring/)
   - 5.5.6 [Kiểm thử tự động CI/CD](5.5-System-testing/5.5.6-CICD/)

6. **[Tổng kết (Conclusion)](5.6-Conclusion/)**
   - 5.6.1 [Tổng kết Workshop & Chi phí](5.6-Conclusion/5.6.1-Summary-Cost/)
   - 5.6.2 [Dọn dẹp tài nguyên](5.6-Conclusion/5.6.2-Cleanup/)
   - 5.6.3 [Bước tiếp theo & Tài liệu tham khảo](5.6-Conclusion/5.6.3-Next-Steps/)
