---
title : "Triển khai AWS Lambda Engine"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

### 1. Cài đặt Docker Desktop

#### 1.1. Yêu cầu

Hệ điều hành: Windows 10 (64-bit: Home, Pro, Enterprise từ Build 19041 trở lên) hoặc Windows 11.

Bật Ảo hóa Phần cứng (Hardware Virtualization Enabled): Đã bật Intel VT-x hoặc AMD-V trong BIOS/UEFI.

RAM: Tối thiểu 4 GB (Khuyên dùng 8 GB – 16 GB).

Dung lượng đĩa trống: Tối thiểu 10 GB.

#### 1.2. Kích hoạt tính năng WSL 2 ( WINDOWS SUBSYSTEM FOR LINUX 2)

Docker Desktop trên Windows tận dụng WSL 2 Engine để đạt hiệu năng xử lý container Linux gốc tối ưu

##### 1.2.1. Mở PowerShell với quyền Quản trị viên ( Run as Administrator)


![image29.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image29.png)


##### 1.2.2. Chạy câu lệnh kích hoạt tính năng WSL và Virtual Machine Platform


![image30.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image30.png)


##### 1.2.3. Khởi động lại máy tính

##### 1.2.4. Mở lại PowerShell (Admin) và đặt WSL 2 làm phiên bản mặc định


![image31.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image31.png)


#### 1.3. Tải và cài đặt Docker Desktop bản chính thức
##### 1.3.1. Truy cập trang chủ của Docker

https://www.docker.com/products/docker-desktop/


![image32.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image32.png)


##### 1.3.2. Nhấn nút Download for Windows để tải về tập tin Docker Desktop Installer.exe

##### 1.3.3. Mở tập tin .exe vừa tải về để tiến hành cài đặt:

Tích chọn "Use WSL 2 instead of Hyper-V (recommended)" (Dùng WSL 2 thay vì Hyper-V).

Tích chọn "Add shortcut to desktop" (Tạo lối tắt trên màn hình chính).

Nhấn Ok và chờ quá trình giải nén tập tin hoàn tất.

##### 1.3.4. Nhấn Close and restart để áp dụng thay đổi hệ thống.

#### 1.4. Khởi chạy và cấu hình Docker Desktop lần đầu

##### 1.4.1. Mở ứng dụng Docker Desktop


![image33.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image33.png)


##### 1.4.2. Chọn Accept để đồng ý với các điều khoản dịch vụ (Subscription Service Agreement).

##### 1.4.3. Tại giao diện chính Docker Desktop, chọn biểu tượng Settings (Bánh răng) ➔ Mục General:

- Đảm bảo ô Use the WSL 2 based engine đã được tích chọn.


![image34.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image34.png)


##### 1.4.4. Quan sát góc dưới bên trái giao diện: Biểu tượng Docker hiển thị màu xanh lá kèm dòng chữ Engine Running là Docker đã sẵn sàng hoạt động.


![image35.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image35.png)


### 2. Viết và tối ưu Docker File


![image36.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image36.png)


### 3. Khởi tạo Reponsitory trên Amazon ERC ( thông qua AWS CLI )

Mở PowerShell và thực hiện lệnh


![image37.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image37.png)


### 4. Đăng nhập Docker CLI vào Amazon ECR ( AUTHENTICATION )

Do Amazon ECR yêu cầu xác thực bảo mậtToken (có hiệu lực trong 12 giờ), ta dùng lệnh lấy mật khẩu tạm thời từ AWS ECR truyền trực tiếp vào Docker Login


![image38.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image38.png)


Kết quả : Login Success là coi như thành công

### 5. Build Docker Image và push lên Amazon ECR

#### 5.1. Build Docker Image

Nhập câu lệnh này trong Terminal


![image39.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image39.png)


#### 5.2. Push Docker Image lên ECR Repository


![image40.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image40.png)


### 6. Đồng bộ Docker Image với AWS Lambda

Mở terminal và nhập lệnh CLI để có thể đồng bộ


![image41.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image41.png)


Kết quả :


![image42.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image42.png)


####  6.1. Lập trình Backend Mangum Handler và cấu hình

