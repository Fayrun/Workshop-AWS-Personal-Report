# DANH SÁCH SCREENSHOT CẦN CHỤP — 5.1.3, 5.1.4, 5.5, 5.6

Tài liệu tổng hợp toàn bộ screenshot còn thiếu cho 4 phần workshop, dùng để bổ sung bằng chứng thực tế bên cạnh các sơ đồ Mermaid đã có sẵn. Mỗi mục gồm: **Tên file**, **Path lưu**, **Mô tả nội dung cần thể hiện**, và **hướng dẫn từng bước có command cụ thể**.

> **Quy ước path:** Lưu vào `static/images/5-Workshop/<section>/<subsection>/`, tham chiếu trong markdown bằng `/images/5-Workshop/<section>/<subsection>/<file>.png`.

## 0. Thông tin dùng chung (chạy trước tiên trong PowerShell)

> **Lưu ý quan trọng:** Trong PowerShell, `curl` là alias của `Invoke-WebRequest` (khác cú pháp với curl thật trên Linux/macOS) và sẽ báo lỗi khi dùng `-H`. **Luôn gõ `curl.exe`** (thêm `.exe`) để gọi đúng curl thật có sẵn trong Windows 10/11.

Mở PowerShell, chạy 3 lệnh sau để: (1) hiển thị đúng tiếng Việt trong terminal, (2) gán biến dùng lại cho các lệnh bên dưới. **Chỉ cần làm 1 lần mỗi phiên terminal mới**:

```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
chcp 65001

$API_BASE = "https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod"
$CLOUDFRONT = "https://dutf3c70nnjzl.cloudfront.net"
$REGION = "us-east-1"
```

> **Vì sao cần `chcp 65001`:** Response API có message tiếng Việt (ví dụ "Đăng ký thành công!"). Nếu không set UTF-8, terminal sẽ hiển thị chữ bị vỡ dạng `ÄÄng kÃ½ thÃ nh cÃ´ng` — đây chỉ là lỗi hiển thị, không phải lỗi dữ liệu thật, nhưng cần sửa để chụp ảnh không bị xấu/khó đọc.

> **Lưu ý:** Khối bên dưới chỉ là **giá trị tham khảo** để bạn biết ID/tên thật của từng resource khi làm theo hướng dẫn — **KHÔNG copy-paste chạy trực tiếp** vào PowerShell (không phải lệnh, sẽ báo lỗi `CommandNotFoundException`). Khi hướng dẫn nào cần dùng, tôi sẽ ghi rõ giá trị luôn trong lệnh, ví dụ `us-east-1_3oq5wIiuu`.

```text
Cognito Pool ID  : us-east-1_3oq5wIiuu
Lambda Name      : smartdocai
S3 Storage       : smartdocai-storage-623035187993
DynamoDB Table   : smartdocai-user-profiles
SNS Topic ARN    : arn:aws:sns:us-east-1:623035187993:smartdocai-alerts
```

**Trước khi bắt đầu:** đảm bảo đã chạy `aws configure` với IAM user có quyền đọc các dịch vụ trên (S3, DynamoDB, Cognito, Lambda, CloudWatch, SNS, CodePipeline, Bedrock).

### 0.1. Cách lấy JWT token thật (cần cho nhiều lệnh bên dưới)

> **Lưu ý:** Dùng `Invoke-RestMethod` thay vì `curl.exe` cho mọi request có JSON body — `curl.exe` trên Windows PowerShell hay bị mất dấu `"` bên trong JSON khi truyền qua `-d`, gây lỗi `JSON decode error`. `Invoke-RestMethod` không bị lỗi này và tự parse luôn JSON response.

> ⚠️ **Bảo mật:** File này sẽ được commit lên GitHub — **KHÔNG bao giờ gõ password thật vào đây**. Luôn thay `<YOUR_PASSWORD>` bằng mật khẩu thật của bạn **trực tiếp trong terminal khi chạy**, không paste password vào file markdown hay commit lên git.

> ⚠️ **Quan trọng:** Backend **KHÔNG có** endpoint `/api/auth/login`. Đăng nhập email/password được xử lý **hoàn toàn ở Frontend** qua thư viện `amazon-cognito-identity-js`, gọi thẳng Cognito bằng giao thức SRP (Secure Remote Password) — không đi qua backend FastAPI. Vì SRP rất phức tạp để làm tay bằng CLI, cách lấy token đáng tin cậy nhất là **đăng nhập qua chính website thật rồi lấy token từ trình duyệt** (xem bên dưới).

**Cách lấy `$TOKEN` (khuyến nghị — dùng được cho mọi tài khoản, kể cả tài khoản đã có sẵn):**

