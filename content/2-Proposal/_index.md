---
title: "Proposal"
date: 2026-07-23
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# SMARTDOCAI - INTELLIGENT DOCUMENT KNOWLEDGE EXTRACTION & Q&A PLATFORM ON AWS SERVERLESS

---

### 1. PROJECT OVERVIEW

**SmartDocAI** is an intelligent web platform solution integrating an AI Assistant powered by **AWS Serverless Container Architecture** and cutting-edge **Generative AI / Large Language Models (LLMs)** on **AWS Bedrock**.

The project addresses the need for automated searching, extraction, and synthesis of knowledge from enterprise and personal text documents (PDF, DOCX). The system leverages **Retrieval-Augmented Generation (RAG)** with 3 flexible processing modes (Standard RAG, Self-RAG, Co-RAG Multi-Agent), guaranteeing high answer accuracy, transparent source citations, and elimination of LLM hallucination.

- **Project Name:** SmartDocAI (AWS Deployment)
- **Primary Architecture:** Serverless Container Architecture (FastAPI + React SPA + AWS Services)
- **Repository:** https://github.com/TakunKenjo/SmartdocAI-AWS
- **Deployment Environment:** AWS Region `us-east-1`
- **Production Domains:** 
  - Frontend CDN (CloudFront): `https://dutf3c70nnjzl.cloudfront.net`
  - Backend API Gateway: `https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod`

---

### 2. PROJECT OBJECTIVES

1. **Build a Complete Serverless RAG System (100% Serverless):**
   - Operate entirely on Serverless infrastructure (AWS Lambda, S3, DynamoDB, Bedrock, API Gateway, CloudFront) enabling automatic scaling with traffic and achieving optimal pay-as-you-go cost efficiency.

2. **Optimize Knowledge Retrieval & Q&A (Advanced RAG):**
   - Support 3 advanced Q&A modes:
     - **Standard RAG:** Rapid semantic analysis via FAISS vector store, keyword search using BM25, and Hybrid Search combining both methods to optimize retrieval performance.
     - **Self-RAG:** Automatic query restructuring, context noise filtering, and self-reflection grounding checks prior to responding.
     - **Co-RAG (Multi-Agent RAG):** Parallel collaboration across 3 independent Agents (Semantic, Keyword, Conceptual) with voting-based result merging for complex queries.

3. **Strict Security & Per-User Data Isolation:**
   - Ensure each user maintains isolated document storage (`uploads/{user_id}/`), dedicated vector index (`vectorstore/{user_id}/`), and private chat history.
   - Standardize authentication via **AWS Cognito User Pool**, supporting native Email/Password login and Google OAuth 2.0. Integrate a PreSignUp Lambda trigger to automatically link matching email accounts (`AdminLinkProviderForUser`) and protect against OAuth CSRF attacks using UUID `state` parameters.

4. **CI/CD Automation & Production Hardening:**
   - Build automated testing and deployment pipelines using **AWS CodePipeline** & **AWS CodeBuild** (integrated pytest suite with ~60 test cases and a hard-fail policy).
   - Cost optimization and operational monitoring: S3 Intelligent-Tiering, DynamoDB KMS Encryption at-rest, EventBridge Auto-Cleanup for unconfirmed users, CloudWatch Alarms & SNS Alerting.

---

### 3. PROBLEM STATEMENT

#### 3.1. Current State & Challenges

- **Time-consuming manual search:** In enterprises and research organizations, manually scanning thousands of pages of PDF/Word documents consumes excessive time and risks missing critical insights.
- **Limitations of traditional LLMs:** Standard AI chat tools lack access to internal document stores and frequently generate inaccurate answers (hallucinations) due to lack of verifiable reference sources.
- **Data privacy & isolation risks:** Many third-party SaaS tools fail to guarantee user data isolation, posing risks of leaking sensitive information.
- **Expensive 24/7 server infrastructure:** Maintaining fixed GPU/EC2 server clusters incurs significant ongoing costs even during idle periods without traffic.
- **Payload bottlenecks with large uploads:** Traditional API Gateway setups restrict request sizes (10 MB), causing failures when users upload large files.

---

### 4. SOLUTION ARCHITECTURE

#### Overall AWS System Architecture Diagram

![SmartDocAI Overall Architecture Diagram on AWS](/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/architecture-diagram.png)

---

#### Detailed Breakdown of AWS Services Used

