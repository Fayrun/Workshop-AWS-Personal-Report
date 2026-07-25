---
title : "Kiểm thử tải lên tài liệu & RAG"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2 </b> "
---

Phần này kiểm tra luồng nghiệp vụ cốt lõi của SmartDocAI: tải tài liệu lên qua S3 presigned URL (bỏ qua giới hạn 10MB của API Gateway), lập chỉ mục vector bằng Bedrock Titan Embeddings, và truy vấn RAG ở 3 chế độ khác nhau (Standard, Self-RAG, Co-RAG). Đây là tính năng quan trọng nhất của hệ thống nên được kiểm thử kỹ cả về chức năng lẫn hiệu năng.

### 1. Bảng tổng hợp test case

| # | Test Case | Endpoint | Input | Expected Result | Status |
|---|-----------|----------|-------|-----------------|--------|
| **PRESIGNED URL** |
| 2.1 | Get S3 presigned URL | POST /api/upload-url | JWT token + `{"filename":"test.pdf","content_type":"application/pdf"}` | 200 OK<br/>Return: upload_url (1h expiry) + s3_key | PASS |
| 2.2 | Presigned URL requires auth | POST /api/upload-url | No JWT token | 401 Unauthorized<br/>Error: "Thiếu Authorization Bearer Token" | PASS |
| **S3 UPLOAD** |
| 2.3 | Upload PDF to S3 | PUT to presigned URL | PDF file + Content-Type: application/pdf | 200 OK (direct S3 upload, bypass Lambda) | PASS |
| 2.4 | CORS allowed origin | PUT to presigned URL | Origin: https://dutf3c70nnjzl.cloudfront.net | 200 OK + CORS headers | PASS |
| 2.5 | CORS blocked origin | PUT to presigned URL | Origin: https://evil.com | 403 Forbidden (no CORS header) | PASS |
| **DOCUMENT INDEXING** |
| 2.6 | Index uploaded document | POST /api/process | JWT + `{"filename":"...","s3_key":"..."}` | 200 OK<br/>Return: status + số trang + số chunks | PASS |
| 2.7 | Verify backend processing | Check CloudWatch logs | Sau khi gọi POST /api/process | Log: trích xuất văn bản → chia chunk → tạo vector qua Bedrock Titan (12 threads song song) | PASS |
| **RAG QUERY - STANDARD MODE** |
| 2.8 | Standard RAG query | POST /api/chat | JWT + `{"message":"...","use_self_rag":false,"use_co_rag":false}` | 200 OK<br/>Return: answer + sources | PASS |
| 2.9 | Query without documents | POST /api/chat | User chưa upload tài liệu nào | 200 OK<br/>Return: thông báo chưa có tài liệu | PASS |
| **RAG QUERY - SELF-RAG MODE** |
| 2.10 | Self-RAG with relevance grading | POST /api/chat | `{"message":"...","use_self_rag":true,"use_co_rag":false}` | 200 OK<br/>Return: answer + sources đã lọc theo relevance | PASS |
| 2.11 | Self-RAG filters low relevance | POST /api/chat | Câu hỏi không liên quan tới tài liệu | Return: sources rỗng + thông báo không tìm thấy ngữ cảnh liên quan | PASS |
| **RAG QUERY - CO-RAG MODE** |
| 2.12 | Co-RAG collaborative retrieval | POST /api/chat | `{"message":"...","use_self_rag":false,"use_co_rag":true}` | 200 OK<br/>Return: answer + sources (3 agent song song: semantic/keyword/conceptual) | PASS |

**Điểm cuối API gốc:** `https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod`

**Các bước xử lý ở Backend (lập chỉ mục tài liệu):**
1. Tải file từ S3 → 2. Parse PDF (pypdf2) → 3. Chia nhỏ văn bản (500 tokens, overlap 50) → 4. Sinh embeddings (Bedrock Titan V2, 1024 chiều) → 5. Lưu vector vào FAISS (in-memory, cô lập theo từng user)

**Cấu hình CORS cho S3:**
- **Origin được phép:** CloudFront (dutf3c70nnjzl.cloudfront.net), localhost:5173, localhost:5174
- **Method được phép:** GET, PUT, POST, DELETE, HEAD
- **Max Age:** 3600 giây

**So sánh hiệu năng giữa các chế độ RAG (đo thật trên production):**

Test với cùng 1 câu hỏi trên tài liệu 3 file/94 trang/143 chunks đã upload sẵn:

| Metric | Standard RAG | Self-RAG | Co-RAG |
|--------|-------------|----------|--------|
| Thời gian phản hồi | 13.89s | 13.43s | 17.39s |
| Số sources trả về | 10 | 10 | 10 |
| Số agent xử lý song song | 1 | 1 (rewrite + grade) | 3 (semantic/keyword/conceptual) |

---