1. Mở trình duyệt, vào `https://dutf3c70nnjzl.cloudfront.net/login`, đăng nhập bằng email/password thật của bạn (nhập trực tiếp trên form, không qua terminal)
2. Sau khi đăng nhập thành công (vào được `/app`), mở DevTools (**F12**) → tab **Console**
3. Dán đoạn script sau vào Console rồi nhấn Enter — script tự tìm và in ra `id_token` đang lưu trong `sessionStorage`:
   ```javascript
   Object.keys(sessionStorage).find(k => k.includes("idToken")) &&
   console.log(sessionStorage.getItem(Object.keys(sessionStorage).find(k => k.includes("idToken"))))
   ```
4. Copy chuỗi token dài in ra (bắt đầu bằng `eyJ...`)
5. Quay lại PowerShell, gán vào biến để dùng cho các lệnh bên dưới:
   ```powershell
   $TOKEN = "eyJ...."   # dán token vừa copy vào đây
   ```

> **Nếu script ở bước 3 chỉ in ra `undefined` mà không thấy giá trị token:** kiểm tra góc trên Console có dòng chữ **"Custom levels"** kèm số **"N hidden"** không — nếu có, nghĩa là DevTools đang ẩn bớt log. Click vào **"Custom levels"** → chọn **"All levels"** (hoặc tích đủ Verbose/Info/Warnings/Errors), rồi chạy lại script.

### 0.2. Helper function gọi API có Bearer token (hiển thị đúng tiếng Việt)

> ⚠️ **Vấn đề quan trọng:** `Invoke-RestMethod`/`Invoke-WebRequest` trên **Windows PowerShell 5.1** thường tự decode JSON response bằng encoding sai (Latin-1 thay vì UTF-8), khiến text tiếng Việt (ví dụ `fullname`) bị vỡ chữ dạng `LÃª Nguyá»n...`. `chcp 65001`/`[Console]::OutputEncoding` ở mục 0 chỉ sửa được cách **hiển thị**, không sửa được lỗi **decode** đã xảy ra trước đó — cần dùng hàm riêng bên dưới.

Chạy đoạn này **1 lần mỗi phiên terminal mới** (sau khi đã gán `$API_BASE` và `$TOKEN` ở mục 0/0.1) để tạo 2 hàm dùng lại cho mọi lệnh GET/POST có Bearer token bên dưới:

```powershell
function Invoke-ApiGet {
  param([string]$Uri, [string]$Token)
  $r = Invoke-WebRequest -Method GET -Uri $Uri -Headers @{Authorization="Bearer $Token"} -UseBasicParsing
  $fixed = [System.Text.Encoding]::UTF8.GetString([System.Text.Encoding]::GetEncoding(28591).GetBytes($r.Content))
  $fixed | ConvertFrom-Json
}

function Invoke-ApiPost {
  param([string]$Uri, [string]$Token, [string]$Body)
  $bodyBytes = [System.Text.Encoding]::UTF8.GetBytes($Body)
  $r = Invoke-WebRequest -Method POST -Uri $Uri -Headers @{Authorization="Bearer $Token"} -ContentType "application/json; charset=utf-8" -Body $bodyBytes -UseBasicParsing
  $fixed = [System.Text.Encoding]::UTF8.GetString([System.Text.Encoding]::GetEncoding(28591).GetBytes($r.Content))
  $fixed | ConvertFrom-Json
}
```

**Cách dùng (thay cho mọi lệnh `Invoke-RestMethod` cần Bearer token trong tài liệu này):**

```powershell
# GET, ví dụ lấy profile
Invoke-ApiGet -Uri "$API_BASE/api/profile" -Token $TOKEN

# POST, ví dụ gọi chat RAG
Invoke-ApiPost -Uri "$API_BASE/api/chat" -Token $TOKEN -Body '{"message":"Tóm tắt nội dung tài liệu vừa upload","use_self_rag":false,"use_co_rag":false}'
```

**Nếu chưa có tài khoản nào, tạo tài khoản test mới** (đăng ký qua backend API vẫn dùng được bình thường vì `/api/auth/register` có tồn tại), sau đó **vẫn phải đăng nhập qua website thật** như hướng dẫn trên để lấy token — không có cách lấy token qua CLI:

```powershell
Invoke-RestMethod -Method POST -Uri "$API_BASE/api/auth/register" `
  -ContentType "application/json" `
  -Body '{"email":"test.workshop@example.com","password":"SecurePass123!","fullname":"Nguyen Van Test","phone":"0901234567","dob":"1990-01-15"}'
```

Kiểm tra email `test.workshop@example.com`, lấy mã 6 số, xác nhận:

