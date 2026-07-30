---
title : "Giám sát & Nhật ký hệ thống"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5.5 </b> "
---

Phần này hướng dẫn cách theo dõi sức khỏe hệ thống SmartDocAI qua CloudWatch Logs và Metrics: đọc log thời gian thực, viết truy vấn Logs Insights để phân tích lỗi/hiệu năng, và kiểm tra các chỉ số Lambda quan trọng như số lần gọi, thời gian xử lý và tỷ lệ lỗi.

### 1. Tổng quan CloudWatch Logs

**Log Group:** `/aws/lambda/smartdocai`

**Các mẫu log quan trọng:**
- `[Auth]` → Sự kiện xác thực (đăng nhập, kiểm tra JWT)
- `[Document]` → Thao tác tải lên/lập chỉ mục
- `[RAG]` → Xử lý truy vấn & gọi Bedrock
- `[Cleanup]` → Kết quả dọn dẹp qua EventBridge
- `ERROR` → Lỗi ứng dụng
- `Traceback` → Ngoại lệ Python

**Lệnh xem log thời gian thực:** `aws logs tail /aws/lambda/smartdocai --follow --region us-east-1`

**Ví dụ log thực tế:**
```
2024-03-15T10:30:45.456Z [INFO] User login: test@example.com
2024-03-15T10:30:45.789Z [INFO] JWT validated successfully
2024-03-15T10:30:50.123Z [INFO] RAG query processed: 5 chunks retrieved
2024-03-15T10:30:50.456Z [INFO] Bedrock LLM response: 234 tokens generated
2024-03-15T10:30:50.999Z REPORT RequestId: abc123 Duration: 5234 ms Memory: 2048 MB Max Used: 1234 MB
```

![CloudWatch tail logs realtime](/images/5-Workshop/5.5-System-testing/5.5.5-Monitoring/cloudwatch-tail-logs-realtime.png)

---

### 2. Truy vấn CloudWatch Insights

| Tên truy vấn | Mục đích | Câu lệnh | Kết quả mong đợi |
|------------|---------|------------|-----------------|
| **Error Count (24h)** | Đếm lỗi theo giờ | `fields @timestamp, @message`<br/>`\| filter @message like /ERROR/`<br/>`\| stats count() as error_count by bin(1h)`<br/>`\| sort @timestamp desc` | Biểu đồ lỗi theo giờ (nên <1% tổng số lần gọi) |
| **Average Lambda Duration** | Chỉ số hiệu năng | `fields @duration`<br/>`\| stats avg(@duration) as avg_ms,`<br/>`max(@duration) as max_ms,`<br/>`pct(@duration, 99) as p99_ms` | avg: 500-5000ms, max: <10s, p99: <8s |
| **Top 10 Slowest Invocations** | Tìm điểm nghẽn hiệu năng | `fields @timestamp, @duration, @message`<br/>`\| filter @type = "REPORT"`<br/>`\| sort @duration desc`<br/>`\| limit 10` | Danh sách request chạy lâu nhất (cold start: 2-3s) |
| **Cleanup Results** | Thống kê dọn dẹp EventBridge | `fields @timestamp, @message`<br/>`\| filter @message like /Deleted.*users/`<br/>`\| parse @message "Deleted * users" as deleted_count`<br/>`\| stats sum(deleted_count) as total_deleted by bin(1d)` | Số lượng user chưa xác thực bị xóa mỗi ngày |
| **Most Active Users** | Xếp hạng hoạt động người dùng | `fields @message`<br/>`\| filter @message like /User login:/`<br/>`\| parse @message "User login: *" as email`<br/>`\| stats count() as login_count by email`<br/>`\| sort login_count desc \| limit 10` | Top 10 user đăng nhập nhiều nhất |

![CloudWatch Logs Insights query](/images/5-Workshop/5.5-System-testing/5.5.5-Monitoring/cloudwatch-logs-insights-query.png)

---

### 3. Chỉ số Lambda (Metrics)

**Bảng các chỉ số CloudWatch:**

| Chỉ số | Lệnh AWS CLI | Giá trị mong đợi (hệ thống khỏe mạnh) |
|--------|-----------------|----------------------------------|
| **Invocations** | `aws cloudwatch get-metric-statistics`<br/>`--namespace AWS/Lambda --metric-name Invocations`<br/>`--dimensions Name=FunctionName,Value=smartdocai`<br/>`--start-time <1h ago> --period 300 --statistics Sum` | 100-1000/giờ (tùy lưu lượng) |
| **Errors** | `aws cloudwatch get-metric-statistics`<br/>`--namespace AWS/Lambda --metric-name Errors`<br/>`--dimensions Name=FunctionName,Value=smartdocai`<br/>`--start-time <1h ago> --period 300 --statistics Sum` | <1% tổng số lần gọi |
| **Throttles** | `aws cloudwatch get-metric-statistics`<br/>`--namespace AWS/Lambda --metric-name Throttles`<br/>`--dimensions Name=FunctionName,Value=smartdocai`<br/>`--start-time <1h ago> --period 300 --statistics Sum` | 0 (không nên bị throttle khi có provisioned concurrency) |
| **Duration** | `aws cloudwatch get-metric-statistics`<br/>`--namespace AWS/Lambda --metric-name Duration`<br/>`--dimensions Name=FunctionName,Value=smartdocai`<br/>`--start-time <1h ago> --period 300 --statistics Average,Max` | Trung bình: 500ms-5s<br/>Cold start: 2-3s<br/>Warm: <1s |

**Lưu ý:** Thay `<1h ago>` bằng timestamp thực tế theo định dạng: `$(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S)`

---

### 4. CloudWatch Alarms & SNS Alerting

> Chủ động phát hiện lỗi/hiệu năng bất thường qua 4 CloudWatch Alarm + SNS Topic, thay vì chỉ chờ user report hoặc tự tra log thủ công.

**SNS Topic:** `arn:aws:sns:us-east-1:623035187993:smartdocai-alerts` — gửi email cảnh báo tới địa chỉ đã đăng ký mỗi khi 1 trong 4 Alarm bên dưới chuyển sang trạng thái `ALARM`.

| Alarm Name | Metric | Ngưỡng | Ý nghĩa |
|-----------|--------|--------|---------|
| `smartdocai-lambda-errors` | Lambda `Errors` | > 5 lỗi / 5 phút | Backend đang lỗi bất thường |
| `smartdocai-lambda-duration` | Lambda `Duration` | > 25000ms / 5 phút | Gần chạm timeout 30s (rủi ro user gặp lỗi) |
| `smartdocai-lambda-throttles` | Lambda `Throttles` | ≥ 1 lần / 5 phút | Vượt concurrency limit, cần tăng reserved concurrency |
| `smartdocai-apigateway-5xx` | API Gateway `5xxError` | > 5 lỗi / 5 phút | Lỗi server phía API Gateway/Lambda integration |

![CloudWatch Alarms status](/images/5-Workshop/5.5-System-testing/5.5.5-Monitoring/cloudwatch-alarms-status.png)

**Xác nhận email đã confirm nhận cảnh báo từ SNS:**

![SNS subscription confirmed](/images/5-Workshop/5.5-System-testing/5.5.5-Monitoring/sns-subscription-confirmed.png)

**Verify bằng CLI:**
```powershell
aws cloudwatch describe-alarms --alarm-name-prefix smartdocai --region us-east-1 --query "MetricAlarms[].{Name:AlarmName,State:StateValue}" --output table
```

---