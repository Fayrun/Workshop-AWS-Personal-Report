---
title : "AWS Services Used"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.1.4 </b> "
---

This section lists in detail each AWS service used by SmartDocAI, its specific configuration, and the reasoning behind the choice — helping readers understand the role of each component in the overall architecture presented in the previous section.

### 1. Compute & API

| Service | Configuration | Role |
|---|---|---|
| **AWS Lambda** | Function `smartdocai`, Docker container image, region `us-east-1` | Runs the entire FastAPI backend (auth, upload, RAG, profile) in a serverless model — billed only when there is a request, auto-scales |
| **Amazon API Gateway** | HTTP API, stage `/prod`, endpoint `d60866voq5.execute-api...` | The single entry point for every request from the Frontend, proxying directly to Lambda |
| **Amazon ECR** | Repository `smartdocai`, Lifecycle Policy (expire untagged images after 1 day, keep up to 5 tagged images) | Stores the Lambda function's Docker image, automatically updated by CodePipeline on every deploy |

### 2. Storage

| Service | Configuration | Role |
|---|---|---|
| **Amazon S3 (Application Storage)** | Bucket `smartdocai-storage-623035187993`, Intelligent-Tiering, default SSE-S3 | Stores user documents, FAISS vector index, chat history, avatar — isolated by `user_id` |
| **Amazon S3 (Frontend Hosting)** | Bucket `aws-smartdocsai-cloud`, Static Website Hosting | Stores the React SPA build output (`index.html`, `assets/`), serving as the CloudFront origin |
| **Amazon DynamoDB** | Table `smartdocai-user-profiles`, Partition Key `user_id`, PAY_PER_REQUEST, SSE-KMS | Stores only the `avatar_url` field — other personal information is stored directly in Cognito attributes |

### 3. Identity & Security

| Service | Configuration | Role |
|---|---|---|
| **Amazon Cognito User Pool** | `us-east-1_3oq5wIiuu`, supports Email/Password + Google OAuth | Manages identity, issues JWT tokens, has built-in throttling/lockout to prevent brute-force attacks |
| **Cognito Hosted UI** | Domain `smartdocai-fayrun2026.auth.us-east-1.amazoncognito.com` | Intermediate screen handling the OAuth flow with Google, includes a `state` parameter for CSRF protection (implemented additionally at the application layer) |
| **Lambda PreSignUp Trigger** | Function `smartdocai-presignup-check` | Automatically links (`AdminLinkProviderForUser`) a Google account with an existing email/password account sharing the same email, preventing duplicate profiles |
| **AWS KMS** | Key `alias/aws/dynamodb` | Encrypts data in DynamoDB, enabling an audit trail via CloudTrail |

### 4. AI & Machine Learning

| Service | Specific Model | Role |
|---|---|---|
| **Amazon Bedrock (LLM)** | `qwen.qwen3-next-80b-a3b` | Generates answers for the 3 RAG modes (Standard, Self-RAG, Co-RAG) |
| **Amazon Bedrock (Embeddings)** | `amazon.titan-embed-text-v2:0` (1024 dimensions) | Converts text into vectors for indexing and semantic search, running 12 parallel threads when processing large documents |

> See the `system-status-live.png` image in section 5.1.3 — the real response confirmed both models are active: `"provider":"Amazon Bedrock","model":"qwen.qwen3-next-80b-a3b","embedding_provider":"Amazon Titan"`.

### 5. CDN & Delivery

| Service | Configuration | Role |
|---|---|---|
| **Amazon CloudFront** | Distribution `dutf3c70nnjzl.cloudfront.net`, origin = S3 frontend bucket | Distributes the React SPA over a global CDN network, reducing latency, enforcing HTTPS with TLS 1.2+ |

### 6. CI/CD

| Service | Configuration | Role |
|---|---|---|
| **AWS CodePipeline** | `smartdocai-be-pipeline` (backend), `smartdocsai-fe-pipeline` (frontend) | Automatically triggers on push to GitHub `main`, orchestrating Source → Build → Deploy |
| **AWS CodeBuild** | Project `smartdocai-be-build` | Installs dependencies, lints (flake8), runs pytest (hard-fail), builds the Docker image, pushes to ECR |
| **S3 (Artifact Store)** | Bucket `codepipeline-us-east-1-...` (automatically created by AWS) | Stores intermediate artifacts between the stages of both pipelines |

### 7. Automation & Monitoring

| Service | Configuration | Role |
|---|---|---|
| **Amazon EventBridge** | Rule `smartdocai-cleanup-unconfirmed`, rate 5 minutes | Automatically triggers Lambda to delete Cognito accounts that have not confirmed their email after 5 minutes, preventing them from occupying a username slot |
| **Amazon CloudWatch Logs** | Log group `/aws/lambda/smartdocai` | Records Lambda execution logs, supports debugging via Logs Insights queries |
| **Amazon CloudWatch Alarms** | 4 alarms: Lambda Errors, Duration, Throttles, API Gateway 5xx | Proactively detects abnormal errors/performance instead of waiting for user reports, sends alerts via SNS |
| **Amazon SNS** | Topic `smartdocai-alerts` | Sends an email alert whenever a CloudWatch Alarm switches to the ALARM state |

### 8. Summary Table by Architecture Layer

| Layer | AWS Services Used |
|---|---|
| **Presentation** | S3 (Frontend), CloudFront |
| **Application** | API Gateway, Lambda, ECR, EventBridge |
| **Data** | S3 (Storage), DynamoDB, Cognito |
| **AI** | Bedrock (LLM + Embeddings) |
| **CI/CD** | CodePipeline, CodeBuild, S3 (Artifact) |
| **Security & Monitoring** | KMS, CloudWatch (Logs + Alarms), SNS |