#### 6.1.1. Cấu hình đường dẫn lưu trữ tạm trong backend/config.py

Môi trường AWS Lambda là Stateless và chỉ cho phép ghi dữ liệu tạm vào thư mục /tmp


![image43.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image43.png)


#### 6.1.2. Tạo Mangum Handler trong backend/app_api.py

Sử dụng thư viện Mangum để chuyển đổi các HTTP Request từ AWS API Gateway sang ASGI Format của FastAPI


![image44.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image44.png)



#### 6.2. Tạo IAM ROLE và phân quyền cho AWS Lambda

Lambda cần có quyền tương tác với S3, DynamoDB, Bedrock và CloudWatch

##### 6.2.1. Tạo Trust Policy (trust-policy.json):


![image46.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image46.png)


##### 6.2.2. Chạy lệnh AWS CLI tạo Role và gán các Managed Policy


![image47.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image47.png)


#### 6.4. Bình đóng gói và push Docker Image lên Amazon ECR


![image48.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image48.png)


### 7. Khởi tạo hàm AWS Lambda Function

#### 7.1. Mở Terminal hoặc PowerShell


![image49.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image49.png)


#### 7.2. Khởi tạo hàm Lambda Function thông qua AWS CLI

##### 7.2.1. Tạo hàm Lambda từ ECR Container Image

Hàm này là Backend Core xử lý toàn bộ logic API và AI RAG


![image50.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image50.png)


Sau đó cấu hình đầy đủ 7 Biến Môi Trường


![image51.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image51.png)


Cấp quyền cho API Gateway để gọi hàm Lambda này


![image52.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image52.png)


##### 7.2.2. Tạo hàm Lambda từ file Zip mã nguồn

Hàm này được dùng làm Cognito Trigger (Pre Sign-up) để dọn dẹp user chưa verify email


![image53.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image53.png)


Sau đó phải Cấp quyền cho Cognito User Pool gọi hàm Lambda này


![image54.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image54.png)


Kết quả :


![image55.png](/images/5-Workshop/5.4-Backend-deployment/5.4.4-creating-AWS-lambda/image55.png)


---

### 8. Cấu hình Amazon EventBridge Rule tự động dọn dẹp tài khoản rác 

Để tự động dọn dẹp các tài khoản rác tạo ra quá 5 phút mà người dùng chưa xác minh mã OTP email trong Cognito:

- **Tạo EventBridge Rule (`smartdocai-cleanup-unconfirmed`)**: Thiết lập lịch trình kích hoạt định kỳ 5 phút/lần (`rate(5 minutes)`).
- **Target Event Payload**: Cấu hình gửi JSON payload với thuộc tính `"source": "aws.events"` tới AWS Lambda `smartdocai`.
- **Cơ chế xử lý**: Khi nhận event từ EventBridge, Lambda Handler trong `app_api.py` phát hiện `"source": "aws.events"` và kích hoạt hàm `cleanup_unconfirmed_users()`. Hàm này quét Cognito User Pool và thực thi `admin_delete_user` đối với các tài khoản ở trạng thái `UNCONFIRMED` có thời gian khởi tạo vượt quá 5 phút.

```bash
# 1. Tạo EventBridge Rule chạy định kỳ 5 phút
aws events put-rule \
    --name smartdocai-cleanup-unconfirmed \
    --schedule-expression "rate(5 minutes)" \
    --state ENABLED

# 2. Cấp quyền cho EventBridge kích hoạt AWS Lambda
aws lambda add-permission \
    --function-name smartdocai \
    --statement-id EventBridgeCleanupPermission \
    --action lambda:InvokeFunction \
    --principal events.amazonaws.com \
    --source-arn arn:aws:events:us-east-1:ACCOUNT_ID:rule/smartdocai-cleanup-unconfirmed

# 3. Gán Lambda Function làm Target với JSON Payload {"source": "aws.events"}
aws events put-targets \
    --rule smartdocai-cleanup-unconfirmed \
    --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:ACCOUNT_ID:function:smartdocai","Input"="{\"source\": \"aws.events\"}"
```


