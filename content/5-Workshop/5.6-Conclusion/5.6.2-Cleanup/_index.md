---
title : "Resource Cleanup"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

### Before You Delete

Deleting resources in this guide is **irreversible**: all data (user profiles, uploaded documents, system logs) will be permanently lost, since the workshop did not set up any backup or snapshot mechanism. If you need to keep any data, do so before deleting:

- Export the DynamoDB table
- Download the files stored in S3 (user documents and avatars)
- Export CloudWatch logs if you may need to analyze them later

---

### Cleanup Process - Resource Deletion Order

**Diagram:** 3 cleanup phases (Stop traffic → Remove compute → Delete data)

![Cleanup resources flow](/images/5-Workshop/5.6-Conclusion/5.6.2-Cleanup/cleanup-resources-flow.png)

**Phase summary:**

| Phase | Resources | Main Action | Time |
|-------|-----------|-------------|------|
| **Phase 1: Stop traffic** | EventBridge, CloudFront, API Gateway | Disable rule → Disable CloudFront (⏱️ wait 30 min) → Delete API Gateway | ~35 min |
| **Phase 2: Remove compute** | CodePipeline, CodeBuild, Lambda, ECR | Delete CI/CD pipeline → Delete Lambda function → Delete Docker image | ~5 min |
| **Phase 3: Delete data** | S3 Buckets (2), DynamoDB, Cognito, IAM | Empty S3 → Delete bucket → Delete user data → Delete permissions | ~5 min |
| **Phase 4: Monitoring** | CloudWatch Alarms, SNS Topic | Delete 4 alarms → Delete SNS topic + subscription | ~2 min |

**Total time:** ~47 minutes (mostly waiting for CloudFront to disable)

> **Note:** The S3 Intelligent-Tiering lifecycle rule and the SSE-KMS configuration on DynamoDB **do not need to be deleted separately** — they automatically disappear once the bucket/table itself is deleted in Phase 3.

---

### Example: Deleting Resources via AWS CLI

**1. Delete the EventBridge Rule:**
```powershell
# Disable rule
aws events disable-rule --name smartdocai-cleanup-unconfirmed --region us-east-1

# Remove Lambda target
aws events remove-targets --rule smartdocai-cleanup-unconfirmed --ids "1" --region us-east-1

# Delete rule
aws events delete-rule --name smartdocai-cleanup-unconfirmed --region us-east-1
```

**2. Delete CloudFront (requires waiting 30+ minutes):**
```powershell
$distId = "E1234ABCD5678"  # Replace with your distribution ID
aws cloudfront get-distribution-config --id $distId --output json > dist-config.json
# Edit dist-config.json: Set "Enabled": false
aws cloudfront update-distribution --id $distId --if-match <ETag> --distribution-config file://dist-config.json
# Wait until status = Disabled (check console every 5 min)
aws cloudfront delete-distribution --id $distId --if-match <new-ETag>
```

**3. Empty & delete the S3 Bucket:**
```powershell
$bucket = "smartdocai-storage-623035187993"
aws s3 rm s3://$bucket --recursive --region us-east-1  # Empty bucket
aws s3api delete-bucket --bucket $bucket --region us-east-1  # Delete bucket
```

**4. Delete the Lambda Function:**
```powershell
aws lambda delete-function --function-name smartdocai --region us-east-1
```

**5. Delete the DynamoDB table (with backup):**
```powershell
# Optional: Backup first
aws dynamodb scan --table-name smartdocai-user-profiles --region us-east-1 > backup.json
# Delete table
aws dynamodb delete-table --table-name smartdocai-user-profiles --region us-east-1
```

**6. Delete the Cognito User Pool:**
```powershell
aws cognito-idp delete-user-pool --user-pool-id us-east-1_3oq5wIiuu --region us-east-1
```

**7. Delete CloudWatch Alarms + SNS Topic:**
```powershell
# Delete the 4 alarms
aws cloudwatch delete-alarms --alarm-names smartdocai-lambda-errors smartdocai-lambda-duration smartdocai-lambda-throttles smartdocai-apigateway-5xx --region us-east-1

# Delete the SNS topic (this also automatically deletes its subscriptions)
aws sns delete-topic --topic-arn arn:aws:sns:us-east-1:623035187993:smartdocai-alerts --region us-east-1
```

**Full command list:** See the AWS Documentation or the `CLEANUP_GUIDE.md` file in the source code

---

## Confirming Cleanup Is Complete

**Checklist:**

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

**Expected final state:** All commands above return empty/no results

---

## Estimated Cost After Cleanup

**Remaining charges (if any):**
- CloudFront distribution (if not yet deleted): ~$0.01/day
- CloudWatch Logs (if not yet deleted): ~$0.001/day per GB stored
- S3 Glacier transitions (if a lifecycle policy exists): Varies

**No charges incurred once cleanup is complete:**
- Lambda: Deleted (no more invocations)
- API Gateway: Deleted (no more requests)
- S3: Deleted (no more storage)
- DynamoDB: Deleted (no more read/write capacity)
- Bedrock: Billed per use (no cost while idle)
- ECR: Deleted (no more image storage)
- CodePipeline: Deleted (no more active pipeline)
- CodeBuild: Deleted (no more builds)

**Check the final bill after 24-48 hours:**
```powershell
# View current month billing
aws ce get-cost-and-usage `
  --time-period Start=$(Get-Date -Format yyyy-MM-01),End=$(Get-Date -Format yyyy-MM-dd) `
  --granularity MONTHLY `
  --metrics BlendedCost `
  --region us-east-1
```

---

## Common Troubleshooting

### Issue 1: "Cannot delete S3 bucket - not empty"

**How to fix:**
```powershell
# Force empty bucket (including versioned objects)
aws s3api delete-objects `
  --bucket smartdocai-storage-623035187993 `
  --delete "$(aws s3api list-object-versions --bucket smartdocai-storage-623035187993 --query '{Objects: Versions[].{Key:Key,VersionId:VersionId}}' --output json)"

# Then delete bucket
aws s3api delete-bucket --bucket smartdocai-storage-623035187993 --region us-east-1
```

### Issue 2: "Cannot delete IAM role - policy still attached"

**How to fix:**
```powershell
# List attached policies
aws iam list-attached-role-policies --role-name smartdocai-lambda-role

# Detach each policy
aws iam detach-role-policy `
  --role-name smartdocai-lambda-role `
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Delete inline policies (if any)
aws iam list-role-policies --role-name smartdocai-lambda-role
aws iam delete-role-policy --role-name smartdocai-lambda-role --policy-name policy-name

# Then delete role
aws iam delete-role --role-name smartdocai-lambda-role
```

### Issue 3: "CloudFront distribution still deploying"

**How to fix:**
- Wait 5-10 minutes for the distribution status to become "Deployed"
- It cannot be disabled while status = "In Progress"
- After being disabled, wait another 15-30 minutes before it can be deleted

### Issue 4: "EventBridge rule has targets"

**How to fix:**
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
