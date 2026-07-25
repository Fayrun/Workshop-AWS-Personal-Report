---
title : "Creating Amazon S3 for document storage"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

### 1. Declare Bucket Name and Region

![image5.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image5.png)

---

### 2. Run Command to Create S3 Bucket

![image6.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image6.png)

---

### 3. Configure CORS Security Hardening (Allow Direct Frontend Uploads/Downloads)

To enforce strict security and protect against unauthorized Cross-Origin requests, the S3 CORS policy replaces wildcard `AllowedOrigins: ["*"]` with an explicit list of trusted domains:
- CloudFront CDN Distribution: `https://dutf3c70nnjzl.cloudfront.net`
- Local Development environment: `http://localhost:5173` and `http://localhost:5174`

#### 3.1. CORS Policy File (`cors.json`):

```json
{
  "CORSRules": [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
      "AllowedOrigins": [
        "https://dutf3c70nnjzl.cloudfront.net",
        "http://localhost:5173",
        "http://localhost:5174"
      ],
      "ExposeHeaders": ["ETag"]
    }
  ]
}
```

#### 3.2. Apply CORS Policy Command:

```bash
aws s3api put-bucket-cors --bucket smartdocai-storage-623035187993 --cors-configuration file://cors.json
```

---

### 4. Configure IAM Role Permissions for AWS Lambda

Ensure the IAM Role running the Lambda Function (e.g., `smartdocai-lambda-role`) has attached S3 access policy.

Run command to attach `AmazonS3FullAccess` policy to the Lambda Role:

![image9.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image9.png)

**Result:**

![image10.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image10.png)

---

### 5. Amazon S3 Storage Directory Structure (Per-User Isolation)

The system stores original documents, vector indices, and JSON metadata directly on S3 partitioned by `user_id`:

![image11.png](/images/5-Workshop/5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/image11.png)

| No. | Directory (Prefix) | File Type | Description & Function |
| --- | --- | --- | --- |
| 1 | `uploads/{user_id}/` | `.pdf`, `.docx` | Stores original document files uploaded by users from Frontend via S3 Presigned URLs. |
| 2 | `vectorstore/{user_id}/smartdoc_index/` | `index.faiss`, `index.pkl` | Stores per-user FAISS Vector DB indices converted from uploaded documents for RAG search. |
| 3 | `processed_files/` | `{user_id}.json` | Stores per-user document lists & file metadata (replacing DynamoDB) displayed on the Frontend. |
| 4 | `chat_history/` | `{user_id}.json` | Stores complete per-user RAG Q&A chat history, AI answers, and source citations (replacing DynamoDB). |
| 5 | `avatars/{user_id}/` | `.jpg`, `.png` | Stores user avatar images when image size > 300KB (if < 300KB, base64 is stored in DynamoDB). |
| 6 | `metadata/` | `search_config.json` | Stores general RAG settings (`chunk_size`, `chunk_overlap`, Reranker/Self-RAG/Co-RAG toggle states). |
