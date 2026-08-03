---
title: "Worklog Tuần 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

- Mở rộng kiến thức sang các dịch vụ Storage/Networking/Database nâng cao hơn: S3, CloudFront, RDS.
- Tìm hiểu EC2 Auto Scaling và mã hoá dữ liệu với KMS.
- Bắt đầu tìm hiểu Amazon Cognito — dịch vụ sẽ dùng làm nền tảng xác thực cho Smart Docs AI.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                                                                                                                                                                                                                                                                                                                                       | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                      |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | --------------------------------------------------- |
| 2   | - Tìm hiểu về Amazon S3: bucket, object, storage class (Standard/IA/Glacier), versioning<br>&emsp;+ Hiểu sự khác biệt giữa các storage class để chọn đúng cho từng loại dữ liệu (dữ liệu truy cập thường xuyên vs dữ liệu lưu trữ lâu dài) - **Thực hành:** tạo bucket, upload/download object, bật versioning thử nghiệm                                                       | 29/06/2026   | 29/06/2026      | [AWS S3](https://000057.awsstudygroup.com/vi/)      |
| 3   | - Tìm hiểu về CloudFront: CDN, Origin, Distribution, cache behavior<br>&emsp;+ Hiểu cơ chế cache tại Edge Location giúp giảm tải cho origin và tăng tốc độ tải trang cho người dùng ở xa - **Thực hành:** tạo 1 CloudFront Distribution trỏ tới bucket S3 tĩnh, kiểm tra tốc độ tải trước/sau khi có CDN                                                                        | 30/06/2026   | 30/06/2026      |                                                     |
| 4   | - Học Amazon RDS: Multi-AZ, Read Replica, backup/restore tự động<br>&emsp;+ So sánh RDS (managed, ít việc vận hành) với việc tự cài database trên EC2 - Học EC2 Auto Scaling: Launch Template, Auto Scaling Group, scaling policy <br>&emsp;+ Hiểu cơ chế scale-out/scale-in dựa trên metric CPU/traffic                                                                        | 01/07/2026   | 01/07/2026      | [Amazon RDS](https://000005.awsstudygroup.com/vi/)  |
| 5   | - Học AWS KMS: Customer Managed Key vs AWS Managed Key, cách mã hoá dữ liệu at-rest<br>&emsp;+ Ghi chú lại để áp dụng sau này cho DynamoDB của Smart Docs AI - Bắt đầu tìm hiểu Amazon Cognito: User Pool, Identity Pool, User Pool Client <br>&emsp;+ Phân biệt User Pool (quản lý danh tính, đăng ký/đăng nhập) và Identity Pool (cấp quyền truy cập tài nguyên AWS tạm thời) | 02/07/2026   | 02/07/2026      | [AWS Cognito](https://000081.awsstudygroup.com/vi/) |
| 6   | -**Thực hành:** tạo thử 1 Cognito User Pool, thử luồng đăng ký/xác nhận OTP cơ bản qua Console <br>&emsp;+ Test thử để hình dung luồng UserStatus (UNCONFIRMED → CONFIRMED) — kiến thức nền quan trọng cho việc code luồng đăng ký thật ở tuần sau                                                                                                                              | 03/07/2026   | 03/07/2026      |                                                     |
| 7   | - Ôn lại toàn bộ kiến thức đã học 2 tuần đầu, tổng hợp thành note cá nhân - Cùng nhóm rà lại tiến độ frontend, phân công rõ phần việc backend sắp tới (ai phụ trách auth, ai phụ trách RAG/document processing)                                                                                                                                                                 | 04/07/2026   | 04/07/2026      |                                                     |

### Kết quả đạt được tuần 2:

- Nắm được các khái niệm cốt lõi của S3 (storage class, versioning), CloudFront (CDN, cache), RDS (Multi-AZ, Read Replica).
- Hiểu cơ chế EC2 Auto Scaling và cách dùng KMS để mã hoá dữ liệu at-rest.
- Có kiến thức nền tảng về Cognito (User Pool/Identity Pool), thử nghiệm luồng đăng ký/OTP cơ bản qua Console.
- Cùng nhóm thống nhất phân công phần việc backend cho các tuần tiếp theo.
