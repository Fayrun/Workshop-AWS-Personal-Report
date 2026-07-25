---
title : "Đặc tả giao diện và chức năng hệ thống"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.1.5 </b> "
---

### 1.Giao diện Đăng nhập

![image56.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/image56.png)

#### 1.1.Tổng quan giao diện
Chức năng cho phép người dùng đã có tài khoản thực hiện xác thực identity để truy cập vào hệ thống **SmartDocAI**. Hệ thống hỗ trợ 2 phương thức xác thực chính:
- **Đăng nhập truyền thống (Native Authentication)**: Sử dụng Email và Mật khẩu.
- **Đăng nhập đơn (SSO - Social Login)**: Sử dụng tài khoản Google thông qua AWS Cognito Hosted UI (OAuth2 / OIDC).

#### 1.2. Thiết kế bố cục và thành phần giao diện

| STT | Tên thành phần | Loại Component | Mô tả / Quy cách hiển thị |
| --- | --- | --- | --- |
| 1 | Tiêu đề Form | `<h2>`, `<p>` | Thể hiện tiêu đề "Đăng nhập" cùng liên kết "Đăng ký ngay!" chuyển sang trang `/register`. |
| 2 | Email | Input (Email) | Ô nhập Email cá nhân. Tích hợp Icon Mail ở góc trái. |
| 3 | Mật khẩu | Input (Password) | Ô nhập Mật khẩu. Tích hợp Icon Lock ở góc trái và Nút ẩn/hiện mật khẩu (Icon Eye/EyeOff) ở góc phải. |
| 4 | Nút "Đăng nhập" | Button | Nút hành động chính (Primary Button). Hiển thị trạng thái xoay (Spinner) "Đang đăng nhập..." khi request đang được xử lý. |
| 5 | Đường phân cách | Divider | Dòng chữ "hoặc tiếp tục với" ngăn cách 2 phương thức đăng nhập. |
| 6 | Nút "Đăng nhập bằng Google" | Button (Social) | Nút bấm chứa biểu tượng chính thức của Google SVG, tạo hiệu ứng chuyển sáng (Shimmer hover effect). Khi click sẽ chuyển hướng tới Cognito Hosted UI. |

#### 1.3. Quy tắc kiểm thử đầu vào của dữ liệu

| Trường dữ liệu | Quy tắc kiểm tra (Validation Rules) | Thông báo lỗi hiển thị |
| --- | --- | --- |
| Email | - Không được để trống.<br/>- Phải khớp với định dạng Email chuẩn (user@domain.com). | - "Vui lòng nhập email."<br/>- "Email không hợp lệ." |
| Mật khẩu | Không được để trống. | "Vui lòng nhập mật khẩu." |

#### 1.4. Cơ chế Xử lý Ngoại lệ và An toàn Hệ thống

**Cơ chế Tự động Tra cứu Username (Automatic Username Resolution):**
Trong trường hợp người dùng tạo tài khoản ban đầu bằng Google OAuth, Cognito sẽ sinh một `username` nội bộ dạng `Google_10293847...`.
Khi người dùng cố gắng đăng nhập thủ công bằng ô nhập Email/Mật khẩu (sau khi đã thiết lập mật khẩu native), Cognito sẽ ném lỗi `UserNotFoundException` vì username nội bộ không phải là địa chỉ email.
Hệ thống được trang bị cơ chế tự động: Ngay khi bắt được lỗi `UserNotFoundException`, Redux Thunk sẽ gọi API `/api/auth/resolve-login-username` để tra cứu username thật gắn với email đó và thực hiện tự động thử lại (retry) mà không yêu cầu người dùng thao tác lại.

**Lưu trữ Session & Token An toàn:**
JWT Tokens và thông tin phiên (`id_token`, `access_token`, `refresh_token`, `auth_user`) được lưu trữ tại `sessionStorage`, giúp tự động xóa session khi người dùng đóng tab trình duyệt, tránh rủi ro đe dọa an toàn thông tin trên máy tính công cộng.

**Phản hồi Trạng thái & Vô hiệu hóa Spam Form:**
Nút "Đăng nhập" lập tức chuyển sang trạng thái `disabled` và bật Spinner khi đang trao đổi dữ liệu với AWS Cognito, giúp phòng tránh việc người dùng nhấp đúp hoặc gửi nhiều request trùng lặp.

