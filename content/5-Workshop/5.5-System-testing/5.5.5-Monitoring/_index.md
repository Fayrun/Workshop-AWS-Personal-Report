---
title : "Monitoring & Logging"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5.5 </b> "
---

This section explains how to monitor the health of the SmartDocAI system via CloudWatch Logs and Metrics: reading real-time logs, writing Logs Insights queries to analyze errors/performance, and checking key Lambda metrics such as invocation count, processing time, and error rate.

### 1. CloudWatch Logs Overview

**Log Group:** `/aws/lambda/smartdocai`

**Key log patterns:**
- `[Auth]` → Authentication events (login, JWT validation)
- `[Document]` → Upload/indexing operations
- `[RAG]` → Query processing & Bedrock calls
- `[Cleanup]` → EventBridge cleanup results
- `ERROR` → Application errors
- `Traceback` → Python exceptions

**Command to view real-time logs:** `aws logs tail /aws/lambda/smartdocai --follow --region us-east-1`

**Example real log:**
```
2024-03-15T10:30:45.456Z [INFO] User login: test@example.com
2024-03-15T10:30:45.789Z [INFO] JWT validated successfully
2024-03-15T10:30:50.123Z [INFO] RAG query processed: 5 chunks retrieved
2024-03-15T10:30:50.456Z [INFO] Bedrock LLM response: 234 tokens generated
2024-03-15T10:30:50.999Z REPORT RequestId: abc123 Duration: 5234 ms Memory: 2048 MB Max Used: 1234 MB
```

![CloudWatch tail logs realtime](/images/5-Workshop/5.5-System-testing/5.5.5-Monitoring/cloudwatch-tail-logs-realtime.png)

---

### 2. CloudWatch Insights Queries

| Query Name | Purpose | Query | Expected Result |
|------------|---------|------------|-----------------|
| **Error Count (24h)** | Count errors by hour | `fields @timestamp, @message`<br/>`\| filter @message like /ERROR/`<br/>`\| stats count() as error_count by bin(1h)`<br/>`\| sort @timestamp desc` | Hourly error histogram (should be <1% of invocations) |
| **Average Lambda Duration** | Performance metrics | `fields @duration`<br/>`\| stats avg(@duration) as avg_ms,`<br/>`max(@duration) as max_ms,`<br/>`pct(@duration, 99) as p99_ms` | avg: 500-5000ms, max: <10s, p99: <8s |
| **Top 10 Slowest Invocations** | Find performance bottlenecks | `fields @timestamp, @duration, @message`<br/>`\| filter @type = "REPORT"`<br/>`\| sort @duration desc`<br/>`\| limit 10` | List of longest-running requests (cold starts: 2-3s) |
| **Cleanup Results** | EventBridge cleanup stats | `fields @timestamp, @message`<br/>`\| filter @message like /Deleted.*users/`<br/>`\| parse @message "Deleted * users" as deleted_count`<br/>`\| stats sum(deleted_count) as total_deleted by bin(1d)` | Daily count of unconfirmed users deleted |
| **Most Active Users** | User activity ranking | `fields @message`<br/>`\| filter @message like /User login:/`<br/>`\| parse @message "User login: *" as email`<br/>`\| stats count() as login_count by email`<br/>`\| sort login_count desc \| limit 10` | Top 10 users by login frequency |

![CloudWatch Logs Insights query](/images/5-Workshop/5.5-System-testing/5.5.5-Monitoring/cloudwatch-logs-insights-query.png)

---

### 3. Lambda Metrics

**CloudWatch metrics table:**

| Metric | AWS CLI Command | Expected Value (healthy system) |
|--------|-----------------|----------------------------------|
| **Invocations** | `aws cloudwatch get-metric-statistics`<br/>`--namespace AWS/Lambda --metric-name Invocations`<br/>`--dimensions Name=FunctionName,Value=smartdocai`<br/>`--start-time <1h ago> --period 300 --statistics Sum` | 100-1000/hour (depends on traffic) |
| **Errors** | `aws cloudwatch get-metric-statistics`<br/>`--namespace AWS/Lambda --metric-name Errors`<br/>`--dimensions Name=FunctionName,Value=smartdocai`<br/>`--start-time <1h ago> --period 300 --statistics Sum` | <1% of total invocations |
| **Throttles** | `aws cloudwatch get-metric-statistics`<br/>`--namespace AWS/Lambda --metric-name Throttles`<br/>`--dimensions Name=FunctionName,Value=smartdocai`<br/>`--start-time <1h ago> --period 300 --statistics Sum` | 0 (should never throttle with provisioned concurrency) |
| **Duration** | `aws cloudwatch get-metric-statistics`<br/>`--namespace AWS/Lambda --metric-name Duration`<br/>`--dimensions Name=FunctionName,Value=smartdocai`<br/>`--start-time <1h ago> --period 300 --statistics Average,Max` | Average: 500ms-5s<br/>Cold start: 2-3s<br/>Warm: <1s |

**Note:** Replace `<1h ago>` with the actual timestamp in the format: `$(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S)`

---

### 4. CloudWatch Alarms & SNS Alerting

> Proactively detects abnormal errors/performance through 4 CloudWatch Alarms + an SNS Topic, instead of only waiting for user reports or manually checking logs.

**SNS Topic:** `arn:aws:sns:us-east-1:623035187993:smartdocai-alerts` — sends an email alert to the registered address whenever one of the 4 alarms below switches to the `ALARM` state.

| Alarm Name | Metric | Threshold | Meaning |
|-----------|--------|--------|---------|
| `smartdocai-lambda-errors` | Lambda `Errors` | > 5 errors / 5 minutes | Backend is experiencing abnormal errors |
| `smartdocai-lambda-duration` | Lambda `Duration` | > 25000ms / 5 minutes | Close to the 30s timeout (risk of user-facing errors) |
| `smartdocai-lambda-throttles` | Lambda `Throttles` | ≥ 1 time / 5 minutes | Exceeded the concurrency limit, need to increase reserved concurrency |
| `smartdocai-apigateway-5xx` | API Gateway `5xxError` | > 5 errors / 5 minutes | Server error on the API Gateway/Lambda integration side |

![CloudWatch Alarms status](/images/5-Workshop/5.5-System-testing/5.5.5-Monitoring/cloudwatch-alarms-status.png)

**Confirming the email has confirmed the SNS subscription:**

![SNS subscription confirmed](/images/5-Workshop/5.5-System-testing/5.5.5-Monitoring/sns-subscription-confirmed.png)

**Verify via CLI:**
```powershell
aws cloudwatch describe-alarms --alarm-name-prefix smartdocai --region us-east-1 --query "MetricAlarms[].{Name:AlarmName,State:StateValue}" --output table
```

---
