---
title : "Backend Deployment"
date : 2024-01-01 
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---


In this section, the team will deploy the entire backend processing system and supporting microservices infrastructure (Serverless architecture) for SmartDocAI on AWS. The deployment process includes initializing database services, document storage, user identity management, building the REST API Gateway, packaging and running the AI RAG Engine on AWS Lambda via Docker Container Image, routing API requests from Frontend through Amazon CloudFront, and automating the entire CI/CD pipeline with AWS CodePipeline.

### AWS Services Used

- **Amazon Cognito**: Manages user registration/login, issues secure JWT Tokens for API authentication, and binds custom domain for Hosted UI.
- **Amazon DynamoDB**: Stores extended user profile information using an On-Demand NoSQL model.
- **Amazon S3**: Centrally stores raw document files, FAISS Vector DB indexes, avatar profile pictures, and JSON metadata files isolated per user.
- **AWS Lambda & Amazon ECR**: Houses the centralized backend processing engine (FastAPI + Mangum handler) packaged as a Docker Container Image stored on ECR, combined with Cognito Lambda Triggers and EventBridge Cron Jobs for automatic unverified account cleanup.
- **Amazon API Gateway**: Provides REST API Proxy, integrates Cognito Authorizer to secure API resources, and configures CORS Preflight for the Frontend.
- **Amazon CloudFront Integration**: Routes API requests from the Frontend application on CloudFront CDN to the API Gateway backend using corresponding Origins and Behaviors.
- **AWS CodePipeline & CodeBuild**: Automates the CI/CD pipeline from the GitHub repository, executes Hard-Fail Linting & Unit Tests, packages Docker Images, and automatically updates the Lambda function source code.

---

### Implementation Content

1. [Creating Amazon Cognito](5.4.1-creating-amazon-cognito/)
2. [Creating Amazon DynamoDB](5.4.2-creating-amazon-dynamoDB/)
3. [Creating Amazon S3 for Document Storage](5.4.3-creating-amazon-S3-for-document-storage/)
4. [Deploying AWS Lambda Engine](5.4.4-creating-AWS-lambda/)
5. [Creating API Gateway](5.4.5-creating-API-gateway/)
6. [Implementing Frontend API Gateway Integration](5.4.6-implementing-frontend-API-gateway-integration/)
7. [Automating Lambda Deployment with CodePipeline](5.4.7-automating-lambda-deployment-with-codePipeline/)