---

### 2.Giao diện Đăng ký

Thiết kế dạng 2 bước trên cùng 1 giao diện:
- **Bước 1 ( Register )**: Nhập thông tin cá nhân và mật khẩu

![image57.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/image57.png)

- **Bước 2 : ( Confirm OTP )**: Nhập mã xác thực 6 chữ số gửi tự động qua Email để kích hoạt tài khoản.

![image58.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/image58.png)

#### 2.1.Tổng quan giao diện
Giao diện Đăng ký tài khoản (`/register`) đóng vai trò là điểm tiếp nhận thông tin người dùng mới vào hệ thống **SmartDocAI**. Giao diện được thiết kế theo phong cách giao diện tối (Dark Mode) hiện đại, trực quan, tuân thủ các quy chuẩn UX/UI nhằm tối ưu hóa trải nghiệm người dùng.

#### 2.2.Thiết kế bố cục và thành phần giao diện

##### 1. Giao diện Bước 1: Nhập thông tin đăng ký

| STT | Tên thành phần | Loại Component | Mô tả / Quy cách hiển thị |
| --- | --- | --- | --- |
| 1 | Tiêu đề Form | `<h2>`, `<p>` | Thể hiện tiêu đề "Tạo tài khoản" cùng đường dẫn liên kết "Đăng nhập" nếu đã có tài khoản. |
| 2 | Họ và tên | Input (Text) | Nhập tên đầy đủ của người dùng. Tích hợp Icon User. |
| 3 | Email | Input (Email) | Nhập địa chỉ Email cá nhân (dùng làm Username xác thực Cognito). Tích hợp Icon Mail. |
| 4 | Điện thoại | Input (Tel) | Nhập số điện thoại liên hệ. Tích hợp Icon Phone. Hiển thị cùng dòng với Ngày sinh trên màn hình Desktop. |
| 5 | Ngày sinh | Input (Date) | Chọn ngày tháng năm sinh. Tích hợp Icon Calendar. |
| 6 | Mật khẩu | Input (Password) | Nhập mật khẩu. Tích hợp Icon Lock và Nút ẩn/hiện mật khẩu (Icon Eye/EyeOff). |
| 7 | Xác nhận mật khẩu | Input (Password) | Nhập lại mật khẩu để kiểm tra trùng khớp. Tích hợp Nút ẩn/hiện mật khẩu. |
| 8 | Nút "Tạo tài khoản" | Button | Nút hành động chính (Primary Button). Khi bấm sẽ hiển thị hiệu ứng Loading Spin nếu đang gửi request. |

##### 2.Giao diện Bước 2 : Xác thực mã OTP

| STT | Tên thành phần | Loại Component | Mô tả / Quy cách hiển thị |
| --- | --- | --- | --- |
| 1 | Thông báo Email | `<p>` | Hiển thị địa chỉ Email vừa đăng ký (tô nổi bật bằng màu Blue) để nhắc người dùng kiểm tra hộp thư. |
| 2 | Mã xác thực (OTP) | Input (Text) | Ô nhập mã 6 chữ số, ký tự được căn giữa (Center-align), định dạng phông chữ to rõ (tracking-widest). |
| 3 | Nút "Kích hoạt tài khoản" | Button | Gửi mã OTP lên Cognito/Backend API để hoàn tất xác thực. |
| 4 | Nút "Quay lại đăng ký" | Button (Text) | Cho phép chuyển về Bước 1 nếu người dùng muốn sửa lại thông tin hoặc Email. |

#### 2.3.Quy tắc kiểm thử đầu vào của dữ liệu

