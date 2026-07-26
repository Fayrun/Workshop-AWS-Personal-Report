---
title : "Workshop Summary & Cost"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

### What we learned

#### 1. **Serverless Architecture & Microservices**
- Deployed a FastAPI application to Lambda using a Docker container
- Used API Gateway as the public HTTP endpoint
- Leveraged CloudFront CDN for static assets and API caching

#### 2. **Authentication & Authorization**
- Integrated a Cognito User Pool for user management
- Implemented JWT-based authentication
- Supported multiple login methods (Email/Password + Google OAuth)
- Isolated data per user using S3 prefixes + DynamoDB partition keys

#### 3. **AI & Machine Learning Integration**
- Used Amazon Bedrock (Qwen3-Next 80B-A3B LLM + Titan Embeddings V2)
- Built a RAG (Retrieval-Augmented Generation) pipeline
- Performed vector search with FAISS (in-memory database)
- Implemented 3 RAG modes: Standard, Self-RAG, Co-RAG

#### 4. **CI/CD Pipeline**
- Set up CodePipeline to trigger automatically on GitHub push
- Integrated pytest unit tests into CodeBuild (hard fail)
- Deployed using Docker with an ECR registry
- Automatically updated the Lambda function

#### 5. **Automation & Monitoring**
- EventBridge rule scheduled for cleanup tasks
- CloudWatch Logs + Insights to monitor the application
- Lambda metrics (invocations, duration, errors, throttles)
- Real-time log viewing (tailing) and querying
- **CloudWatch Alarms (4 alarms) + SNS Topic Alerting** — proactively detects abnormal errors/performance (Lambda Errors, Duration, Throttles, API Gateway 5xx) and sends email alerts via SNS, instead of waiting for user reports

#### 6. **Security Best Practices**
- Input data validation (phone, DOB, fullname, XSS prevention)
- CORS restriction (no wildcard `*`)
- HTTPS only (TLS 1.2+)
- JWT expiration + signature verification
- Security audit with documented limitations

#### 7. **Cost Optimization**
- Pay-per-use model with serverless services
- DynamoDB on-demand billing (no cost while idle)
- S3 presigned URLs (bypassing Lambda for uploads)
- CloudFront Free Tier (1 TB/month)
- **Estimated cost:** $7-15/month for moderate usage

---

## Total Cost

### Estimated Cost (assuming usage during the workshop period)

Assuming the workshop ran for **7 days** with a moderate level of testing:

| Service | Usage Level | Cost (7 days) | Estimated Cost (monthly) |
|---------|-------|------------------|---------------------------|
| Lambda | 5K invocations, 2GB RAM, 5s avg | $0.20 | $0.85 |
| API Gateway | 5K requests | $0.02 | $0.08 |
| S3 (Frontend) | 2 GB storage, 5 GB transfer | $0.10 | $0.35 |
| S3 (Storage) | 10 GB storage, 10 GB transfer | $0.35 | $1.20 |
| DynamoDB | 2K reads, 1K writes | $0.05 | $0.15 |
| Cognito | 50 users (MAU) | Free | Free |
| CloudFront | 20 GB transfer (Free tier) | $0.00 | $0.00 |
| Bedrock (LLM) | 200K input tokens, 50K output tokens | $0.13 | $0.45 |
| Bedrock (Embeddings) | 1M tokens | $0.10 | $0.35 |
| CodePipeline | 1 active pipeline | Free | Free |
| CodeBuild | 5 builds, 5 min each | $0.02 | $0.08 |
| ECR | 1.5 GB storage | $0.15 | $0.20 |
| EventBridge | 2016 events (every 5min × 7 days) | Free | Free |
| CloudWatch Logs | 0.5 GB ingested | $0.25 | $1.00 |
| **TOTAL** | | **~$1.37** | **~$4.71** |

> **Note:** The table above is **estimated data** based on assumed usage levels, not actual figures pulled from AWS Cost Explorer/Billing (root/billing access is required to view them). Real figures will be updated once access is available, along with a Cost Explorer screenshot for comparison.

**Notes:**
- Workshop cost: **Under $2** (very low)
- Free Tier: Cognito, EventBridge, CodePipeline, CloudFront (1 TB) are all free
- Lambda provisioned concurrency (optional): +$4/month if enabled
- Production cost with higher traffic: $15-50/month
