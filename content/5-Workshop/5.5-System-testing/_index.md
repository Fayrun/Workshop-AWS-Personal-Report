---
title : "System Testing"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

After completing the Frontend and Backend deployment to AWS in the previous chapters, the team carried out comprehensive testing of the **SmartDocAI** system in a real production environment. The goal of this section is to verify that the main business flows (registration/login, document upload and RAG query, personal profile management) work as designed, while also assessing the monitoring, security, and CI/CD stability of the system.

Unlike the infrastructure deployment chapters (creating resources through the AWS Console UI), testing is mostly carried out through the terminal (`curl`, AWS CLI) and cross-checked against the Console, so the content focuses on test-case tables and real logs rather than step-by-step screenshots.

### Contents

1. [Authentication Testing](5.5.1-Authentication/)
2. [Document Upload & RAG Testing](5.5.2-Document-RAG/)
3. [Security Testing](5.5.3-Security/)
4. [Profile Testing](5.5.4-Profile/)
5. [Monitoring & Logging](5.5.5-Monitoring/)
6. [CI/CD Automated Testing](5.5.6-CICD/)
