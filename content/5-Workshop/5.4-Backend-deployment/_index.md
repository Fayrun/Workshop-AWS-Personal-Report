---
title : "Backend Deployment"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---


In this section, the team will implement and deploy the entire Backend architecture and serverless services for the SmartDocAI application on AWS. This includes initializing database services, document storage, user authentication management, building REST API Gateway endpoints, packaging and operating the AI RAG Engine on AWS Lambda via Container Images, routing frontend API requests through Amazon CloudFront, and automating the complete CI/CD pipeline using AWS CodePipeline.

### AWS Services Used

- **Amazon Cognito**: Manages user signup/login, issues JWT Tokens (`ID/Access Token`) to secure APIs, and links custom domain for Hosted UI.
- **Amazon DynamoDB**: Stores extended user profile data (`smartdocai-user-profiles`) in NoSQL On-Demand mode (`PAY_PER_REQUEST`).
- **Amazon S3**: Central storage for original user document uploads, FAISS Vector DB index files, user avatar images, and per-user isolated JSON metadata files (`uploads/`, `vectorstore/`, `processed_files/`, `chat_history/`).
- **AWS Lambda & Amazon ECR**: Core backend engine running FastAPI with Mangum ASGI handler packaged as Docker Container Images stored on ECR, integrated with Lambda Cognito Triggers and EventBridge Cron cleanup tasks.
- **Amazon API Gateway**: Provides REST API Proxy (`{proxy+}`) endpoints protected by Cognito Authorizer with CORS preflight configuration.
- **Amazon CloudFront Integration**: Routes API requests (`/api/*`) from the Frontend application on CloudFront CDN to the API Gateway backend using custom Origins and Behaviors.
- **AWS CodePipeline & CodeBuild**: Automates the CI/CD pipeline from the GitHub repository, running automated linting (`flake8`) & unit tests (`pytest`) with Hard Fail rules, building Docker Images, and updating AWS Lambda functions automatically.

---

### Contents

1. [Creating Amazon Cognito](5.4.1-creating-amazon-cognito/)
2. [Creating Amazon DynamoDB](5.4.2-creating-amazon-dynamoDB/)
3. [Creating Amazon S3 for Document Storage](5.4.3-creating-amazon-S3-for-document-storage/)
4. [Deploying AWS Lambda Engine](5.4.4-creating-AWS-lambda/)
5. [Creating API Gateway](5.4.5-creating-API-gateway/)
6. [Implementing Frontend API Gateway Integration](5.4.6-implementing-frontend-API-gateway-integration/)
7. [Automating Lambda Deployment with CodePipeline](5.4.7-automating-lambda-deployment-with-codePipeline/)