```powershell
Invoke-RestMethod -Method POST -Uri "$API_BASE/api/auth/confirm-signup" `
  -ContentType "application/json" `
  -Body '{"email":"test.workshop@example.com","code":"<MA_6_SO>"}'
```

Sau đó **đăng nhập qua website thật** như hướng dẫn ở trên (mở `/login`, đăng nhập, lấy `id_token` từ sessionStorage qua DevTools Console) để có `$TOKEN` — không có endpoint backend nào để login qua CLI.

---

## 1. Phần 5.1.3 — Kiến trúc tổng thể trên AWS ✅ Đã hoàn thành (2/2 ảnh đã có)

Phần này đã có đầy đủ 5 sơ đồ Mermaid (kiến trúc, 3-tier, storage, lambda modules, CI/CD). Bổ sung thêm 2 screenshot để chứng minh kiến trúc là **thật đang chạy**, không chỉ là sơ đồ lý thuyết.

**Lưu vào:** `static/images/5-Workshop/5.1-Workshop-overview/5.1.3-overall-aws-architecture/`

### 1.1. `system-status-live.png` ✅ Đã có ảnh (đã tồn tại trong `static/images/`)
**Nội dung cần thể hiện:** Terminal gọi API health-check thật, chứng minh hệ thống đang chạy production.

> ⚠️ **Lưu ý:** `/api/status` **bắt buộc cần Bearer token** (dùng để trả về số liệu file theo từng user), không phải public health-check như tên gọi. Cần đã có `$TOKEN` từ mục 0.1 trước khi chạy.

**Các bước:**
1. Mở terminal (PowerShell)
2. Chạy lệnh:
   ```powershell
   curl.exe -H "Authorization: Bearer $TOKEN" "https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod/api/status"
   ```
3. Kết quả mong đợi: JSON trả về `{"online": true, "model_ready": true, ...}`, HTTP 200 — nếu thấy `{"detail":"Token không hợp lệ"}` nghĩa là thiếu token hoặc token đã hết hạn (token chỉ sống ~1 tiếng, cần lấy lại theo mục 0.1 nếu hết hạn)
4. Chụp toàn bộ terminal gồm cả lệnh đã gõ và kết quả trả về

### 1.2. `lambda-function-overview.png` ✅ Đã có ảnh (đã tồn tại trong `static/images/`)
**Nội dung cần thể hiện:** Trang Configuration của Lambda function `smartdocai`: Runtime = Container Image, Memory, Timeout, Last modified.

**Các bước:**
1. Đăng nhập AWS Console → tìm kiếm **Lambda** ở thanh search trên cùng
2. Click vào function tên **`smartdocai`**
3. Chọn tab **Configuration** → mục con **General configuration** (menu bên trái)
4. Chụp toàn bộ khung hiển thị Memory, Timeout, Ephemeral storage, Package type = Image

---

## 2. Phần 5.1.4 — Các dịch vụ AWS được sử dụng ✅ Đã hoàn thành (5/5 ảnh đã có/đã xử lý)

**Lưu vào:** `static/images/5-Workshop/5.1-Workshop-overview/5.1.4-aws-services-used/`

### 2.1. `s3-buckets-list.png` ✅ Đã có ảnh (đã tồn tại trong `static/images/`)
**Các bước:**
1. Console → search **S3** → trang **General purpose buckets**
2. Đảm bảo hiển thị đủ 3 bucket: `aws-smartdocsai-cloud`, `codepipeline-us-east-1-...`, `smartdocai-storage-623035187993`
3. Chụp bảng danh sách (giống ảnh mẫu đã xem trong chat trước đó)

### 2.2. `dynamodb-table-encryption.png` ✅ Đã có ảnh (đã tồn tại trong `static/images/`)
**Các bước:**
1. Console → search **DynamoDB** → **Tables** → click `smartdocai-user-profiles`
2. Chọn tab **Additional settings**
3. Cuộn tới mục **Encryption** — phải thấy `AWS KMS key` = `alias/aws/dynamodb`
4. Chụp riêng khung Encryption

**Cách xác nhận trước bằng CLI (không bắt buộc, chỉ để chắc chắn trước khi chụp):**
```powershell
aws dynamodb describe-table --table-name smartdocai-user-profiles --region us-east-1 --query "Table.SSEDescription"
```

### 2.3. `cognito-user-pool-providers.png` ✅ Đã có ảnh (đã tồn tại trong `static/images/`)

> ⚠️ **Cập nhật:** Giao diện Cognito Console đã đổi, không còn mục "Sign-in experience" → "Federated sign-in" như trước.

