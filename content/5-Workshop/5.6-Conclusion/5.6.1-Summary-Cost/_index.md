---
title : "Workshop Summary & Cost"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

### What Was Accomplished

#### 1. **5.1 - Introduction & Architecture**
- Specified the Frontend architecture and the Backend & RAG Pipeline architecture
- Drew the overall architecture diagram on AWS
- Listed all the AWS services used
- Described the system's interface and functionality

#### 2. **5.2 - Preparation**
- Prepared the project source code
- Created and configured the AWS account
- Created a dedicated IAM User to manage access instead of using the root user

#### 3. **5.3 - Frontend Deployment**
- Created an S3 Bucket to store static files
- Enabled Static Website Hosting
- Configured Bucket Policy/Block Public Access
- Accelerated access with CloudFront
- Automated deployment with CodePipeline (GitHub → S3)

#### 4. **5.4 - Backend Deployment**
- Set up Cognito (user authentication)
- Set up DynamoDB (user profiles)
- Set up S3 (document storage + FAISS index)
- Packaged the RAG Engine running on Lambda (Docker container via ECR)
- Created API Gateway
- Integrated the Frontend calling the API through CloudFront
- Automated backend CI/CD with CodePipeline

#### 5. **5.5 - System Testing**
- Authentication (registration/login/JWT)
- Document Upload & RAG (3 modes: Standard/Self-RAG/Co-RAG)
- Security (input validation, CORS/JWT, CSRF OAuth state)
- Profile (CRUD profile + avatar)
- Monitoring (CloudWatch Logs/Insights/Alarms + SNS)
- CI/CD (automated pytest, hard-fail mechanism)

### What We Learned

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
- **Actual cost:** ~$1.65 over the 30-day development period (06/27 - 07/26/2026) — see details below

---

## Total Cost

### Actual Cost (from AWS Cost Explorer, 06/27/2026 - 07/26/2026, 30 days)

The figures below come directly from AWS Cost Explorer, covering exactly **30 days** during the development + workshop testing period:

| Service | Actual Cost (USD) | Share |
|---------|------------------------|----------|
| Amazon ECR (Docker image storage) | $0.5977 | 36.3% |
| AWS CodePipeline | $0.4440 | 26.9% |
| Amazon Bedrock (LLM + Embeddings) | $0.3453 | 20.9% |
| AWS CodeBuild | $0.2250 | 13.7% |
| Amazon S3 | $0.0197 | 1.2% |
| Amazon API Gateway | $0.0140 | 0.9% |
| Amazon DynamoDB | $0.0021 | 0.1% |
| Amazon SES | $0.0003 | <0.1% |
| AWS Secrets Manager | $0.00002 | <0.1% |
| Amazon CloudFront | $0.0000052 | ~0% |
| AWS Lambda | $0.00 (Free Tier) | 0% |
| Amazon Cognito | $0.00 (Free Tier) | 0% |
| Amazon EventBridge | $0.00 (Free Tier) | 0% |
| Amazon CloudWatch | $0.00 (Free Tier) | 0% |
| Amazon SNS | $0.00 (Free Tier) | 0% |
| **TOTAL (30 days)** | **~$1.6481** | 100% |

### Comparison with the Estimated Cost in Section 2 - Proposal

In [Section 2 - Proposal](/2-proposal/), the team estimated the total monthly infrastructure cost at around **$0.66 - $1.85 USD/month**. The actual cost over exactly 30 days is **~$1.65/month** — **falling within the originally estimated range**, showing the overall estimate was fairly accurate.

However, when comparing service by service, the actual cost breakdown deviates quite a bit from the original assumptions:

| Service | Estimate (Section 2) | Actual | Notes |
|---------|-------------------|---------|----------|
| **Amazon ECR** | $0.10 - $0.20 | $0.5977 | **The largest actual expense**, significantly higher than estimated due to being in an active development phase with frequent builds/pushes, not yet a stable operating phase |
| **CodePipeline & CodeBuild** | $0.05 - $0.15 | $0.6690 | **4-13x higher** than estimated, due to the development phase having many pushes/bug fixes/rebuilds (unlike the "~30 build minutes/month" assumption for an already-stable system) |
| **Amazon Bedrock** | $0.25 - $0.95 | $0.3453 | Within the estimated range |
| **Amazon S3** | $0.15 - $0.30 | $0.0197 | Much lower than estimated — actual test document volume is still small |
| **API Gateway** | $0.01 - $0.05 | $0.0140 | Within the estimated range |
| **DynamoDB** | $0.00 (Free Tier) | $0.0021 | Nearly matches, still essentially free |
| **Lambda, Cognito, CloudFront, EventBridge, CloudWatch** | $0.00 - $0.10 | $0.00 | Free Tier performed exactly as expected |
| **Amazon SES, Secrets Manager** | *(not in the estimate table)* | $0.0003 + $0.00004 | Small, negligible amounts, but show 2 extra services being used beyond the originally listed architecture |

**Conclusion:** The total actual cost matches the original estimate fairly closely, but **CI/CD (CodePipeline + CodeBuild + ECR) turns out to be the largest cost group in practice (~77% of total cost)** instead of Bedrock/S3 as originally predicted — which makes sense since this is an active development phase (many builds/deploys), not a stable operating phase.

---

**Notes:**
- Actual cost during the development/workshop phase: **under $1.7** (very low)
- Free Tier: Lambda, Cognito, EventBridge, CloudWatch, and SNS all reached $0.00 thanks to usage staying within the free tier limits
- The largest actual cost group: the CI/CD pipeline (ECR + CodePipeline + CodeBuild), not AI/Bedrock as originally predicted
- Production cost with higher traffic and more frequent builds/deploys could be significantly higher — should be tracked via CloudWatch Budget Alerts

