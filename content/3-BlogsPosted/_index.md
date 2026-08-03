---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section presents the technical blog posts published and shared during the internship. The first two blogs were written based on issues encountered while developing the Smart Docs AI project on AWS. The third blog is a technical article shared by a teammate to broaden the team's understanding of Amazon Bedrock architecture and AWS Landing Zone best practices.

### [Blog 1 - Lambda Tenant Isolation Mode: Can It Really Solve Multi-Tenant Data Leakage?](3.1-Blog1/)

This blog analyzes a real multi-tenant data leakage issue encountered during the Smart Docs AI project. It explains how AWS Lambda Tenant Isolation Mode works, why it cannot automatically resolve application-level data isolation problems, and how the issue was ultimately fixed through proper tenant-aware data modeling.

### [Blog 2 - EventBridge Scheduler: When Should You Upgrade from EventBridge Rule?](3.2-Blog2/)

This blog compares Amazon EventBridge Scheduler with the traditional EventBridge Rule through a practical use case. It discusses the advantages of Scheduler, explains why EventBridge Rule remains the most suitable choice for the project's periodic cleanup task, and highlights the importance of selecting AWS services based on actual requirements rather than adopting newer services by default.

### [Blog 3 - Amazon Bedrock Baseline Architecture in an AWS Landing Zone (Shared)](3.3-Blog3/)

This shared article, written by a teammate, introduces a secure baseline architecture for deploying Amazon Bedrock in an AWS Landing Zone. It covers multi-account governance, network isolation, identity and access management, centralized logging, and security best practices for building enterprise-grade Generative AI solutions on AWS.