**Các bước (đã cập nhật theo giao diện mới):**
1. Console → search **Cognito** → **User pools** → click pool có tên `SmartDocUserPool` (ID `us-east-1_3oq5wIiuu`)
2. Menu trái → mục **Authentication** → click **Social and external providers**
3. Phải thấy provider **Google** đã cấu hình trong danh sách
4. Chụp toàn bộ trang danh sách provider

### 2.4. `bedrock-model-access.png` ✅ Đã xử lý — dùng tham chiếu, không cần chụp ảnh riêng

> ⚠️ **Cập nhật:** AWS đã **retire (khai tử) trang "Model access"** — theo thông báo chính thức trên Console: *"Serverless foundation models are now automatically enabled across all AWS commercial regions when first invoked... You no longer need to manually activate model access."* Không còn màn hình "Access granted" thủ công như trước. Ngoài ra menu Bedrock Console hay đổi vị trí (menu **Build** hiện tại không có "Model catalog").

**Đã chốt cách xử lý:** không chụp ảnh riêng cho mục này. Trong file `_index.vi.md` của `5.1.4`, thay vì chèn `<img>` mới, ghi chú text:

```markdown
> Xem ảnh `system-status-live.png` ở mục 5.1.3 — response thực tế đã xác nhận cả 2 model đang hoạt động: `"provider":"Amazon Bedrock","model":"qwen.qwen3-next-80b-a3b","embedding_provider":"Amazon Titan"`.
```

(Nếu vẫn muốn có ảnh Console riêng, xem Cách 2 ở bản lưu trước — không bắt buộc.)

### 2.5. `codepipeline-2-pipelines.png` ✅ Đã có ảnh (đã tồn tại trong `static/images/`)
**Các bước:**
1. Console → search **CodePipeline** → **Pipelines**
2. Phải thấy 2 dòng: `smartdocai-be-pipeline` và `smartdocsai-fe-pipeline`
3. Chụp bảng danh sách, đảm bảo cột **Most recent execution** hiển thị **Succeeded** (màu xanh)

---

## 3. Phần 5.5 — Kiểm thử hệ thống

### 3.1. 5.5.1 — Authentication Testing ✅ Đã hoàn thành (Phúc)

Không cần làm thêm — đã đủ 4/4 ảnh: `registration-success.png`, `email-confirmation-code.png`, `login-jwt-tokens.png`, `cognito-users-list.png`.

### 3.2. 5.5.2 — Document Upload & RAG Testing

**Lưu vào:** `static/images/5-Workshop/5.5-System-testing/5.5.2-Document-RAG/`

Đã có 3/4 ảnh (Phúc).

#### `rag-modes-comparison.png` ✅ Đã xử lý — dùng bảng số liệu thật, không cần ảnh

Đã đo thực tế 3 chế độ RAG qua terminal (xem quá trình điều tra đầy đủ tại `SmartdocAI-AWS/LOGS_CORAG_TIMEOUT_2026-07-24.md`). Kết quả đã viết thẳng thành bảng vào file `content/5-Workshop/5.5-System-testing/5.5.2-Document-RAG/_index.vi.md` (mục "So sánh hiệu năng giữa các chế độ RAG") — không cần chụp ảnh/vẽ chart riêng.

**Phát hiện quan trọng trong lúc đo:** Co-RAG bị lỗi `503` khi Lambda cold start (do cộng dồn overhead cold start + xử lý vượt ngưỡng 29-30s của API Gateway), nhưng chạy bình thường (4-10s, nhanh hơn cả Standard/Self-RAG) khi Lambda đã "ấm". Đây là ví dụ thực tế tốt về đặc điểm cold start trong kiến trúc serverless — đã ghi vào nội dung workshop kèm giải thích.

### 3.3. 5.5.3 — Security Testing

**Lưu vào:** `static/images/5-Workshop/5.5-System-testing/5.5.3-Security/`

Đã có 1/4 ảnh (Phúc: `xss-input-blocked.png`). Còn thiếu 3 ảnh sau:

#### `cors-preflight-headers.png`
**Nội dung cần thể hiện:** Request OPTIONS (preflight) trả về đúng CORS header cho domain CloudFront.

**Các bước:**
1. Mở trình duyệt, truy cập `https://dutf3c70nnjzl.cloudfront.net`
2. Mở DevTools (phím **F12**) → chọn tab **Network**
3. Trong ô filter, gõ `chat` hoặc `upload-url`
4. Thực hiện 1 hành động bất kỳ trên UI (ví dụ gửi 1 tin nhắn chat) để trigger request
5. Trong danh sách request, tìm dòng có **Method = OPTIONS** (thường xuất hiện ngay trước request POST/GET thật)
6. Click vào dòng đó → tab **Headers** → cuộn xuống **Response Headers**
7. Chụp phần Response Headers, đảm bảo thấy dòng `access-control-allow-origin: https://dutf3c70nnjzl.cloudfront.net`