| Trường dữ liệu | Quy tắc kiểm tra (Validation Rules) | Thông báo lỗi hiển thị |
| --- | --- | --- |
| Họ và tên | Không được để trống hoặc chỉ chứa khoảng trắng. | "Vui lòng nhập họ tên." |
| Email | - Không được để trống.<br/>- Phải đúng định dạng Email (name@domain.com). | - "Vui lòng nhập email."<br/>- "Email không hợp lệ." |
| Số điện thoại | - Không được để trống.<br/>- Phải chứa từ 9 đến 11 chữ số. | - "Vui lòng nhập số điện thoại."<br/>- "Số điện thoại không hợp lệ." |
| Ngày sinh | - Không được để trống.<br/>- Năm sinh phải trong khoảng 1900 đến Năm hiện tại.<br/>- Không được chọn ngày ở tương lai. | - "Vui lòng chọn ngày sinh."<br/>- "Năm sinh phải từ 1900 đến [Năm hiện tại]."<br/>- "Ngày sinh không được ở tương lai." |
| Mật khẩu | - Không được để trống.<br/>- Độ dài tối thiểu 6 ký tự. | - "Vui lòng nhập mật khẩu."<br/>- "Mật khẩu phải từ 6 ký tự." |
| Xác nhận mật khẩu | Giá trị phải trùng khớp hoàn toàn với ô Mật khẩu. | "Mật khẩu không khớp." |
| Mã xác thực (OTP) | Không được để trống khi nhấn Kích hoạt. | "Vui lòng nhập mã xác thực." |

#### 2.4. Xử lý các trường hợp đặc biệt & An toàn dữ liệu

**Cơ chế Account Linking (PreSignUp Lambda):** Khi người dùng từng đăng ký tài khoản Native qua Email/Mật khẩu nhưng sau đó đăng nhập bằng nút Google OAuth, hàm PreSignUp Trigger sẽ chủ động liên kết Google Provider vào User ID hiện có thay vì tạo User mới. Cơ chế này tránh phân mảnh tài khoản và đảm bảo không bị mất dữ liệu tài liệu cá nhân lưu trữ trên Amazon S3 / DynamoDB.

**Tự động Khởi tạo Profile (DynamoDB `ensure_user_profile`):** Trong trường hợp người dùng lần đầu truy cập qua Google, backend API tự động khởi tạo hồ sơ chuẩn với `user_id` chính là `sub` từ Cognito JWT, đặt `password_set = false` để đánh dấu tài khoản chưa thiết lập mật khẩu native.

**Vô hiệu hóa đa lần nhấn (Debounce & Loading state):** Nút bấm sẽ chuyển sang trạng thái `disabled` kèm biểu tượng quay (Spinning Loader) trong khi đợt request API đang thực thi, loại bỏ rủi ro gửi trùng lặp yêu cầu (Spam requests).

---

### 3.Giao diện Trang chủ

![image59.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/image59.png)

#### 3.1.Tổng quan giao diện
Trang chủ hay Không gian làm việc chính (`/app`) là trung tâm điều khiển của hệ thống **SmartDocAI**. Màn hình này tích hợp toàn bộ quy trình làm việc với tài liệu: từ khâu tải lên, cài đặt thông số phân đoạn, tùy chỉnh các chiến lược tìm kiếm RAG (Standard, Hybrid, Self-RAG, Co-RAG) cho đến việc thực hiện hỏi đáp thông minh và truy xuất nguồn trích dẫn.
- **Đường dẫn ứng dụng**: `/app` (Yêu cầu xác thực tài khoản qua `ProtectedRoute`).
- **Phong cách Thiết kế**: Giao diện tối/sáng (Dark/Light Mode) tương tác linh hoạt, bố cục 2 cột (2-Column Responsive Layout) tối ưu cho việc tra cứu tài liệu song song với giao tiếp AI.
- **Tối ưu trải nghiệm (UX)**: Đạt tính tương thích cao trên thiết bị di động với cơ chế Sidebar trượt (Mobile Drawer Overlay) mở bằng nút Hamburger Menu.

#### 3.2.Thiết kế bố cục và thành phần giao diện
Bố cục Trang chủ được chia thành 2 khu vực chức năng chính: **Thanh Điều hướng & Quản lý Tài liệu (Sidebar - Trái)** và **Khu vực Trò chuyện & Hỏi đáp AI (Chat Area - Phải)**.

#### 3.3.Các thành phần chính trong giao diện trang chủ

##### 3.3.1. Thanh Điều hướng & Quản lý Tài liệu (SmartSidebar)

- **Giao diện Header và logo**:

![rId68.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId68.png)

  - *Mô tả chức năng*: Hiển thị tên thương hiệu "SmartDocAI", nút Thu gọn/Mở rộng Sidebar (Collapse), nút Chuyển đổi giao diện Sáng/Tối.

