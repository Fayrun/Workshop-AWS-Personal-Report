---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

- Rà soát và chỉnh sửa lại nội dung Workshop (mục 5) cho rõ ràng, nhất quán hơn.
- Viết và đăng các bài blog kỹ thuật đã chuẩn bị lên cộng đồng AWS Study Group.
- Bổ sung đầy đủ ảnh chụp minh chứng cho phần Kiểm thử hệ thống (5.5), và hoàn thiện bản báo cáo cá nhân để nộp.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                     |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | -------------------------------------------------- |
| 2   | - Rà soát và chỉnh sửa lại nội dung mục Workshop cho rõ ràng, nhất quán hơn <br>&emsp;+ Đọc lại toàn bộ mục 5-Workshop, thống nhất lại văn phong và thuật ngữ giữa các phần do nhiều thành viên viết <br>&emsp;+ Phát hiện và dọn các đoạn nội dung mẫu còn sót lại từ template gốc (chưa được thay bằng nội dung thật của dự án)                                                                                                                                                                                                                                            | 27/07/2026   | 27/07/2026      |                                                    |
| 3   | - Viết và đăng 2 bài blog kỹ thuật lên nhóm Facebook AWS Study Group <br>&emsp;+ Blog 1: phân tích bug rò rỉ dữ liệu multi-tenant đã gặp phải và đánh giá tính năng Lambda Tenant Isolation Mode mới của AWS <br>&emsp;+ Blog 2: so sánh EventBridge Rule và EventBridge Scheduler, lý giải vì sao chọn Rule cho use case dọn dẹp user hiện tại <br>&emsp;+ Blog 3: shared a teammate's post on the baseline architecture for Amazon Bedrock in an AWS Landing Zone environment — multi-account governance, network isolation, and Bedrock's built-in data security features | 28/07/2026   | 28/07/2026      | <https://www.facebook.com/groups/awsstudygroupfcj> |
| 4   | - Chụp bổ sung ảnh minh chứng còn thiếu cho mục Kiểm thử hệ thống (5.5) <br>&emsp;+ Chụp lại kết quả test Authentication, Document/RAG, Security (bao gồm test CSRF state param cho luồng Google Login), Profile <br>&emsp;+ Chụp minh chứng CloudWatch Alarms, SNS subscription, CloudWatch Logs Insights cho mục Monitoring — phần được đánh giá quan trọng nhất <br>&emsp;+ Chụp kết quả chạy `pytest` local và log CodeBuild cho mục CI/CD <br> - Rà soát lại danh sách ảnh đã chụp so với checklist, xác nhận không thiếu ảnh nào theo yêu cầu                          | 29/07/2026   | 29/07/2026      |                                                    |
| 5   | - Tách bản báo cáo cá nhân sang repository riêng, deploy qua GitHub Pages <br>&emsp;+ Cấu hình lại `baseURL` và workflow GitHub Actions cho đúng domain cá nhân <br>&emsp;+ Gỡ thư mục `public/` (build output) ra khỏi Git tracking, tránh xung đột giữa các lần build <br> - Kiểm tra lại toàn bộ trang web cá nhân hiển thị đúng (giao diện, ảnh đại diện, logo) sau khi tách repo                                                                                                                                                                                        | 30/07/2026   | 30/07/2026      |                                                    |
| 6   | - Chèn toàn bộ ảnh minh chứng đã chụp vào đúng vị trí trong các file Markdown tương ứng <br>&emsp;+ Build lại site (`hugo --minify`) sau mỗi lần chỉnh sửa để đảm bảo không phát sinh lỗi                                                                                                                                                                                                                                                                                                                                                                                    | 31/07/2026   | 31/07/2026      |                                                    |
| 7   | - Rà soát lại toàn bộ báo cáo cá nhân lần cuối trước khi bước sang tuần nộp Workshop <br>&emsp;+ Kiểm tra lại đồng bộ nội dung giữa nhánh làm việc cá nhân và nhánh chung của nhóm                                                                                                                                                                                                                                                                                                                                                                                           | 01/08/2026   | 01/08/2026      |                                                    |

### Kết quả đạt được tuần 6:

- Rà soát và chỉnh sửa lại nội dung mục Workshop (5-Workshop) cho rõ ràng, nhất quán hơn, dọn sạch nội dung mẫu còn sót từ template gốc.
- Viết và đăng 2 bài blog kỹ thuật lên nhóm Facebook AWS Study Group — xem chi tiết tại mục 3.1-3.2.
- Bổ sung đầy đủ ảnh minh chứng cho toàn bộ phần Kiểm thử hệ thống (5.5), đặc biệt là Monitoring và Security.
- Tách và cấu hình thành công bản báo cáo cá nhân trên GitHub Pages riêng, hoạt động độc lập với repo chung của nhóm.
- Rà soát, chỉnh sửa và hoàn thiện nội dung báo cáo cá nhân trước khi bước sang tuần nộp Workshop.
- …