**Cách thay thế bằng terminal (nếu không thấy request OPTIONS trên UI):**
```powershell
curl.exe -X OPTIONS "$API_BASE/api/chat" `
  -H "Origin: https://dutf3c70nnjzl.cloudfront.net" `
  -H "Access-Control-Request-Method: POST" `
  -i
```
Chụp lại toàn bộ response headers trong terminal (chú ý dòng `access-control-allow-origin`).

#### `csrf-state-in-url.png`
**Nội dung cần thể hiện:** URL redirect sang Cognito khi bấm nút Google Login có chứa tham số `state=<UUID>`.

**Các bước:**
1. Mở trình duyệt (khuyến khích tab ẩn danh) → vào `https://dutf3c70nnjzl.cloudfront.net/login`
2. Mở DevTools (**F12**) → tab **Network** → tick chọn **Preserve log**
3. Bấm nút **"Đăng nhập bằng Google"**
4. Trong danh sách Network, tìm request có domain `smartdocai-fayrun2026.auth.us-east-1.amazoncognito.com`, path `/oauth2/authorize`
5. Click vào request đó → xem tab **Headers** → mục **Request URL** hoặc **Query String Parameters**
6. Chụp lại, đảm bảo nhìn rõ tham số `state=` với 1 giá trị dạng UUID (ví dụ `36706f9d-61bb-48a3-...`)

#### `csrf-attack-blocked.png`
**Nội dung cần thể hiện:** Trang `/login` hiển thị lỗi khi giả lập tấn công CSRF (state sai).

**Các bước:**
1. Làm theo 3 bước đầu của mục trên để bắt đầu luồng Google Login, dừng lại ở màn hình chọn tài khoản Google
2. Bấm nút **Back** của trình duyệt để quay lại app (giữ nguyên tab)
3. Mở DevTools → tab **Application** (Chrome) → **Session Storage** → chọn domain CloudFront → copy giá trị key `oauth_state`
4. Dán URL sau vào thanh địa chỉ (thay `<STATE_THAT_KHAC>` bằng bất kỳ chuỗi nào khác với giá trị vừa copy):
   ```
   https://dutf3c70nnjzl.cloudfront.net/auth/callback?code=test123&state=<STATE_KHAC>
   ```
5. Nhấn Enter → trang phải tự động chuyển về `/login` kèm thông báo lỗi
6. Chụp màn hình `/login` lúc đang hiển thị thông báo lỗi (thông báo thường hiện dạng toast/alert đỏ ở góc màn hình, xuất hiện trong vài giây — cần chụp nhanh hoặc dùng chức năng quay màn hình rồi cắt ảnh)

### 3.4. 5.5.4 — Profile Testing ✅ Đã hoàn thành (Phúc)

Không cần làm thêm — đã đủ 2/2 ảnh.

### 3.5. 5.5.5 — Monitoring & Logging (quan trọng nhất, còn thiếu 3/5)

**Lưu vào:** `static/images/5-Workshop/5.5-System-testing/5.5.5-Monitoring/`

Đã có: `cloudwatch-logs-insights-query.png`, 4 ảnh `lambda-metrics-dashboard-*.jpg` (Phúc). Còn thiếu 3 ảnh sau:

#### `cloudwatch-tail-logs-realtime.png`
**Nội dung cần thể hiện:** Log Lambda hiện theo thời gian thực khi có request tới.

**Các bước:**
1. Mở **2 cửa sổ terminal** cạnh nhau
2. Ở terminal 1, chạy lệnh theo dõi log (lệnh này sẽ "treo" và tự in log mới):
   ```powershell
   aws logs tail /aws/lambda/smartdocai --follow --region us-east-1
   ```
3. Ở terminal 2, gọi thử 1 API bất kỳ để tạo log mới, ví dụ (cần `$TOKEN` từ mục 0.1 vì `/api/status` yêu cầu Bearer token):
   ```powershell
   curl.exe -H "Authorization: Bearer $TOKEN" "https://d60866voq5.execute-api.us-east-1.amazonaws.com/prod/api/status"
   ```
4. Quay lại terminal 1, đợi vài giây sẽ thấy dòng log mới xuất hiện tự động
5. Chụp terminal 1 khi đang hiển thị 10-15 dòng log (nhấn `Ctrl+C` để dừng lệnh tail sau khi chụp xong)

#### `cloudwatch-alarms-status.png`
**Nội dung cần thể hiện:** 4 CloudWatch Alarm đã tạo, đang ở trạng thái theo dõi.