- **Giao diện trạng thái hệ thống**:

![rId69.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId69.png)

  - *Mô tả chức năng*: Hiển thị đèn báo xanh/đỏ thể hiện kết nối tới dịch vụ AI (Cloud LLM / Bedrock).

- **Giao diện thống kê KPI**:

  - *Trước khi đăng tải file*:

![rId70.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId70.png)

  - *Sau khi đăng tải file*:

![rId71.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId71.png)

  - *Mô tả chức năng*: Thẻ hiển thị các chỉ số tổng quan: Số file đã tải lên, Tổng số trang (`totalPages`), Tổng số đoạn văn bản (`totalChunks`).

- **Giao diện cấu hình phân đoạn**:

![rId72.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId72.png)

  - *Mô tả chức năng*: Điều chỉnh thanh trượt: Chunk Size (kích thước đoạn văn bản, mặc định 500-1000 tokens) và Chunk Overlap (độ chồng lấp, mặc định 100-200 tokens).

- **Giao diện khu vực tải file**:

  - *Trước khi tải file*:

![rId73.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId73.png)

  - *Sau khi tải file*:

![rId74.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId74.png)

  - *Mô tả chức năng*: Khung Kéo & Thả (Drag & Drop) hoặc bấm chọn file. Hỗ trợ định dạng PDF, DOCX với kích thước tối đa quy định.

- **Giao diện danh sách tài liệu**:

  - *Trước khi xử lí tài liệu*:

![rId75.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId75.png)

  - *Sau khi xử lí tài liệu*:

![rId76.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId76.png)

  - *Mô tả chức năng*: Hiển thị danh sách các tài liệu đã xử lý thành công, kèm thông tin định dạng, số trang và nút Xóa từng tài liệu.

- **Giao diện các nút thao tác**:
  - *Mô tả chức năng*: Xóa toàn bộ lịch sử của các đoạn hội thoại và xóa toàn bộ tài liệu đã đăng tải lên hệ thống.

- **Giao diện lịch sử hội thoại**:

  - *Trước khi người dùng chưa đặt câu hỏi*:

![rId78.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId78.png)

  - *Sau khi người dùng đặt câu hỏi*:

![rId79.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId79.png)

  - *Mô tả chức năng*: Liệt kê các phiên thảo luận trước đó, cho phép chọn phiên làm việc hoặc xóa lịch sử chat.

##### 3.3.2. Khu vực Trò chuyện & Hỏi đáp AI (ChatArea)

- **Giao diện khi người dùng chưa sử dụng**:

![rId79.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId79.png)

  - *Mô tả chức năng*: Hiển thị khi chưa có cuộc hội thoại nào. Hướng dẫn 3 bước sử dụng: 1. Tải tài liệu → 2. Hệ thống xử lý → 3. Đặt câu hỏi.

- **Giao diện khi người dùng đã tiến hành đặt câu hỏi**:

![rId80.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId80.png)

  - *Mô tả chức năng*: Khung hiển thị chuỗi câu hỏi (User) và câu trả lời (SmartDocAI).

- **Giao diện trích dẫn nguồn**:

![rId81.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId81.png)

  - *Mô tả chức năng*: Khối hiển thị dưới mỗi câu trả lời AI, chỉ rõ nguồn tài liệu tham khảo (Tên file, Trang số bao nhiêu, và đoạn nội dung gốc tương ứng).

- **Giao diện bảng tìm kiếm nâng cao**:

![rId82.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId82.png)

  - *Mô tả chức năng*: Panel mở rộng (Expander) cho phép cấu hình các thuật toán RAG: Lọc file, Hybrid Search, Re-ranking, Self-RAG, Co-RAG.

- **Giao diện thanh nhập câu hỏi**:

![rId83.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId83.png)

  - *Mô tả chức năng*: Khung nhập liệu đa dòng (Textarea tự động co giãn theo văn bản), nút Gửi tin nhắn (SendIcon).

#### 3.3.Quy tắc và tùy chỉnh chiến lược tìm kiếm 

Bảng điều khiển `SearchSettings` cho phép người dùng linh hoạt bật/tắt các công nghệ RAG nâng cao theo từng bài toán tìm kiếm:

