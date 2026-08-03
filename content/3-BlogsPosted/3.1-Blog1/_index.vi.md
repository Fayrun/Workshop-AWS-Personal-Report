---
title: "LAMBDA TENANT ISOLATION MODE: TÍNH NĂNG MỚI CÓ GIẢI QUYẾT ĐƯỢC BUG RÒ RỈ DỮ LIỆU MULTI-TENANT KHÔNG?"
date: 2026-07-28
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

<!-- # LAMBDA TENANT ISOLATION MODE: TÍNH NĂNG MỚI CÓ GIẢI QUYẾT ĐƯỢC BUG RÒ RỈ DỮ LIỆU MULTI-TENANT KHÔNG? -->

## Giới thiệu

Chào mọi người, nhóm mình đang làm một dự án chatbot hỏi-đáp tài liệu (RAG), phục vụ nhiều người dùng cùng lúc (multi-tenant): mỗi người đăng nhập qua Cognito, upload tài liệu riêng, và dữ liệu phải được cách ly tuyệt đối giữa các user. Backend chạy trên 1 Lambda function dạng Docker container duy nhất, phục vụ tất cả user.

Đây là bug nghiêm trọng nhất mình từng gặp khi làm dự án này: User A upload tài liệu, User B đăng nhập vào lại thấy được cả tài liệu của User A, và tệ hơn — khi User B xóa hết tài liệu của mình, User A cũng mất sạch dữ liệu theo. Sau khi tự điều tra và fix xong bug này bằng tay, mình mới biết AWS vừa công bố (tháng 11/2025) một tính năng mới cho đúng vấn đề này: **Lambda Tenant Isolation Mode**. Bài viết này kể lại quá trình debug thật, giải thích tính năng mới, và — quan trọng nhất — phân tích vì sao tính năng này **KHÔNG** thể tự động sửa được bug mình gặp phải, dù nghe tên có vẻ như vậy.

## Bug thật: Vì sao dữ liệu bị lẫn giữa các user?

Lambda có đặc tính tái sử dụng execution environment (container) giữa nhiều lần gọi để tăng tốc (tránh cold start) — nhưng mặc định, môi trường được tái sử dụng cho bất kỳ request nào của cùng 1 function, không phân biệt request đó đến từ user nào. Trong code backend của mình, có 2 chỗ vô tình dựa vào việc "container được giữ nguyên giữa các request":

- Một dict toàn cục (`state`) lưu `raw_documents`, `vector_store`… dùng chung cho mọi request trên cùng 1 container.
- FAISS vector index dùng path cố định `vectorstore/smartdoc_index` trên S3 — không có `user_id` trong path, nên mọi user đọc/ghi cùng 1 file index.

Khi 2 user khác nhau vô tình được route vào cùng 1 container đã ấm, họ dùng chung dữ liệu trong RAM lẫn dữ liệu trên S3.

## Lambda Tenant Isolation Mode giải quyết vấn đề gì?

Tính năng mới cho phép Lambda xử lý các lần gọi hàm trong các execution environment riêng biệt cho từng end-user/tenant, thay vì chia sẻ chung giữa mọi request của cùng 1 function.

Cách hoạt động: bạn truyền thêm tham số `tenant-id` (qua header `X-Amz-Tenant-Id` khi tích hợp API Gateway) trong mỗi lần invoke. Lambda dùng ID này để đảm bảo environment chỉ được tái sử dụng cho cùng 1 tenant — vẫn giữ được lợi ích warm start, nhưng không còn nguy cơ dữ liệu trong RAM/`/tmp` bị rò rỉ chéo giữa các tenant.

## Cách mình đã fix bug (trước khi biết tới tính năng này)

- `get_user_index_name(user_id)` — tính riêng path FAISS theo từng user: `f"{user_id}/{FAISS_INDEX_NAME}"`
- Mọi endpoint tài liệu/chat đọc lại dữ liệu tươi mới trực tiếp từ S3/FAISS mỗi request, không còn phụ thuộc biến `state` toàn cục
- `/api/upload-url`: đổi S3 key thành `uploads/{user_id}/{filename}` thay vì `uploads/{filename}`

## Vì sao Tenant Isolation Mode KHÔNG tự động sửa được bug này?

Đây là phần mình muốn nhấn mạnh, vì rất dễ bị hỏi "sao không đợi/dùng tính năng có sẵn cho nhàn":

Tenant Isolation Mode chỉ cách ly ở **compute layer** — execution environment, RAM, `/tmp`. Nó không và không thể can thiệp vào cách ứng dụng đặt tên key trên S3 hay DynamoDB. Nếu mình chỉ bật Tenant Isolation Mode mà không sửa đường dẫn S3 (`vectorstore/smartdoc_index` dùng chung), thì:

- Biến `state` trong RAM sẽ được cách ly đúng — không còn user này thấy cache của user kia trong cùng lần chạy.
- Nhưng khi Lambda (dù chạy trong environment riêng của tenant nào) đọc lại FAISS index từ S3 theo path cố định `vectorstore/smartdoc_index`, nó vẫn đọc đúng 1 file dùng chung đó — bug persistent-data vẫn còn nguyên, vì gốc rễ nằm ở **tầng data modeling (application logic)**, không phải tầng compute isolation.

Nói cách khác: Tenant Isolation Mode là lớp phòng thủ bổ sung rất tốt cho các lỗi tương tự trong tương lai (ví dụ: cache tạm trong `/tmp`, biến global vô tình dùng chung) — nhưng nó **không thay thế** được việc thiết kế đúng data path theo từng tenant ngay từ tầng ứng dụng. Fix đúng gốc vẫn phải sửa ở nơi bug thật sự nằm.

## Kết quả kiểm chứng

Sau khi fix thủ công (trước khi tính năng Tenant Isolation Mode ra mắt), mình test lại với 2 Cognito user thật:

- Mỗi user chỉ thấy đúng file của mình qua `/api/files`
- Hỏi AI về "mã số bí mật" trong tài liệu của người kia → AI trả lời "không có trong tài liệu"
- User B xóa hết file → User A vẫn còn nguyên dữ liệu

## Kết luận

Bug multi-tenancy là một trong những lớp lỗi khó phát hiện nhất trong serverless, vì nó chỉ xuất hiện khi 2 request tình cờ rơi vào cùng 1 warm container. Lambda Tenant Isolation Mode là một bước tiến lớn ở tầng compute, nhưng an toàn dữ liệu thật sự luôn bắt đầu từ **tầng thiết kế dữ liệu (data modeling)** — mọi resource per-tenant (S3 key, DynamoDB partition key, cache key…) đều cần chứa tenant/user identifier một cách tường minh, bất kể lớp compute bên dưới có cách ly tốt đến đâu.

## Link bài viết

Bài blog chính thức của AWS về Lambda Tenant Isolation Mode: https://aws.amazon.com/blogs/compute/building-multi-tenant-saas-applications-with-aws-lambdas-new-tenant-isolation-mode/

## Link bài viết trong nhóm AWS Study Group

https://www.facebook.com/groups/awsstudygroupfcj/permalink/2226685264763100/