**Các bước:**
1. Console → search **CloudWatch** → menu trái **Alarms** → **All alarms**
2. Gõ `smartdocai` vào ô tìm kiếm để lọc
3. Phải thấy đủ 4 dòng: `smartdocai-lambda-errors`, `smartdocai-lambda-duration`, `smartdocai-lambda-throttles`, `smartdocai-apigateway-5xx`
4. Chụp toàn bộ bảng 4 alarm (cột **State** có thể là `OK` hoặc `Insufficient data` — đều hợp lệ)

**Xác nhận trước bằng CLI:**
```powershell
aws cloudwatch describe-alarms --alarm-name-prefix smartdocai --region us-east-1 --query "MetricAlarms[].{Name:AlarmName,State:StateValue}" --output table
```

#### `sns-subscription-confirmed.png`
**Nội dung cần thể hiện:** Email đã confirm nhận cảnh báo từ SNS topic.

**Các bước:**
1. Console → search **SNS** → **Topics** → click `smartdocai-alerts`
2. Chọn tab **Subscriptions** (nếu không thấy tab, cuộn xuống phần Subscriptions ở dưới trang)
3. Phải thấy 1 dòng Protocol = `email`, Status = **Confirmed**
4. Chụp bảng subscription đó

**Xác nhận trước bằng CLI:**
```powershell
aws sns list-subscriptions-by-topic --topic-arn arn:aws:sns:us-east-1:623035187993:smartdocai-alerts --region us-east-1
```
Nếu `SubscriptionArn` là 1 chuỗi ARN thật (không phải `PendingConfirmation`) nghĩa là đã confirm, có thể chụp thẳng output CLI này thay cho ảnh Console.

### 3.6. 5.5.6 — CI/CD Automated Testing

**Lưu vào:** `static/images/5-Workshop/5.5-System-testing/5.5.6-CICD/`

#### `pytest-local-results.png`
**Các bước:**
1. Mở terminal, di chuyển vào thư mục backend:
   ```powershell
   cd C:\Users\lnnta\Downloads\AWS-TT\SmartdocAI-AWS\backend
   ```
2. Cài dependency test nếu chưa có:
   ```powershell
   pip install pytest pytest-cov
   ```
3. Chạy toàn bộ unit test:
   ```powershell
   pytest -v --tb=short --color=yes
   ```
   > ⚠️ **Lưu ý:** **Không** gõ `pytest test_*_unit.py` — PowerShell (khác với bash trên Linux) **không tự expand ký tự `*`** khi gọi chương trình ngoài, nên pytest sẽ nhận nguyên văn chuỗi `test_*_unit.py` làm tên file, không tìm thấy và báo lỗi `file or directory not found`. Chỉ cần chạy `pytest` không kèm tên file — pytest tự động quét mọi file khớp `test_*.py`/`*_test.py` trong thư mục hiện tại (bao gồm `test_auth_service_unit.py`, `test_validators_unit.py`).
4. Đợi chạy xong, chụp toàn bộ terminal — đặc biệt dòng cuối cùng dạng `60 passed in 3.45s`

#### `codebuild-prebuild-test-phase.png`
**Các bước:**
1. Console → search **CodeBuild** → **Build history** → chọn project `smartdocai-be-build`
2. Click vào build gần nhất có status **Succeeded**
3. Cuộn tới phần **Build logs** (hoặc click nút **View entire log**)
4. Dùng `Ctrl+F` tìm chữ `pytest` hoặc `passed` trong log
5. Chụp đoạn log hiển thị dòng chạy `pytest test_*_unit.py` và kết quả `passed`

> **Vì sao lệnh trong CodeBuild dùng được `test_*_unit.py` mà local PowerShell không được:** `buildspec.yml` chạy trên container Linux của CodeBuild, dùng shell `bash` — bash **tự expand** ký tự `*` thành danh sách file thật trước khi truyền cho `pytest`, khác với PowerShell trên Windows.

**Lấy build ID nhanh bằng CLI (nếu muốn xác nhận trước):**
```powershell
aws codebuild list-builds-for-project --project-name smartdocai-be-build --region us-east-1 --query "ids[0:3]"
```

---

## 4. Phần 5.6 — Tổng kết & Dọn dẹp tài nguyên

### 4.1. 5.6.1 — Workshop Summary & Cost

**Lưu vào:** `static/images/5-Workshop/5.6-Conclusion/5.6.1-Summary-Cost/`

#### `billing-cost-explorer.png`
**Các bước:**
1. Console → góc trên bên phải, click tên account → **Billing and Cost Management**
2. Menu trái → **Cost Explorer**
3. Click **Launch Cost Explorer** (nếu lần đầu vào, có thể mất vài phút để load dữ liệu)
4. Chọn khoảng thời gian (**Date range**) trùng với thời gian chạy workshop (ví dụ 7 ngày gần nhất)
5. Ở mục **Group by**, chọn **Service**
6. Chụp biểu đồ cột/đường hiển thị chi phí theo từng dịch vụ

