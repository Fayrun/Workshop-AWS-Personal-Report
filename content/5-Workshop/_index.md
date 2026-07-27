---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

### Overview

This chapter details the step-by-step process of building, configuring, deploying, testing, and automating the complete SmartDocAI application on AWS Serverless infrastructure.

The system combines a Serverless Container Architecture using AWS Lambda (FastAPI Container), Amazon API Gateway, AWS Cognito, Amazon DynamoDB, Amazon S3, AWS CloudFront, and advanced Generative AI models from Amazon Bedrock (Titan Embeddings V2 & Qwen 3 Next 80B A3B).

Learners will practice from resource preparation, Docker Container packaging, cloud service configuration, IAM security permission setup, CI/CD automation pipeline integration via AWS CodePipeline/CodeBuild, to load testing and resource cleanup.

---

#### Detailed Workshop Guide Table of Contents

1. **[Workshop Overview](5.1-Workshop-overview/)**
   - 5.1.1 [Frontend Architecture Specification](5.1-Workshop-overview/5.1.1%20-frontend-architecture/)
   - 5.1.2 [Backend Architecture & RAG Pipeline Specification](5.1-Workshop-overview/5.1.2%20-backend-architecture/)
   - 5.1.3 [Overall AWS Architecture Diagram](5.1-Workshop-overview/5.1.3%20-overall-aws-architecture/)
   - 5.1.4 [AWS Services Summary](5.1-Workshop-overview/5.1.4%20-aws-services-used/)
   - 5.1.5 [UI Specification & System Functionality](5.1-Workshop-overview/5.1.5%20-ui-function/)

2. **[Prerequisites](5.2-Prerequiste/)**
   - 5.2.1 [Source Code Preparation](5.2-Prerequiste/5.2.1-source-code-preparation/)
   - 5.2.2 [AWS Account & Amazon Bedrock Access Setup](5.2-Prerequiste/5.2.2-aws-account-preparation/)
   - 5.2.3 [IAM User Creation & AWS CLI Configuration](5.2-Prerequiste/5.2.3-creating-an-IAM-user/)

3. **[Frontend SPA Deployment](5.3-Frontend-deployment/)**
   - 5.3.1 [Create S3 Bucket for Static Website Hosting](5.3-Frontend-deployment/5.3.1-create-S3-bucket/)
   - 5.3.2 [Enable Static Website Hosting Feature](5.3-Frontend-deployment/5.3.2-enable-static-web-hosting/)
   - 5.3.3 [Configure Block Public Access Settings](5.3-Frontend-deployment/5.3.3-config-block-public-access-settings/)
   - 5.3.4 [Configure S3 Bucket Policy for Public Read Permission](5.3-Frontend-deployment/5.3.4-config-bucket-policy-public-read/)
   - 5.3.5 [Verify Static Website Deployment](5.3-Frontend-deployment/5.3.5-website-verification/)
   - 5.3.6 [Accelerate Performance & Security with AWS CloudFront (CDN)](5.3-Frontend-deployment/5.3.6-accelerating-website-performance-with-cloudfront/)
   - 5.3.7 [Automate Frontend Deployment with AWS CodePipeline](5.3-Frontend-deployment/5.3.7-codepipeline-to-S3-frontend/)

4. **[AWS Backend Infrastructure Deployment](5.4-Backend-deployment/)**
   - 5.4.1 [Create Amazon Cognito User Pool & Bind Hosted UI Domain](5.4-Backend-deployment/5.4.1-creating-amazon-cognito/)
   - 5.4.2 [Create Amazon DynamoDB NoSQL Tables](5.4-Backend-deployment/5.4.2-creating-amazon-dynamoDB/)
   - 5.4.3 [Create Amazon S3 Document Bucket & Configure CORS](5.4-Backend-deployment/5.4.3-creating-amazon-S3-for-document-storage/)
   - 5.4.4 [Package Docker Container & Deploy AWS Lambda Engine](5.4-Backend-deployment/5.4.4-creating-AWS-lambda/)
   - 5.4.5 [Create Amazon API Gateway REST API & Cognito Authorizer](5.4-Backend-deployment/5.4.5-creating-API-gateway/)
   - 5.4.6 [Routing & Integrating Frontend CloudFront to API Gateway](5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/)
   - 5.4.7 [Automate Lambda Deployment with AWS CodePipeline & CodeBuild](5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/)

5. **[System Testing](5.5-System-testing/)**
   - 5.5.1 [Authentication Testing](5.5-System-testing/5.5.1-Authentication/)
   - 5.5.2 [Document Upload & RAG Testing](5.5-System-testing/5.5.2-Document-RAG/)
   - 5.5.3 [Security Testing](5.5-System-testing/5.5.3-Security/)
   - 5.5.4 [Profile Testing](5.5-System-testing/5.5.4-Profile/)
   - 5.5.5 [Monitoring & Logging](5.5-System-testing/5.5.5-Monitoring/)
   - 5.5.6 [CI/CD Automated Testing](5.5-System-testing/5.5.6-CICD/)

6. **[Conclusion](5.6-Conclusion/)**
   - 5.6.1 [Workshop Summary & Cost](5.6-Conclusion/5.6.1-Summary-Cost/)
   - 5.6.2 [Resource Cleanup](5.6-Conclusion/5.6.2-Cleanup/)
   - 5.6.3 [Next Steps & References](5.6-Conclusion/5.6.3-Next-Steps/)
