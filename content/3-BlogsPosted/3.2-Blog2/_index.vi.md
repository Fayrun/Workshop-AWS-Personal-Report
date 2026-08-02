---
title: "EventBridge Scheduler: Khi nào 'nâng cấp' khỏi EventBridge Rule?"
date: 2026-07-28
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

<!-- # EVENTBRIDGE SCHEDULER: KHI NÀO "NÂNG CẤP" KHỎI EVENTBRIDGE RULE? -->

## Giới thiệu

Chào mọi người, nhóm mình đang làm một dự án chatbot hỏi-đáp tài liệu (RAG): người dùng upload file PDF/DOCX rồi hỏi đáp trực tiếp với AI dựa trên nội dung tài liệu đó. Toàn bộ hệ thống chạy serverless trên AWS — API Gateway + Lambda (đóng gói Docker) cho backend, Cognito cho xác thực, DynamoDB + S3 cho lưu trữ, Amazon Bedrock cho phần LLM/Embeddings.

Trong quá trình triển khai, mình cần một cơ chế tự động dọn dẹp các tài khoản Cognito đăng ký nhưng không xác thực email (UNCONFIRMED) — nếu để lâu, chúng chỉ là "rác" trong User Pool. Giải pháp mình chọn là một **EventBridge Rule** chạy `rate(1 minute)` để trigger Lambda kiểm tra và xóa các user quá hạn.

Khi tìm hiểu sâu hơn để viết bài này, mình phát hiện AWS đã có hẳn một dịch vụ riêng cho việc lập lịch — **Amazon EventBridge Scheduler** — mạnh mẽ hơn nhiều so với EventBridge Rule truyền thống. Bài viết này sẽ đi qua sự khác biệt giữa 2 lựa chọn, và quan trọng hơn: tại sao mình vẫn chọn Rule cho use case hiện tại, chứ không phải chuyển ngay sang Scheduler chỉ vì nó "mới hơn".

## EventBridge Scheduler là gì?

Amazon EventBridge Scheduler là dịch vụ serverless giúp tạo, chạy và quản lý các tác vụ theo lịch (schedule) ở quy mô lớn — một lần hoặc lặp lại — trên hơn 270 dịch vụ AWS và hơn 6.000 API actions, mà không cần tự quản lý hạ tầng.

Trước đây, nếu muốn lập lịch một tác vụ, lựa chọn phổ biến nhất là EventBridge Rule với biểu thức `cron()` hoặc `rate()`. Cách này hoạt động tốt, nhưng có một số giới hạn:

- Tối đa **300 rule/region/tài khoản** — không phù hợp nếu bạn cần lập lịch riêng cho hàng nghìn khách hàng (ví dụ: mỗi tenant trong 1 SaaS cần 1 lịch nhắc việc riêng).
- Chỉ hỗ trợ khoảng **20 loại target**.
- Không có sẵn cơ chế retry với exponential backoff, dead-letter queue (DLQ), hay time window để giãn tải request.
- Không hỗ trợ lịch chạy 1 lần (one-time schedule) — chỉ có recurring.

EventBridge Scheduler giải quyết toàn bộ các giới hạn trên: hỗ trợ tới **1 triệu schedule/tài khoản** (thay vì 300 rule/region), throughput lên tới hàng nghìn TPS, kết nối được **hơn 270 dịch vụ AWS và 6.000+ API actions** (thay vì ~20 target của Rule), hỗ trợ lịch chạy 1 lần (`at()`) bên cạnh recurring, có time window để giãn tải request, có sẵn retry + dead-letter queue (mặc định thử lại 185 lần trong 24h), và hỗ trợ đầy đủ time zone/DST thay vì chỉ UTC như Rule.

## Kiến trúc thực tế trong dự án của mình

Lambda backend được thiết kế theo dạng phân nhánh sự kiện: cùng 1 function xử lý cả HTTP request (qua API Gateway/Mangum) lẫn event định kỳ từ EventBridge, dựa vào `event.get("source") == "aws.events"`. Đây là lựa chọn tối giản — không cần thêm Lambda riêng chỉ để chạy cron job.

## Vì sao mình chọn EventBridge Rule chứ không phải Scheduler?

Đây là phần mình nghĩ quan trọng nhất, vì rất dễ bị hỏi ngược "sao không dùng luôn cái mới cho chuẩn":

- **Chỉ có đúng 1 loại tác vụ định kỳ** trong toàn hệ thống — dọn user chưa xác thực. Không có nhu cầu lập lịch riêng theo từng user/tenant, nên không chạm tới giới hạn 300 rule hay cần 1 triệu schedule của Scheduler.
- **Không cần retry/DLQ phức tạp** — nếu 1 lần chạy dọn dẹp bị lỗi, lần chạy tiếp theo (1 phút sau) sẽ tự chạy lại và vẫn dọn đúng các user quá hạn, không cần cơ chế retry riêng.
- **Không cần lịch chạy 1 lần (one-time)** — đây là tác vụ lặp lại vĩnh viễn theo `rate()`, tính năng nổi bật nhất của Scheduler (one-time schedule, time window) không mang lại lợi ích ở use case này.
- **Chi phí & độ phức tạp vận hành:** Scheduler vẫn miễn phí trong Free Tier và không đắt hơn đáng kể, nhưng thêm 1 dịch vụ mới vào kiến trúc đồng nghĩa thêm 1 thứ phải học/maintain — với quy mô demo/thực tập hiện tại, điều đó không tương xứng lợi ích mang lại.

Nói cách khác: Scheduler không "tốt hơn" Rule một cách tuyệt đối — nó tốt hơn ở đúng bài toán mà nó được sinh ra để giải quyết (lập lịch số lượng lớn, đa dạng target, cần retry/DLQ tinh vi). Nếu dự án sau này phát triển thành SaaS thật với hàng nghìn tenant, mỗi tenant có nhu cầu lập lịch riêng (ví dụ: nhắc hạn dùng thử, tự động dọn dữ liệu theo chính sách retention riêng từng khách hàng) — đó chính là lúc nên chuyển sang Scheduler.

## Kết luận

EventBridge Scheduler là một nâng cấp thực sự mạnh mẽ so với EventBridge Rule, đặc biệt cho các hệ thống SaaS multi-tenant cần lập lịch ở quy mô lớn. Nhưng việc chọn công nghệ nên dựa trên bài toán thực tế, không phải dựa trên việc dịch vụ nào "mới hơn". Với một tác vụ định kỳ đơn giản như dọn dẹp user chưa xác thực, EventBridge Rule vẫn là lựa chọn tối giản và hợp lý.

## Link bài viết

Bài blog chính thức của AWS về Amazon EventBridge Scheduler:
https://aws.amazon.com/blogs/compute/introducing-amazon-eventbridge-scheduler/
