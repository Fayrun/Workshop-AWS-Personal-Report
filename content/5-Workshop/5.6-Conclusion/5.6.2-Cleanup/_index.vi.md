---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

### Lưu ý trước khi xóa

Việc xóa tài nguyên trong hướng dẫn này là **không thể hoàn tác**: toàn bộ dữ liệu (hồ sơ người dùng, tài liệu đã tải lên, log hệ thống) sẽ mất vĩnh viễn, vì workshop không thiết lập cơ chế backup hay snapshot nào. Nếu cần giữ lại dữ liệu, hãy làm trước khi xóa:

- Export bảng DynamoDB
- Tải về các file trên S3 (tài liệu và avatar người dùng)
- Export CloudWatch logs nếu sau này cần phân tích lại

---

### Quy trình dọn dẹp - Thứ tự xóa tài nguyên

**Sơ đồ:** 3 giai đoạn dọn dẹp (Dừng traffic → Gỡ compute → Xóa dữ liệu)

![Cleanup resources flow](/images/5-Workshop/5.6-Conclusion/5.6.2-Cleanup/cleanup-resources-flow.png)

**Tóm tắt các giai đoạn:**

| Giai đoạn | Tài nguyên | Hành động chính | Thời gian |
|-------|-----------|-------------|------|
| **Giai đoạn 1: Dừng traffic** | EventBridge, CloudFront, API Gateway | Tắt rule → Tắt CloudFront (⏱️ chờ 30 phút) → Xóa API Gateway | ~35 phút |
| **Giai đoạn 2: Gỡ compute** | CodePipeline, CodeBuild, Lambda, ECR | Xóa CI/CD pipeline → Xóa Lambda function → Xóa Docker image | ~5 phút |
| **Giai đoạn 3: Xóa dữ liệu** | S3 Buckets (2), DynamoDB, Cognito, IAM | Làm rỗng S3 → Xóa bucket → Xóa dữ liệu user → Xóa quyền | ~5 phút |
| **Giai đoạn 4: Giám sát** | CloudWatch Alarms, SNS Topic | Xóa 4 alarms → Xóa SNS topic + subscription | ~2 phút |

**Tổng thời gian:** ~47 phút (chủ yếu chờ CloudFront disable)

> **Lưu ý:** Lifecycle rule S3 Intelligent-Tiering và cấu hình SSE-KMS trên DynamoDB **không cần xóa riêng** — chúng tự động biến mất khi xóa hẳn bucket/table ở Giai đoạn 3.

---

### Ví dụ: Xóa tài nguyên bằng AWS CLI

**1. Xóa EventBridge Rule:**
```powershell
# Disable rule
aws events disable-rule --name smartdocai-cleanup-unconfirmed --region us-east-1

# Remove Lambda target
aws events remove-targets --rule smartdocai-cleanup-unconfirmed --ids "1" --region us-east-1

# Delete rule
aws events delete-rule --name smartdocai-cleanup-unconfirmed --region us-east-1
```

**2. Xóa CloudFront (cần chờ 30+ phút):**
```powershell
$distId = "E1234ABCD5678"  # Replace with your distribution ID
aws cloudfront get-distribution-config --id $distId --output json > dist-config.json
# Edit dist-config.json: Set "Enabled": false
aws cloudfront update-distribution --id $distId --if-match <ETag> --distribution-config file://dist-config.json
# Wait until status = Disabled (check console every 5 min)
aws cloudfront delete-distribution --id $distId --if-match <new-ETag>
```

**3. Làm rỗng & xóa S3 Bucket:**
```powershell
$bucket = "smartdocai-storage-623035187993"
aws s3 rm s3://$bucket --recursive --region us-east-1  # Empty bucket
aws s3api delete-bucket --bucket $bucket --region us-east-1  # Delete bucket
```

**4. Xóa Lambda Function:**
```powershell
aws lambda delete-function --function-name smartdocai --region us-east-1
```

**5. Xóa bảng DynamoDB (có backup):**
```powershell
# Optional: Backup first
aws dynamodb scan --table-name smartdocai-user-profiles --region us-east-1 > backup.json
# Delete table
aws dynamodb delete-table --table-name smartdocai-user-profiles --region us-east-1
```

**6. Xóa Cognito User Pool:**
```powershell
aws cognito-idp delete-user-pool --user-pool-id us-east-1_3oq5wIiuu --region us-east-1
```

**7. Xóa CloudWatch Alarms + SNS Topic:**
```powershell
# Xóa 4 alarms
aws cloudwatch delete-alarms --alarm-names smartdocai-lambda-errors smartdocai-lambda-duration smartdocai-lambda-throttles smartdocai-apigateway-5xx --region us-east-1

# Xóa SNS topic (tự động xóa luôn các subscription đính kèm)
aws sns delete-topic --topic-arn arn:aws:sns:us-east-1:623035187993:smartdocai-alerts --region us-east-1
```

