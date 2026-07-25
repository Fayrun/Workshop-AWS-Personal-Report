---
title : "Next Steps & References"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3 </b> "
---

The workshop has achieved its goal of deploying and testing SmartDocAI at a demo/internship scale. This section lists items worth considering if you want to bring the system to a real production environment with higher traffic.

### 1. Production Deployment Considerations
- Deploy Lambda inside a VPC with a NAT Gateway
- Add an API Gateway resource policy (IP whitelist) for administrative endpoints
- Implement per-user rate limiting (AWS WAF recommended)
- Enable Cognito MFA
- Implement blue/green deployment for Lambda

### 2. Cost Optimization
- Implement DynamoDB auto-scaling (reserved capacity)
- Add CloudFront caching for API responses (on suitable endpoints)
- Lambda SnapStart (reduces cold start to ~100ms) — *note: currently only supports Java/.NET, not yet supported for Python*
- Compress documents before upload (reduces S3 storage)

### 3. Feature Expansion
- Multi-region deployment (ap-southeast-1 Singapore)
- Real-time collaboration (WebSocket API)
- Document OCR support (Textract)
- Full-text search (OpenSearch)
- Email notifications (SES)
- Audit logs (CloudTrail + S3)

### 4. Further References
- [AWS Serverless Application Model (SAM)](https://aws.amazon.com/serverless/sam/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Amazon Bedrock Best Practices](https://docs.aws.amazon.com/bedrock/)
- [LangChain Documentation](https://python.langchain.com/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)

---

## Documentation References

- **Source code:** [GitHub - TakunKenjo/SmartdocAI-AWS](https://github.com/TakunKenjo/SmartdocAI-AWS)
- **Architecture:** Section 5.1.3 "Overall Architecture on AWS"
- **AWS services:** Section 5.1.4 "AWS Services Used"
- **Testing guide:** Section 5.5 "System Testing"
- **Security audit:** `SmartdocAI-AWS/SECURITY_CONSIDERATIONS.md`
- **EventBridge setup:** `SmartdocAI-AWS/EVENTBRIDGE_SETUP_GUIDE.md`
- **Handover document:** `SmartdocAI-AWS/BANGIAO.md` (branch: tam)

---

## Feedback & Contact

If you have any questions or feedback about this workshop, please reach out:

- **Email:** 12345levan@gmail.com
- **GitHub:** [@TakunKenjo](https://github.com/TakunKenjo)
- **Workshop Repository:** [Workshop-AWS-Group-Report](https://github.com/TakunKenjo/Workshop-AWS-Group-Report)

**Thank you for completing the SmartDocAI workshop!**
