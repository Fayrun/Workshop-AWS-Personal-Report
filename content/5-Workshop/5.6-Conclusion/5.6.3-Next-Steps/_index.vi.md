---
title : "Bước tiếp theo & Tài liệu tham khảo"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3 </b> "
---

Workshop đã hoàn thành mục tiêu triển khai và kiểm thử SmartDocAI ở quy mô demo/thực tập. Phần này liệt kê các hạng mục nên cân nhắc bổ sung nếu muốn đưa hệ thống lên môi trường production thực sự với traffic lớn hơn.

### 1. Các cân nhắc khi triển khai Production
- Triển khai Lambda trong VPC kèm NAT Gateway
- Thêm API Gateway resource policy (IP whitelist) cho các endpoint quản trị
- Triển khai rate limiting theo từng user (khuyến nghị dùng AWS WAF)
- Bật Cognito MFA
- Triển khai blue/green deployment cho Lambda

### 2. Tối ưu chi phí
- Triển khai DynamoDB auto-scaling (reserved capacity)
- Thêm CloudFront caching cho API response (ở những endpoint phù hợp)
- Lambda SnapStart (giảm cold start còn ~100ms) — *lưu ý: hiện chỉ hỗ trợ Java/.NET, chưa hỗ trợ Python*
- Nén tài liệu trước khi upload (giảm dung lượng S3)

### 3. Tính năng mở rộng
- Triển khai multi-region (ap-southeast-1 Singapore)
- Cộng tác thời gian thực (WebSocket API)
- Hỗ trợ OCR tài liệu (Textract)
- Tìm kiếm full-text (OpenSearch)
- Thông báo qua email (SES)
- Audit logs (CloudTrail + S3)

### 4. Tài liệu tham khảo thêm
- [AWS Serverless Application Model (SAM)](https://aws.amazon.com/serverless/sam/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Amazon Bedrock Best Practices](https://docs.aws.amazon.com/bedrock/)
- [LangChain Documentation](https://python.langchain.com/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)

---

## Tài liệu tham khảo

- **Source code:** [GitHub - TakunKenjo/SmartdocAI-AWS](https://github.com/TakunKenjo/SmartdocAI-AWS)
- **Architecture:** Section 5.1.3 "Kiến trúc tổng thể trên AWS"
- **AWS services:** Section 5.1.4 "Các dịch vụ AWS được sử dụng"
- **Testing guide:** Section 5.5 "Kiểm thử hệ thống"
- **Security audit:** `SmartdocAI-AWS/SECURITY_CONSIDERATIONS.md`
- **EventBridge setup:** `SmartdocAI-AWS/EVENTBRIDGE_SETUP_GUIDE.md`
- **Handover document:** `SmartdocAI-AWS/BANGIAO.md` (branch: tam)

---

## Góp ý & Liên hệ

Nếu có câu hỏi hoặc góp ý về workshop này, vui lòng liên hệ:

- **Email:** 12345levan@gmail.com
- **GitHub:** [@TakunKenjo](https://github.com/TakunKenjo)
- **Kho mã nguồn Workshop:** [Workshop-AWS-Group-Report](https://github.com/TakunKenjo/Workshop-AWS-Group-Report)

**Cảm ơn bạn đã hoàn thành workshop SmartDocAI!**