> **Lưu ý:** Cost Explorer có độ trễ dữ liệu ~24h, nếu mới phát sinh chi phí hôm nay có thể chưa hiện đủ.

### 4.2. 5.6.2 — Cleanup Resources

**Lưu vào:** `static/images/5-Workshop/5.6-Conclusion/5.6.2-Cleanup/`

> 🚫 **ĐÃ QUYẾT ĐỊNH BỎ (24/07/2026):** Không chụp 3 ảnh dưới đây cho lần nộp này. Lý do: hệ thống vẫn đang live để demo/chấm điểm, và `resources-deleted-verification.png` chỉ có ý nghĩa **sau khi** đã dọn dẹp thật (chụp lúc này sẽ cho kết quả ngược — resource vẫn còn). Nội dung text + lệnh CLI hướng dẫn cleanup đã có sẵn trong `5.6.2-Cleanup/_index.vi.md` là đủ để thể hiện quy trình, không cần ảnh minh chứng. Nếu sau này dự án thực sự kết thúc và muốn bổ sung ảnh, có thể làm theo hướng dẫn cũ bên dưới (giữ lại để tham khảo).

<details>
<summary>Hướng dẫn cũ (đã bỏ, chỉ giữ tham khảo)</summary>

#### `s3-empty-bucket-progress.png`
**Cách làm bằng Console (không cần terminal, không đụng vào dữ liệu thật):**
1. Console → **S3** → tạo 1 bucket test rỗng, đặt tên bất kỳ (ví dụ `smartdocai-cleanup-demo-test`) — bấm **Create bucket** với cấu hình mặc định
2. Vào bucket vừa tạo → tab **Objects** → **Upload** → chọn 2-3 file bất kỳ trên máy (ảnh, txt, ...) để làm dữ liệu giả
3. Sau khi upload xong, tick chọn ô checkbox ở đầu bảng (chọn tất cả) → bấm nút **Delete**
4. Console hiện màn hình xác nhận, liệt kê danh sách object sẽ xóa + ô yêu cầu gõ chữ **`permanently delete`** để xác nhận
5. Chụp màn hình xác nhận này (đã đủ minh họa quy trình xóa; có thể bấm **Delete objects** để hoàn tất và chụp thêm màn hình báo "Objects deleted successfully" nếu muốn)
6. Xóa luôn bucket test này sau khi chụp xong (**Empty** → **Delete bucket**) để dọn sạch, không để lại rác

> **Cách thay thế (không khuyến nghị, có tính phá hủy thật):** chạy `aws s3 rm s3://smartdocai-storage-623035187993 --recursive --region us-east-1` trên bucket dữ liệu thật — chỉ làm khi workshop đã kết thúc hoàn toàn.

#### `cloudfront-status-disabled.png`
**Cách làm (đã là thao tác Console/chuột sẵn, không cần CLI):**
1. Console → search **CloudFront** → **Distributions**
2. Chụp trước bảng danh sách lúc **Status = Enabled** (trạng thái hiện tại, an toàn, không phá gì)
3. **Chưa bấm Disable ngay** — nút **Disable** đã hiển thị rõ trong ảnh này, đủ để minh họa "biết cách tắt ở đâu"
4. Chỉ khi nào workshop **đã chấm điểm/demo xong hoàn toàn**, quay lại bấm **Disable** trên distribution `dutf3c70nnjzl.cloudfront.net`, đợi Status chuyển sang **Disabled**, rồi chụp thêm ảnh thứ 2 để thay thế/bổ sung

> **Lý do:** đây là hành động phá hủy thật sự (site sẽ ngừng hoạt động) bất kể làm bằng Console hay CLI — nên thời điểm thực hiện mới là yếu tố cần cân nhắc, không phải công cụ.

#### `resources-deleted-verification.png`
**Cách làm bằng Console (không cần terminal) — vào từng trang dịch vụ, gõ `smartdocai` vào ô tìm kiếm, chụp kết quả rỗng:**
1. **EventBridge** → **Rules** → gõ `smartdocai` vào ô search → chụp bảng rỗng "No rules found"
2. **CloudFront** → **Distributions** → gõ `smartdocai` vào ô search (lọc theo Comment) → chụp bảng rỗng
3. **API Gateway** → **APIs** → gõ `smartdocai` vào ô search → chụp bảng rỗng
4. **Lambda** → **Functions** → gõ `smartdocai` vào ô search → chụp bảng rỗng
5. **ECR** → **Repositories** → gõ `smartdocai` vào ô search → chụp bảng rỗng
6. **S3** → **General purpose buckets** → gõ `smartdocai` vào ô search → chụp bảng rỗng

