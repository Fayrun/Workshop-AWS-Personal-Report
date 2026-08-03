---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Phần này tổng hợp các bài blog kỹ thuật đã được đăng và chia sẻ trong quá trình thực tập. Hai bài đầu được xây dựng từ những vấn đề thực tế gặp phải khi phát triển dự án Smart Docs AI trên AWS, trong khi bài cuối là bài viết được chia sẻ từ thành viên trong nhóm nhằm mở rộng kiến thức về kiến trúc Amazon Bedrock và AWS Landing Zone.

### [Blog 1 - Lambda Tenant Isolation Mode: Tính năng mới có giải quyết được bug rò rỉ dữ liệu Multi-Tenant không?](3.1-Blog1/)

Bài viết phân tích lỗi rò rỉ dữ liệu multi-tenant xảy ra trong quá trình phát triển dự án Smart Docs AI. Nội dung giải thích cơ chế hoạt động của Lambda Tenant Isolation Mode, lý do tính năng này không thể tự động khắc phục lỗi ở tầng ứng dụng, đồng thời trình bày giải pháp thực tế đã được áp dụng để đảm bảo cách ly dữ liệu giữa các người dùng.

### [Blog 2 - EventBridge Scheduler: Khi nào "nâng cấp" khỏi EventBridge Rule?](3.2-Blog2/)

Bài viết so sánh Amazon EventBridge Scheduler với EventBridge Rule thông qua một tình huống triển khai thực tế. Nội dung phân tích ưu điểm của Scheduler, đồng thời giải thích vì sao EventBridge Rule vẫn là lựa chọn phù hợp nhất đối với tác vụ dọn dẹp định kỳ trong dự án, qua đó nhấn mạnh việc lựa chọn dịch vụ AWS cần dựa trên nhu cầu thực tế thay vì chỉ theo xu hướng.

### [Blog 3 - Kiến trúc cơ sở cho Amazon Bedrock trong môi trường AWS Landing Zone (Chia sẻ)](3.3-Blog3/)

Đây là bài viết được chia sẻ từ thành viên trong nhóm nhằm giới thiệu kiến trúc triển khai Amazon Bedrock trong môi trường AWS Landing Zone. Nội dung trình bày mô hình quản trị đa tài khoản, kiến trúc mạng cô lập, quản lý định danh và quyền truy cập, cơ chế giám sát tập trung cùng các phương pháp bảo mật giúp xây dựng hệ thống Generative AI an toàn và có khả năng mở rộng trên AWS.