**Toàn bộ danh sách lệnh:** Tham khảo AWS Documentation hoặc file `CLEANUP_GUIDE.md` trong source code

---

## Kiểm tra xác nhận đã dọn dẹp xong

**Danh sách kiểm tra:**

```powershell
# 1. EventBridge
aws events list-rules --region us-east-1 | Select-String "smartdocai"
# Expected: No results

# 2. CloudFront
aws cloudfront list-distributions --query "DistributionList.Items[?contains(Comment, 'smartdocai')]" --output table
# Expected: Empty

# 3. API Gateway
aws apigateway get-rest-apis --region us-east-1 | Select-String "smartdocai"
# Expected: No results

# 4. Lambda
aws lambda list-functions --region us-east-1 | Select-String "smartdocai"
# Expected: No results

# 5. ECR
aws ecr describe-repositories --region us-east-1 | Select-String "smartdocai"
# Expected: No results

# 6. S3
aws s3 ls | Select-String "smartdocai"
# Expected: No results

# 7. DynamoDB
aws dynamodb list-tables --region us-east-1 | Select-String "smartdocai"
# Expected: No results

# 8. Cognito
aws cognito-idp list-user-pools --max-results 60 --region us-east-1 | Select-String "smartdocai"
# Expected: No results

# 9. CloudWatch Logs
aws logs describe-log-groups --region us-east-1 | Select-String "smartdocai"
# Expected: No results

# 10. IAM Roles
aws iam list-roles | Select-String "smartdocai"
# Expected: No results
```

**Trạng thái cuối cùng mong đợi:** Tất cả các lệnh trên trả về kết quả rỗng/không tìm thấy

---

## Ước tính chi phí sau khi dọn dẹp

**Các khoản phí còn lại (nếu có):**
- CloudFront distribution (nếu chưa xóa): ~$0.01/ngày
- CloudWatch Logs (nếu chưa xóa): ~$0.001/ngày mỗi GB lưu trữ
- S3 Glacier transitions (nếu có lifecycle policy): Tùy thuộc

**Không phát sinh chi phí sau khi dọn dẹp hoàn tất:**
- Lambda: Đã xóa (không còn invocation)
- API Gateway: Đã xóa (không còn request)
- S3: Đã xóa (không còn lưu trữ)
- DynamoDB: Đã xóa (không còn read/write capacity)
- Bedrock: Tính phí theo mức dùng (không tốn phí khi rảnh)
- ECR: Đã xóa (không còn lưu trữ image)
- CodePipeline: Đã xóa (không còn pipeline hoạt động)
- CodeBuild: Đã xóa (không còn build)

**Kiểm tra hóa đơn cuối cùng sau 24-48 giờ:**
```powershell
# View current month billing
aws ce get-cost-and-usage `
  --time-period Start=$(Get-Date -Format yyyy-MM-01),End=$(Get-Date -Format yyyy-MM-dd) `
  --granularity MONTHLY `
  --metrics BlendedCost `
  --region us-east-1
```

---

## Xử lý sự cố thường gặp

### Sự cố 1: "Cannot delete S3 bucket - not empty"

**Cách xử lý:**
```powershell
# Force empty bucket (including versioned objects)
aws s3api delete-objects `
  --bucket smartdocai-storage-623035187993 `
  --delete "$(aws s3api list-object-versions --bucket smartdocai-storage-623035187993 --query '{Objects: Versions[].{Key:Key,VersionId:VersionId}}' --output json)"

# Then delete bucket
aws s3api delete-bucket --bucket smartdocai-storage-623035187993 --region us-east-1
```

### Sự cố 2: "Cannot delete IAM role - policy still attached"

**Cách xử lý:**
```powershell
# List attached policies
aws iam list-attached-role-policies --role-name smartdocai-lambda-role

# Detach each policy
aws iam detach-role-policy `
  --role-name smartdocai-lambda-role `
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Delete inline policies (nếu có)
aws iam list-role-policies --role-name smartdocai-lambda-role
aws iam delete-role-policy --role-name smartdocai-lambda-role --policy-name policy-name

# Then delete role
aws iam delete-role --role-name smartdocai-lambda-role
```

### Sự cố 3: "CloudFront distribution still deploying"

**Cách xử lý:**
- Đợi 5-10 phút cho distribution status = "Deployed"
- Không thể disable khi status = "In Progress"
- Sau khi disabled, đợi thêm 15-30 phút mới có thể delete

### Sự cố 4: "EventBridge rule has targets"

**Cách xử lý:**
```powershell
# Remove targets first
aws events remove-targets `
  --rule smartdocai-cleanup-unconfirmed `
  --ids "1" `
  --region us-east-1

# Then delete rule
aws events delete-rule --name smartdocai-cleanup-unconfirmed --region us-east-1
```

---