| AWS Service | Service Type | Role & Function in SmartDocAI System |
|---|---|---|
| **AWS CloudFront** | Content Delivery Network (CDN) | Distributes the React + Vite Frontend SPA globally over HTTPS (`https://dutf3c70nnjzl.cloudfront.net`), optimizing page load speeds and reducing latency. |
| **AWS Cognito** | Identity & Access Management | Manages user registration/login (User Pool `us-east-1_3oq5wIiuu`), issues JWT Tokens, integrates Hosted UI **Google OAuth 2.0**, and triggers PreSignUp account merging (`AdminLinkProviderForUser`). |
| **Amazon API Gateway** | RESTful API Gateway | Centralized REST API endpoint (`https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod`), receives Frontend HTTP Requests, authenticates with Cognito Authorizer, and securely proxies (AWS_PROXY) to AWS Lambda. Handles CORS preflight (`OPTIONS`). |
| **AWS Lambda** | Serverless Compute | Serverless execution environment running FastAPI backend containerized via Docker (Memory: 3008 MB, Timeout: 300s). Handles all RAG logic, embedding, Bedrock calls, S3 and DynamoDB I/O. |
| **Amazon ECR** | Container Registry | Stores Docker Container Images for the FastAPI Backend. Serves as the source image repository for Lambda auto-updates via CI/CD pipelines. |
| **Amazon S3** | Object Storage | Stores raw document files (`uploads/{user_id}/`), FAISS Vector Store indexes (`vectorstore/{user_id}/`), chat history (`chat_history/`), and user avatars. Configured with **S3 Intelligent-Tiering** for long-term cost optimization and issues **Presigned URLs** for secure uploads. |
| **Amazon DynamoDB** | NoSQL Database | Stores detailed user profile records (`smartdocai-user-profiles`) including: Full Name, Email, Phone, DOB, subscription plan, document quotas. Guarantees single-digit millisecond latency with On-Demand capacity. |
| **AWS KMS** | Key Management Service | Encrypts data at-rest on Amazon DynamoDB using **AWS Managed KMS Keys** (`alias/aws/dynamodb`), protecting user personal information. |
| **Amazon Bedrock** | Generative AI Platform | AI analysis and text generation platform: <br>- **Titan Embeddings V2 (`amazon.titan-embed-text-v2:0`):** Generates 1024-dimensional embeddings (12 parallel threads).<br>- **Qwen 3 Next 80B A3B (`qwen.qwen3-next-80b-a3b`):** LLM reasoning engine for RAG synthesis and cited answers. |
| **AWS CodePipeline** | CI/CD Orchestration | Automates continuous deployment workflows. Automatically triggers on GitHub `main` pushes, orchestrating CodeBuild for testing and deployment. |
| **AWS CodeBuild** | Automated Build & Test | Container build execution environment. Runs linters (`flake8`) and automated unit tests (~60 test cases with `pytest`) under a **Hard Fail** policy before building Docker Images and updating Lambda. |
| **Amazon EventBridge** | Event Bus & Scheduling | Scheduled cron job invoking Lambda every 5 minutes to clean up unconfirmed user registrations older than 5 minutes (`smartdocai-cleanup-unconfirmed`). |
| **Amazon CloudWatch** | Monitoring & Observability | Collects system logs (CloudWatch Logs) and monitors 4 critical health alarms: Lambda Errors > 0, Duration > 25s, Throttles > 0, API Gateway 5xx Errors > 0. |
| **Amazon SNS** | Push Notification Service | Emergency incident alerting channel (`smartdocai-alerts`). When any CloudWatch Alarm trips, SNS immediately sends email alerts to administrators. |

---

### 5. TIMELINE

The project was successfully executed across 6 main phases from June 2026 to August 2026:

