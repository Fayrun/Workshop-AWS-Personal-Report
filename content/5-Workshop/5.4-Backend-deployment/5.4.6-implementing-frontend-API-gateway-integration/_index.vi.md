---
title : "Viết Frontend gọi API Gateway"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.4.6 </b> "
---
### 1. Tạo Origin mới trên Cloudfront trỏ đến API Gateway
- Đăng nhập vào AWS Console và truy cập dịch vụ CloudFront.
- Chọn **Distribution** của bạn (**ID: E5QRZJSUK0QTJ**).
![distribution-id](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/1.png)
- Chọn tab **Origins** và nhấn **Create origin**.
![tab-origin](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/2.png)
- Cấu hình các thông số sau:
  - Origin domain: Chọn hoặc nhập địa chỉ **API Gateway d60866voq5.execute-api.us-east-1.amazonaws.com**
  - Protocol: Chọn **HTTPS only**.
  - Origin path: Nhập ```/prod```
  - Name: Nhập ```APIGatewayBackend```
![create-origin](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/3.png)
- Giữ các cấu hình khác mặc định và nhấn **Create origin**.
![create-origin-next](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/4.png)
### 2. Tạo Behavior để định tuyến các request /api/*
- Chuyển sang tab **Behaviors** và nhấn **Create behavior**.
![tab-behavior](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/5.png)
- Cấu hình các thông số quan trọng sau:
  - Path pattern: Nhập ```/api/*``` (để hứng toàn bộ request gửi lên backend).
  - Origin and origin groups: Chọn **Origin APIGatewayBackend** vừa tạo ở Bước 1.
  - Viewer protocol policy: Chọn **Redirect HTTP to HTTPS.**
  - Allowed HTTP methods: Chọn **GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE** (Quan trọng: Phải chọn mục này để hỗ trợ tải file lên S3, chat POST, xóa tài liệu).
![create-behavior1](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/6.png)
- Cache key and origin requests:
  - Chọn **Cache policy and origin request policy (recommended)**.
  - Cache policy: Chọn **CachingDisabled **
  - Origin request policy: Chọn **policy** có sẵn của AWS tên là **AllViewerExceptHostHeader**
![create-behavior2](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/7.png)
- Cuộn đến cuối trang và nhấn **Create behavior.**
![create-behavior3](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/8.png)
### 3. Tạo Invalidation để xóa cache cũ trên CloudFront
- Chuyển sang tab **Invalidations** và nhấn **Create invalidation**.
![tab-invalidation](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/9.png)
- Nhập ```/*``` vào ô Object paths rồi nhấn **Create invalidation**.
![create-invalidation](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/10.png)
- Chờ CloudFront deploy xong (khoảng 1-3 phút).