Ghép 6 ảnh nhỏ này thành 1 ảnh tổng hợp (ví dụ dùng Snipping Tool/Paint để xếp lưới 2x3), hoặc lưu riêng từng ảnh rồi chèn cả 6 vào content — tùy cách nào tiện hơn khi trình bày.

> **Cách thay thế bằng CLI (không bắt buộc):**
> ```powershell
> aws events list-rules --region us-east-1 --query "Rules[?contains(Name, 'smartdocai')]"
> aws cloudfront list-distributions --query "DistributionList.Items[?contains(Comment, 'smartdocai')]"
> aws apigatewayv2 get-apis --region us-east-1 --query "Items[?contains(Name, 'smartdocai')]"
> aws lambda list-functions --region us-east-1 --query "Functions[?contains(FunctionName, 'smartdocai')].FunctionName"
> aws ecr describe-repositories --region us-east-1 --query "repositories[?contains(repositoryName, 'smartdocai')].repositoryName"
> aws s3 ls | Select-String "smartdocai"
> ```
> Kết quả mong đợi sau khi đã dọn dẹp thật: tất cả trả về rỗng `[]` hoặc không in ra dòng nào.

</details>

### 4.3. 5.6.3 — Next Steps & References

Phần này thuần văn bản, không cần screenshot minh họa.

---

## 5. Tổng hợp số lượng

| Phần | Số ảnh cần chụp | Đã có | Còn thiếu | Ưu tiên |
|------|------------------|:---:|:---:|---------|
| 5.1.3 | 2 | ✅ 2 (`system-status-live.png`, `lambda-function-overview.png`) | 0 | Thấp — **Hoàn thành** (đã có 5 diagram + 2 ảnh) |
| 5.1.4 | 5 | ✅ 5 (`s3-buckets-list.png`, `dynamodb-table-encryption.png`, `cognito-user-pool-providers.png`, `codepipeline-2-pipelines.png`, 2.4 dùng tham chiếu) | 0 | Trung bình — **Hoàn thành** |
| 5.5.1 | 4 | ✅ 4 | 0 | Cao — **Hoàn thành** |
| 5.5.2 | 4 | ✅ 4 (rag-modes-comparison dùng bảng số liệu, không ảnh) | 0 | Cao — **Hoàn thành** |
| 5.5.3 | 4 (2 mới cho CSRF) | ✅ 1 | 3 (2 ảnh CSRF mới) | Cao |
| 5.5.4 | 2 | ✅ 2 | 0 | Trung bình — **Hoàn thành** |
| 5.5.5 | 5 (2 mới cho Alarms/SNS) | ✅ 2 | 3 (`cloudwatch-tail-logs-realtime`, `cloudwatch-alarms-status`, `sns-subscription-confirmed`) | **Rất cao** |
| 5.5.6 | 2 | 0 | 2 | Trung bình |
| 5.6.1 | 1 | 0 | 1 | Thấp |
| 5.6.2 | 3 | 🚫 Bỏ, không làm | 0 | — (đã quyết định bỏ cho lần nộp này) |
| **TỔNG** | **29 ảnh** (32 − 3 đã bỏ) | **20** | **9** | |

> ⚠️ **Lưu ý:** 6 ảnh vừa đánh dấu ✅ ở 5.1.3/5.1.4 đã tồn tại file trong `static/images/` nhưng **chưa được nhúng `<img>` vào file `_index.vi.md`** tương ứng — cần bổ sung thẻ `<img>` để hiển thị trên trang Hugo (xem mục 6 bên dưới).

## 6. Sau khi chụp xong

1. Copy ảnh vào đúng thư mục `static/images/5-Workshop/.../` theo path đã ghi ở mỗi mục
2. Thêm reference vào file `.vi.md` tương ứng bằng cú pháp:
   ```markdown
   <img src="/images/5-Workshop/<path>/<file>.png" width="90%" style="max-width:900px">
   ```
3. Hugo server (`hugo server -D`) sẽ tự rebuild — F5 lại trình duyệt để kiểm tra ảnh hiển thị đúng
4. Xóa/che thông tin nhạy cảm trước khi chụp nếu cần (Account ID, token thật) — có thể thay bằng giá trị mẫu nếu lo ngại bảo mật khi đưa vào báo cáo công khai
5. Nếu gặp lỗi khi chạy lệnh CLI (ví dụ `AccessDenied`), kiểm tra lại IAM user hiện tại có đủ quyền không bằng lệnh: `aws sts get-caller-identity`