| Phase | Timeline | Primary Work Scope | Status |
|---|---|---|---|
| **Phase 1: Research & Design** | 22/06/2026 - 27/06/2026 | - Analyze RAG requirements.<br>- Survey LLMs on AWS Bedrock (Titan V2, qwen3-next-80b-a3b).<br>- Design Serverless Container architecture & Per-User Isolation.<br>- Design UI mockups. | Completed |
| **Phase 2: Backend Core & Auth** | 29/06/2026 - 04/07/2026 | - Build FastAPI Backend, integrate FAISS vector store & Bedrock API.<br>- Configure Cognito User Pool, Hosted UI & PreSignUp Lambda trigger.<br>- Implement 3-step document upload via S3 Presigned URLs. | Completed |
| **Phase 3: Advanced RAG & Frontend** | 06/07/2026 - 11/07/2026 | - Complete React + Vite SPA frontend, set up static website hosting.<br>- Implement User Profile management on DynamoDB. | Completed |
| **Phase 4: Production Hardening** | 13/07/2026 - 18/07/2026 | - Connect CloudFront CDN.<br>- Build CodePipeline/CodeBuild CI/CD with ~60 unit test cases (hard-fail).<br>- Deploy EventBridge automated unconfirmed account cleanup.<br>- Production Hardening: DynamoDB KMS Encryption, OAuth CSRF State UUID, S3 Intelligent-Tiering, CloudWatch Alarms & SNS Topic Alerts. | Completed |
| **Phase 5: System Testing & Optimization** | 20/07/2026 - 25/07/2026 | - Comprehensive End-to-End System Testing, latency measurement, and AWS Lambda cold start mitigation.<br>- Optimize RAG strategies (Self-RAG, Co-RAG, Re-ranking) and source citations.<br>- Handle edge cases, sync per-user data, and polish UI/UX experience. | Completed |
| **Phase 6: Documentation & Workshop Report** | 27/07/2026 - 01/08/2026 | - Build Workshop report documentation on Hugo platform (Bilingual Vietnamese - English).<br>- Document detailed AWS Backend, Cognito, DynamoDB, S3, Lambda, API Gateway architecture & UI specs.<br>- Wrap up results, project sign-off, and final documentation. | Completed |

---

### 6. BUDGET & COST ESTIMATION

The system maximizes **AWS Free Tier** benefits and **Serverless Pay-As-You-Go** pricing (only paying for actual resource consumption), reducing operational costs to an absolute minimum.

#### Estimated Monthly Infrastructure Cost Breakdown (Demo & Testing Scale)

| AWS Service | Estimated Monthly Usage | Estimated Cost (USD) |
|---|---|---|
| **AWS Lambda** | 100,000 requests, 3008 MB RAM | **$0.00** (Free Tier 1M requests & 400,000 GB-s) |
| **Amazon API Gateway** | 100,000 HTTP API calls | **$0.01 - $0.05** |
| **Amazon S3** | 10 GB storage (Standard + Intelligent-Tiering), 5,000 requests | **$0.15 - $0.30** |
| **Amazon DynamoDB** | < 1 GB data, On-Demand Read/Write | **$0.00** (Free Tier 25 WCU/RCU) |
| **Amazon Cognito** | < 1,000 MAUs (Monthly Active Users) | **$0.00** (Free Tier for 50,000 MAUs) |
| **AWS CloudFront** | < 50 GB Data Transfer Out | **$0.00** (Free Tier for 1 TB) |
| **AWS CodePipeline & CodeBuild** | ~30 build minutes / month | **$0.05 - $0.15** |
| **Amazon ECR** | ~1-2 GB Docker image storage | **$0.10 - $0.20** |
| **Amazon Bedrock - Titan Embeddings V2** | ~5,000,000 tokens / month ($0.00002 / 1k tokens) | **$0.10 - $0.20** |
| **Amazon Bedrock - Qwen 3 Next 80B A3B (`qwen.qwen3-next-80b-a3b`)** | ~1,000,000 input & 1,000,000 output tokens | **$0.15 - $0.75** |
| **EventBridge & CloudWatch Alarms** | 4 Alarms, 1 Rule, SNS email alerts | **$0.10** |
| **TOTAL ESTIMATED COST** | **Actual System Operation** | **~$0.66 - $1.85 USD / month** |

---

### 7. RISKS & MITIGATION STRATEGIES

#### Risk Matrix