| Chiến lược / Tính năng | Cơ chế hoạt động | Trường hợp sử dụng tối ưu |
| --- | --- | --- |
| Lọc tài liệu (File Filter) | Giới hạn không gian tìm kiếm Vector trong phạm vi một hoặc nhiều tập tin cụ thể do người dùng tick chọn. | Tìm kiếm dữ liệu trong đúng 1 báo cáo/hợp đồng chỉ định mà không bị lẫn sang tài liệu khác. |
| Hybrid Search | Kết hợp Vector Search (Bedrock Titan Embeddings V2 60%) và Keyword Search (Thuật toán BM25 40%). | Khi câu hỏi chứa các thuật ngữ chuyên ngành, mã định danh, số hợp đồng hoặc từ khóa chính xác. |
| Re-ranking | Sử dụng mô hình Cross-Encoder đa ngôn ngữ (`cross-encoder/mmarco-mMiniLMv2-L12-H384-v1`) để xếp hạng lại Top-K đoạn văn bản có độ liên quan cao nhất (tối ưu hóa xử lý tiếng Việt). | Loại bỏ các kết quả nhiễu, tăng độ chính xác thông tin truyền vào prompt cho LLM. |
| Self-RAG | Quy trình AI tự phản hồi & đánh giá gồm 3 bước: Query Rewriting (Viết lại câu hỏi), Relevance Filtering (Lọc độ liên quan), Answer Grading (Chấm điểm câu trả lời). | Dùng cho các câu hỏi phức tạp, yêu cầu AI tự sửa lại câu hỏi nếu thông tin tìm kiếm ban đầu không đủ. |
| Co-RAG (Multi-Agent) | Kiến trúc đa tác tử hợp lực: Semantic Agent (Vector), Keyword Agent (BM25), Conceptual Agent (LLM Concept). Gộp kết quả theo các chiến lược: voting, union, intersection. | Yêu cầu mức độ tổng hợp tri thức chuyên sâu từ nhiều khía cạnh khác nhau trong tài liệu. |

#### 3.4.Xử lí trạng thái và ngoại lệ trên trang chủ 
- **Chưa có tài liệu được chọn/tải lên**: Khung chat hiển thị màn hình chào mừng WelcomeHero với các gợi ý từng bước. Nút kích hoạt các chế độ RAG nâng cao trong SearchSettings tự động chuyển sang trạng thái disabled.
- **Quá trình xử lý file bị lỗi (File Corrupted / Format Mismatch)**: Hệ thống lập tức bắn thông báo Toast cảnh báo lỗi, giữ nguyên giao diện không làm đứt quãng trải nghiệm.
- **Phân quyền & Cách ly dữ liệu (Per-User Isolation)**: Mọi tài liệu và lịch sử chat đều gắn liền với user_id (Cognito sub). Người dùng A không thể xem hoặc truy vấn tài liệu của Người dùng B trên S3/Vector Index.

---

### 4.Giao diện Thông tin cá nhân

#### 4.1. TỔNG QUAN GIAO DIỆN

- **Giao diện thay đổi thông tin người dùng**:

![rId84.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId84.png)

- **Giao diện đổi mật khẩu**:

![rId85.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId85.png)

Giao diện Thông tin cá nhân (`Profile Settings`) cho phép người dùng xem và cập nhật hồ sơ cá nhân trong hệ thống **SmartDocAI**:
- **Phong cách thiết kế**: Theo chuẩn Dark Mode với bảng màu xám/xanh tím đậm chủ đạo (Deep Dark Theme), giao diện dạng Card viền mỏng hiện đại.
- **Thanh điều hướng Tab (Tab Bar)**:
  - **Tab Thông tin cá nhân (Active)**: Giao diện cập nhật hình ảnh đại diện và các thông tin hồ sơ.
  - **Tab Bảo mật**: Giao diện dành cho các thiết lập bảo mật và đổi mật khẩu tài khoản.
