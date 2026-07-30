---
title : "Overall Architecture on AWS"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.1.3 </b> "
---

After exploring the Frontend and Backend architecture separately in the previous two sections, this section brings everything together into a **complete picture** of how the AWS components work together to form the full SmartDocAI system, along with the per-user data storage structure.

### 1. Overall Architecture Diagram

![Overall Architecture Diagram](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/architecture-diagram.png)

**Flow legend (numbered in the diagram):**

| # | Flow | # | Flow |
|---|---|---|---|
| 1 | Users → CloudFront | 9 | GitHub Repository → CodePipeline (Backend) & (Frontend) |
| 2 | CloudFront → S3 Frontend Bucket | 10 | CodePipeline (Backend) → CodeBuild (pytest) |
| 3 | CloudFront → API Gateway (proxy `/api/*`) | 11 | CodeBuild (pytest) → Amazon ECR |
| 4 | API Gateway → Lambda | 12 | ECR → Lambda (deploy new container) |
| 5 | Lambda → Cognito User Pool (validate JWT) | 13 | CodePipeline (Frontend) → AWS CodeBuild |
| 6 | Lambda → Data Storage (DynamoDB + S3) | 14 | AWS CodeBuild (Frontend) → S3 Frontend Bucket |
| 7 | Lambda → Amazon Bedrock (LLM + Embeddings) | 15 | Cognito ↔ Google Identity Provider (OAuth) |
| 8 | EventBridge → Lambda (cleanup every 5 minutes) | 16 | Cognito ↔ Lambda presignup-check (merge account) |

SmartDocAI is built on a **Serverless Container Architecture** combined with **Managed Identity (Cognito)**, consisting of the following main components:

| Component | AWS Service | Specific Value |
|---|---|---|
| Frontend hosting | S3 + CloudFront | `https://dutf3c70nnjzl.cloudfront.net` |
| Backend API | Lambda + API Gateway | `https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod` |
| Authentication | Cognito User Pool | `us-east-1_3oq5wIiuu` |
| Cognito Hosted UI | Cognito | `https://smartdocai-fayrun2026.auth.us-east-1.amazoncognito.com` |
| PreSignUp trigger | Lambda | `smartdocai-presignup-check` |
| Profile database | DynamoDB | `smartdocai-user-profiles` (SSE-KMS) |
| File & Index storage | S3 | `smartdocai-storage-623035187993` (Intelligent-Tiering) |
| LLM | Bedrock | `qwen.qwen3-next-80b-a3b` |
| Embeddings | Bedrock | `amazon.titan-embed-text-v2:0` (1024 dimensions) |
| CI/CD | 2 separate CodePipelines | `smartdocai-be-pipeline` (Backend), `smartdocsai-fe-pipeline` (Frontend) |
| Scheduled task | EventBridge | Rule to clean up unconfirmed users (rate 5 minutes) |
| Monitoring | CloudWatch + SNS | Alarms for Lambda Errors/Duration/Throttles + API Gateway 5xx, email alerting (see section 5.5.5) |
| Access & encryption | IAM + KMS | IAM Roles for Lambda/CodeBuild, KMS key encrypting DynamoDB |

### 2. Data Storage Structure

![Data Storage Structure](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/storage-structure.png)

All data is designed to be **isolated per user** (`user_id` = Cognito `sub`), preventing cross-account data leaks:

- `uploads/{user_id}/` — original documents (PDF/DOCX) uploaded by the user
- `vectorstore/{user_id}/` — per-user FAISS index (1024 dimensions) for semantic search
- `chat_history/{user_id}.json` — RAG conversation history
- `processed_files/{user_id}.json` — list of documents that have finished processing
- DynamoDB `smartdocai-user-profiles` (partition key `user_id`) — stores only the `avatar_url` field (other personal information such as full name, phone number, date of birth is stored directly in Cognito attributes to avoid data mismatch between two storage locations)

### 3. Backend Modularization (Lambda Modules)

![Backend Modularization Lambda Modules](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/lambda-modules.png)

`app_api.py` serves as the main entry point (FastAPI + Mangum adapter), routing requests to specialized modules under `modules/`: `auth_service.py` (authentication), `document_processor.py` + `vector_store.py` (document processing & indexing), `rag_chain.py` + `self_rag.py` + `co_rag.py` (3 RAG modes), `profile_service.py` (user profile).

### 4. CI/CD Pipeline

![CI/CD Pipeline](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/cicd-pipeline.png)

The system has **2 separate CodePipelines**, both receiving code from the same GitHub repository:

- **CodePipeline (Backend)** `smartdocai-be-pipeline`: Every time code is pushed to the `main` branch, it automatically triggers CodeBuild: install dependencies → lint with flake8 → run pytest (hard-fail if tests do not pass) → build Docker image → push to ECR → update the Lambda function. This mechanism ensures that broken code cannot make it into production.
- **CodePipeline (Frontend)** `smartdocsai-fe-pipeline`: has 3 stages, **Source → Build → Deploy**. The Build stage uses **AWS CodeBuild** (project `smartdocsai-fe-build`, driven by a `buildspec.yml`) to install dependencies and build the React/Vite app, packaging the output as `smartdocsai-fe.zip`; the Deploy stage then pushes that file straight to the S3 Frontend Bucket (auto-extracted). When the pipeline was created, the Test stage was set to **Skip test stage** since the frontend doesn't have its own automated test suite yet, unlike the Backend pipeline where pytest is a mandatory gate.
