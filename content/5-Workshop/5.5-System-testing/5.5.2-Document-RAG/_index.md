---
title : "Document Upload & RAG Testing"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2 </b> "
---

This section tests SmartDocAI's core business flow: uploading documents via S3 presigned URL (bypassing the API Gateway 10MB limit), indexing vectors using Bedrock Titan Embeddings, and querying RAG in 3 different modes (Standard, Self-RAG, Co-RAG). This is the most important feature of the system, so it is tested thoroughly for both functionality and performance.

### 1. Test Case Summary Table

| # | Test Case | Endpoint | Input | Expected Result | Status |
|---|-----------|----------|-------|-----------------|--------|
| **PRESIGNED URL** |
| 2.1 | Get S3 presigned URL | POST /api/upload-url | JWT token + `{"filename":"test.pdf","content_type":"application/pdf"}` | 200 OK<br/>Return: upload_url (1h expiry) + s3_key | PASS |
| 2.2 | Presigned URL requires auth | POST /api/upload-url | No JWT token | 401 Unauthorized<br/>Error: "Missing Authorization Bearer Token" | PASS |
| **S3 UPLOAD** |
| 2.3 | Upload PDF to S3 | PUT to presigned URL | PDF file + Content-Type: application/pdf | 200 OK (direct S3 upload, bypass Lambda) | PASS |
| 2.4 | CORS allowed origin | PUT to presigned URL | Origin: https://dutf3c70nnjzl.cloudfront.net | 200 OK + CORS headers | PASS |
| 2.5 | CORS blocked origin | PUT to presigned URL | Origin: https://evil.com | 403 Forbidden (no CORS header) | PASS |
| **DOCUMENT INDEXING** |
| 2.6 | Index uploaded document | POST /api/process | JWT + `{"filename":"...","s3_key":"..."}` | 200 OK<br/>Return: status + page count + chunk count | PASS |
| 2.7 | Verify backend processing | Check CloudWatch logs | After calling POST /api/process | Log: extract text → chunk splitting → generate vectors via Bedrock Titan (12 parallel threads) | PASS |
| **RAG QUERY - STANDARD MODE** |
| 2.8 | Standard RAG query | POST /api/chat | JWT + `{"message":"...","use_self_rag":false,"use_co_rag":false}` | 200 OK<br/>Return: answer + sources | PASS |
| 2.9 | Query without documents | POST /api/chat | User has not uploaded any document | 200 OK<br/>Return: notice that no document is available | PASS |
| **RAG QUERY - SELF-RAG MODE** |
| 2.10 | Self-RAG with relevance grading | POST /api/chat | `{"message":"...","use_self_rag":true,"use_co_rag":false}` | 200 OK<br/>Return: answer + sources filtered by relevance | PASS |
| 2.11 | Self-RAG filters low relevance | POST /api/chat | Question unrelated to the documents | Return: empty sources + notice that no relevant context was found | PASS |
| **RAG QUERY - CO-RAG MODE** |
| 2.12 | Co-RAG collaborative retrieval | POST /api/chat | `{"message":"...","use_self_rag":false,"use_co_rag":true}` | 200 OK<br/>Return: answer + sources (3 parallel agents: semantic/keyword/conceptual) | PASS |

**Base API endpoint:** `https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod`

**Backend processing steps (document indexing):**
1. Download file from S3 → 2. Parse PDF (pypdf2) → 3. Split text into chunks (500 tokens, overlap 50) → 4. Generate embeddings (Bedrock Titan V2, 1024 dimensions) → 5. Store vectors in FAISS (in-memory, isolated per user)

**CORS configuration for S3:**
- **Allowed origins:** CloudFront (dutf3c70nnjzl.cloudfront.net), localhost:5173, localhost:5174
- **Allowed methods:** GET, PUT, POST, DELETE, HEAD
- **Max Age:** 3600 seconds

**Performance comparison between RAG modes (measured on production):**

Tested with the same question over 3 files / 94 pages / 143 chunks already uploaded:

| Metric | Standard RAG | Self-RAG | Co-RAG |
|--------|-------------|----------|--------|
| Response time | 13.89s | 13.43s | 17.39s |
| Number of sources returned | 10 | 10 | 10 |
| Number of parallel processing agents | 1 | 1 (rewrite + grade) | 3 (semantic/keyword/conceptual) |

---