- **Cấu trúc 2 Card chính**:
  - **Card bên trái (Ảnh đại diện)**: Hiển thị avatar người dùng (chữ cái đầu hoặc ảnh upload), hỗ trợ kéo-thả hoặc click chọn ảnh cập nhật (JPG, PNG, WEBP, tối đa 5MB).
  - **Card bên phải (Thông tin cá nhân/ Bảo mật)**: Tùy thuộc vào Tab được chọn, sẽ hiển thị Form quản lý các trường thông tin (Họ tên, Email, Số điện thoại, Ngày sinh) hoặc Form cập nhật mật khẩu đăng nhập.

#### 4.2. THIẾT KẾ BỐ CỤC VÀ THÀNH PHẦN GIAO DIỆN (UI SPECIFICATION)

| STT | Tên thành phần | Loại Component | Mô tả / Quy cách hiển thị |
| --- | --- | --- | --- |
| 1 | Nút Quay lại | Button / Link | Text ← Quay lại, nằm ở góc trên bên trái, dùng để quay lại màn hình Chat / Dashboard |
| 2 | Header User Info | Avatar & Text | Hiển thị avatar tròn chứa chữ cái đầu (N) kèm Họ tên (nam) và Email (namnp6567@gmail.com) ở góc trên |
| 3 | Tab Bar | Navigation Tabs | Gồm 2 tab: Thông tin cá nhân (mặc định được chọn) và Bảo mật |
| 4 | Title Card Ảnh đại diện | Heading 3 | Dòng chữ Ảnh đại diện nằm ở trên cùng của Card bên trái |
| 5 | Vùng hiển thị Avatar | Circle Avatar | Khung hình tròn màu xanh hiển thị chữ cái đầu tên người dùng (VD: N) hoặc hình ảnh đại diện đã upload thành công |
| 6 | Khung Kéo & Thả ảnh | Dropzone Container | Khung viền nét đứt (Dashed border) với dòng hướng dẫn: "Kéo & thả ảnh vào đây hoặc click để chọn - JPG, PNG, WEBP - tối đa 5MB" |
| 7 | Nút Đổi ảnh | Secondary Button | Nút viền mỏng kèm icon máy ảnh và chữ Đổi ảnh, cho phép mở cửa sổ chọn tập tin từ máy tính |
| 8 | Title Card Thông tin | Heading 3 & Subtitle | Tiêu đề Thông tin cá nhân kèm dòng mô tả phụ "Cập nhật thông tin hiển thị của tài khoản bạn" ở đầu Card bên phải (khi ở Tab Thông tin) |
| 9 | Trường HỌ VÀ TÊN | Input Textbox | Label: HỌ VÀ TÊN, hiển thị giá trị họ tên hiện tại (VD: nam), cho phép nhập từ 2 - 100 ký tự |
| 10 | Trường EMAIL | Input Textbox | Label: EMAIL, hiển thị email tài khoản hiện tại (VD: namnp6567@gmail.com) |
| 11 | Trường SỐ ĐIỆN THOẠI | Input Textbox | Label: SỐ ĐIỆN THOẠI, hiển thị số điện thoại (VD: +84973251414), tự động chuẩn hóa dạng E.164 |
| 12 | Trường NGÀY SINH | Datepicker Input | Label: NGÀY SINH, chứa icon lịch hiển thị ngày sinh (VD: 09/01/2001), hỗ trợ chọn ngày/tháng/năm |
| 13 | Nút Lưu thay đổi | Primary Button | Nút bấm màu xanh lam nổi bật với icon đĩa lưu và chữ Lưu thay đổi, gửi request cập nhật dữ liệu về Backend |
| 14 | Title Card Bảo mật | Heading 3 & Subtitle | Tiêu đề Bảo mật kèm dòng mô tả phụ "Thay đổi mật khẩu đăng nhập của bạn" ở đầu Card bên phải (khi ở Tab Bảo mật) |
| 15 | Trường MẬT KHẨU HIỆN TẠI | Input Password | Label: MẬT KHẨU HIỆN TẠI, hiển thị dưới dạng ẩn ký tự, tích hợp Nút ẩn/hiện mật khẩu (Icon Eye/EyeOff) ở góc phải |
| 16 | Trường MẬT KHẨU MỚI | Input Password | Label: MẬT KHẨU MỚI, có placeholder "Tối thiểu 6 ký tự", tích hợp Nút ẩn/hiện mật khẩu (Icon Eye/EyeOff) |
| 17 | Trường XÁC NHẬN MẬT KHẨU MỚI | Input Password | Label: XÁC NHẬN MẬT KHẨU MỚI, có placeholder "Nhập lại mật khẩu mới", tích hợp Nút ẩn/hiện mật khẩu (Icon Eye/EyeOff) |
| 18 | Dòng cảnh báo | Text | Hiển thị dòng chú thích: "Mật khẩu mới sẽ được áp dụng cho tất cả thiết bị đăng nhập tiếp theo." |
| 19 | Nút Cập nhật mật khẩu | Primary Button | Nút bấm màu xanh lam nổi bật với icon đĩa lưu và chữ Cập nhật mật khẩu để gửi request thay đổi mật khẩu |

