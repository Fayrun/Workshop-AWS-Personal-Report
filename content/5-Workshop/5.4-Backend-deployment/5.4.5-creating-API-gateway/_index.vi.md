---
title : "Tạo API Gateway"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.4.5 </b> "
---

### 1. Khai Báo Biến PowerShell


![image12.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image12.png)


### 2. Lệnh Tạo REST API Mới


![image13.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image13.png)


Lấy Resource ID gốc (/) của API:


![image14.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image14.png)


### 3. Tạo Cognito Authorizer (Bảo vệ API bằng JWT Token)
![image15.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image15.png)

### 4. Tạo Proxy Resource và Method




#### 4.1. Tạo Resource {proxy+}:


![image16.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image16.png)


#### 4.2. Gán Method ANY với Cognito Authorizer:


![image17.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image17.png)


### 5. Tích Hợp API Gateway với AWS Lambda (AWS_PROXY Integration)


![image18.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image18.png)


### 6. Cấp Quyền cho API Gateway Invoke Lambda Function


![image19.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image19.png)


### 7. Deploy API Gateway lên Stage “prod”


![image20.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image20.png)


Kết quả:


![image21.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image21.png)


### 8. Cấu Hình CORS trên API Gateway

Để trình duyệt không bị chặn CORS khi thực hiện yêu cầu Preflight Request (OPTIONS), chạy thêm lệnh tạo method OPTIONS:


![image22.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image22.png)

