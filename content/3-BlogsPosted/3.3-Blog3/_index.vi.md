---
title: "Kiến Trúc Cơ Sở Cho Amazon Bedrock Trong Môi Trường AWS Landing Zone"
date: 2026-01-25
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

> **Bài viết của: Phong** (thành viên trong nhóm). Bài được chia sẻ lại trong báo cáo cá nhân này làm tư liệu tham khảo chung của nhóm.

## 1. Mục Đích Hệ Thống Và Bài Toán Triển Khai

Khi ứng dụng Trí tuệ nhân tạo tạo sinh (GenAI) ở quy mô doanh nghiệp, các tổ chức thường phải đối mặt với bài toán lớn về quản trị rủi ro, bảo mật và tuân thủ. Nếu cho phép các nhóm phát triển tự do triển khai Amazon Bedrock trong một tài khoản AWS duy nhất hoặc thiếu kiểm soát, doanh nghiệp dễ gặp phải tình trạng rò rỉ dữ liệu, vi phạm mạng và khó kiểm soát chi phí.

Bài toán đặt ra là cần xây dựng một nền tảng kiến trúc cơ sở (baseline architecture) vững chắc. Hạ tầng này phải cho phép các nhóm kỹ sư đổi mới sáng tạo và thử nghiệm AI một cách nhanh chóng, nhưng vẫn nằm trong một ranh giới an toàn, được giám sát tập trung và cô lập nghiêm ngặt về mặt dữ liệu.

## 2. Giải Pháp: Quản Trị Đa Tài Khoản Với AWS Landing Zone

Giải pháp cốt lõi là không chạy các khối lượng công việc GenAI trong một tài khoản nguyên khối, mà phân bổ chúng vào một môi trường đa tài khoản (multi-account) thông qua AWS Control Tower (Landing Zone).

Hệ thống phân tách rõ ràng các tài khoản theo mục đích sử dụng (ví dụ: Sandbox cho thử nghiệm, Dev/Test, và Production).

Cách tiếp cận này giúp giới hạn "bán kính ảnh hưởng" (blast radius). Nếu một môi trường thử nghiệm bị cấu hình sai, các tài khoản chứa dữ liệu sản xuất hoặc dịch vụ lõi của doanh nghiệp vẫn hoàn toàn an toàn và không bị ảnh hưởng.

## 3. Kiến Trúc Mạng Cô Lập Và Quản Lý Truy Cập Chi Tiết

Kiến trúc cơ sở này thiết lập các lớp bảo vệ nghiêm ngặt ở cả mức độ mạng và định danh:

**Lớp 1: Kết nối mạng riêng tư (Private Networking)**

Thay vì gọi API của Amazon Bedrock qua internet công cộng, kiến trúc sử dụng AWS PrivateLink (VPC Endpoints).

Mọi luồng dữ liệu (từ ứng dụng đến mô hình LLM) đều đi qua mạng trục nội bộ của AWS. Kết hợp với AWS Transit Gateway trong mô hình Hub-and-Spoke, doanh nghiệp có thể kiểm soát hoàn toàn đường đi của dữ liệu, ngăn chặn tuyệt đối nguy cơ đánh cắp dữ liệu qua internet.

**Lớp 2: Quản lý định danh và quyền hạn (Identity & Access)**

Sử dụng AWS IAM Identity Center để cấp quyền truy cập tạm thời cho các nhà phát triển.

Hệ thống áp dụng quyền kiểm soát truy cập dựa trên vai trò (RBAC) và thuộc tính (ABAC), đảm bảo nguyên tắc đặc quyền tối thiểu (Least Privilege). Kỹ sư ở dự án A sẽ chỉ thấy và sử dụng được các model/tài nguyên được cấp phép riêng cho dự án đó.

## 4. Quy Trình Vận Hành Và Giám Sát Tập Trung

Để duy trì tính tuân thủ mà không làm chậm quá trình phát triển, hệ thống áp dụng cơ chế tự động hóa quản trị:

- **Rào chắn tự động (Guardrails/SCPs):** Tại cấp độ Tổ chức (AWS Organizations), hệ thống áp dụng các Chính sách kiểm soát dịch vụ (SCPs). Ví dụ: một SCP có thể cấm người dùng xuất mô hình (model exfiltration) hoặc chỉ cho phép gọi các mô hình Foundation Models (FMs) đã được phê duyệt.
- **Nhật ký tập trung (Centralized Logging):** Toàn bộ lịch sử gọi API, bao gồm cả nội dung đầu vào (prompt) và đầu ra (response) của mô hình (thông qua tính năng Model Invocation Logging), được mã hóa và gửi thẳng về một tài khoản Log Archive cách ly. Đội ngũ an ninh có thể giám sát theo thời gian thực bằng Amazon CloudWatch và CloudTrail mà không cần truy cập vào tài khoản của nhà phát triển.

## 5. Tăng Cường An Toàn Dữ Liệu Chuyên Sâu Của Bedrock

Bên cạnh các lớp bảo mật hạ tầng của Landing Zone, kiến trúc kế thừa các tính năng bảo mật mặc định mạnh mẽ của chính dịch vụ Amazon Bedrock:

- **Quyền riêng tư tuyệt đối:** Dữ liệu prompt, câu trả lời và dữ liệu tinh chỉnh (fine-tuning) của doanh nghiệp được lưu trữ cục bộ, mã hóa bằng AWS KMS và hoàn toàn không được sử dụng để huấn luyện lại các mô hình nền tảng của Amazon hay các bên thứ ba.
- **Tích hợp Bedrock Guardrails:** Kết hợp hạ tầng an toàn với Guardrails nội tại của Bedrock để tự động chặn các prompt yêu cầu cung cấp thông tin nhận dạng cá nhân (PII), bảo vệ toàn vẹn dữ liệu nhạy cảm của khách hàng ở ngay lớp ứng dụng.

## Tài liệu tham khảo

https://aws.amazon.com/vi/blogs/architecture/amazon-bedrock-baseline-architecture-in-an-aws-landing-zone/

## Link bài post trong nhóm AWS Study Group

https://www.facebook.com/groups/awsstudygroupfcj/permalink/2207763766655250