#### 4.3. QUY TẮC KIỂM THỬ ĐẦU VÀO CỦA DỮ LIỆU (VALIDATION RULES MATRIX)

| Trường dữ liệu | Quy tắc kiểm tra (Validation Rules) | Thông báo lỗi hiển thị |
| --- | --- | --- |
| Họ và tên | Bắt buộc nhập, không được để trống | "Họ tên không được để trống" |
| Họ và tên | Độ dài từ 2 đến 100 ký tự, không chứa ký tự đặc biệt nguy hiểm hoặc mã HTML/Script (chống XSS) | "Tên người dùng phải từ 2-100 ký tự, không chứa ký tự đặc biệt" |
| Email | Bắt buộc nhập, không được để trống | "Email không được để trống" |
| Email | Đảm bảo đúng định dạng RFC 5322 Email chuẩn (user@domain.com) | "Email không hợp lệ" |
| Số điện thoại | Bắt buộc nhập, không được để trống | "Số điện thoại không được để trống" |
| Số điện thoại | Chấp nhận định dạng Việt Nam (0901234567) hoặc chuẩn quốc tế E.164 (+84901234567), độ dài 10 - 15 chữ số | "Số điện thoại không hợp lệ" |
| Ngày sinh | Bắt buộc nhập, không được để trống | "Ngày sinh không được để trống" |
| Ngày sinh | Đúng định dạng YYYY-MM-DD, năm sinh trong khoảng từ năm 1900 đến năm hiện tại | "Ngày sinh phải theo định dạng YYYY-MM-DD, năm từ 1900 đến 2026" |
| Ảnh đại diện | Định dạng file phải là ảnh: JPG, PNG, WEBP | "Định dạng tập tin ảnh không hỗ trợ. Chỉ chấp nhận JPG, PNG, WEBP" |
| Ảnh đại diện | Kích thước file tối đa không vượt quá 5 MB | "Ảnh quá lớn (...MB), tối đa 5MB" |
| Mật khẩu hiện tại | Bắt buộc nhập, không được để trống (nếu tài khoản đã set mật khẩu native) | "Vui lòng nhập mật khẩu hiện tại" |
| Mật khẩu mới | Bắt buộc nhập, độ dài tối thiểu 6 ký tự | "Mật khẩu mới phải từ 6 ký tự trở lên" |
| Xác nhận mật khẩu mới | Bắt buộc nhập, giá trị phải trùng khớp hoàn toàn với ô Mật khẩu mới | "Mật khẩu xác nhận không khớp" |

#### 4.4. XỬ LÝ CÁC TRƯỜNG HỢP ĐẶC BIỆT & AN TOÀN DỮ LIỆU
- **Xử lý linh hoạt cờ `password_set`**: Đảm bảo người dùng đăng nhập qua Google OAuth lần đầu không bị chặn bởi ô "Mật khẩu hiện tại", cho phép tạo mật khẩu native dễ dàng để từ đó về sau có thể đăng nhập bằng cả 2 phương thức.
- **Kiểm soát file ảnh đầu vào**: Tự động chặn các file không phải định dạng ảnh hoặc có dung lượng gốc lớn hơn 5MB trước khi thực hiện nén.
- **Đồng bộ giao diện tức thì (Reactive UI Sync)**: Khi ảnh đại diện hoặc Họ tên thay đổi ở trang Profile, Redux Store lập tức cập nhật state chung, giúp thông tin ở Thanh Sidebar và Header thay đổi đồng bộ ngay lập tức mà không cần F5 reload trang.