| Technical / Operational Risk | Impact | Probability | Mitigation Strategy |
|---|---|---|---|
| **1. LLM Response Latency or Lambda Timeouts** | **High** | Medium | - Configure Lambda Memory to **3008 MB** and Timeout to **300 seconds**.<br>- Execute 12 parallel threads for Titan embeddings.<br>- Configured CloudWatch Alarm `smartdocai-lambda-duration` (>25s) for early alerts. |
| **2. AI Model Hallucination** | **Medium** | Medium | - Implement **Self-RAG** (context relevance check & self-reflection answer grounding).<br>- Implement **Co-RAG** merging results from 3 Agents via voting.<br>- Mandate verifiable source citations from original text. |
| **3. Unauthorized Access / Cross-User Data Leaks** | **Very High** | Low | - Enforce Cognito JWT Token verification on all API endpoints.<br>- Encrypt DynamoDB data at-rest with **AWS Managed KMS Keys**.<br>- Strict S3 data isolation using keys `uploads/{user_id}/` and `vectorstore/{user_id}/`. |
| **4. OAuth CSRF Attacks during Login** | **High** | Low | - Generate random UUID `state` parameters stored in `sessionStorage`.<br>- Perform two-layer independent verification at Frontend Callback and Cognito OAuth Server token validation. |
| **5. Unexpected Cost Spikes** | **Medium** | Low | - Apply **S3 Intelligent-Tiering** to auto-archive old documents to cheaper storage classes.<br>- Use AWS Managed Keys for DynamoDB KMS (zero key management fee).<br>- Configure CloudWatch Budget Alerts for threshold notifications. |
| **6. Broken Builds / Production Outages** | **High** | Low | - Set up CI/CD pipeline running linters and unit tests (~60 test cases) in `pre_build`.<br>- Enforce **Hard Fail** policy: any test failure immediately aborts build and blocks deployment. |

---

### 8. EXPECTED OPERATIONAL FUNCTIONALITIES

Upon completion, the **SmartDocAI** system delivers the following operational feature groups:

#### 1. Authentication & Account Security Features
- **Flexible 2-step registration:** Collects user details, sends a 6-digit OTP code via Email for account activation on AWS Cognito.
- **Multi-method login:** Supports native Email/Password login and fast Google SSO (OAuth 2.0 / OIDC via Cognito Hosted UI).
- **Automatic account linking:** PreSignUp Lambda Trigger automatically merges matching Google and Email/Password accounts (`AdminLinkProviderForUser`).
- **Automatic Username Resolution:** Automatically looks up real usernames when users log in via Email instead of Cognito internal usernames (`Google_...`), retrying seamlessly.
- **Session security:** Stores JWT Tokens securely in `sessionStorage`, protects against OAuth CSRF attacks with UUID `state` parameters, and encrypts DynamoDB data at-rest using AWS KMS Keys.

#### 2. Document Management & Ingestion Features
- **Direct document upload:** Allows drag & drop or file selection (PDF, DOCX). Uploads directly from Frontend to S3 via Presigned URLs.
- **Automated text extraction & indexing:** Automatically chunks text, generates embeddings with **Amazon Titan Embeddings V2**, and builds FAISS Vector Stores stored on S3 per user.
- **Document list management:** Displays processed files (format, page count, chunk count, file size), supporting individual file deletion or full vector store purging.

#### 3. AI Knowledge Search & Q&A Engine (Advanced RAG Engine)
- **3 Configurable Advanced RAG Modes:**
  - **Standard RAG:** Rapid response using **Hybrid Search** (60% FAISS Vector + 40% BM25 Keyword) and Cross-Encoder Re-ranking (`ms-marco-MiniLM-L-12-v2`).
  - **Self-RAG:** Automatic Query Rewriting, context noise filtering, and grounding score verification before returning answers.
  - **Co-RAG (Multi-Agent):** Coordinates 3 specialized Agents (Semantic, Keyword, Conceptual) and merges results via voting for complex queries.
- **File Filter:** Limits search scope to one or multiple specific documents selected by the user.
- **Transparent Source Citation Viewer:** Accompanies LLM answers (Qwen 3 Next 80B A3B on Bedrock) with original filename, page number, and text excerpt citations.
- **Thinking Process Display:** Enables viewing AI retrieval and reasoning steps in real-time.

#### 4. Chat History & User Profile Features
- **Session history management:** Automatically persists Q&A history per user (`chat_history/{user_id}.json`), allowing session review or history clearing.
- **Rich text response:** Renders AI answers with Markdown, copyable Code Blocks, LaTeX math formulas, and Data Tables.
- **Personal profile management:** View and update personal information (Name, Phone, DOB), change passwords, and update Avatars with drag-and-drop support.

#### 5. System Automation & Monitoring Features
- **Automated unconfirmed account cleanup:** EventBridge 5-minute cron rule invokes Lambda to purge unconfirmed registrations older than 5 minutes.
- **Automated CI/CD deployment:** CodePipeline & CodeBuild automatically run linters and unit test suites (~60 test cases) before building Docker Images and deploying Lambda.
- **System health monitoring & alerting:** CloudWatch Alarms track 4 critical system metrics and send instant email notifications via AWS SNS Topics.
