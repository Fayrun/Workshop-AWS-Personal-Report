---
title : "Kiểm thử tự động CI/CD"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.5.6 </b> "
---

Phần này kiểm tra quy trình tích hợp unit test vào pipeline CI/CD của SmartDocAI: bộ test pytest chạy tự động mỗi lần push code, cấu hình hard-fail để chặn deploy nếu test không qua, và cách debug khi build gặp lỗi. Đây là lớp bảo vệ giúp phát hiện lỗi trước khi code lọt vào production.

### 1. Bộ test Pytest

**Các file test:**
- `backend/test_validators_unit.py` → **50+ tests** (phone, DOB, fullname validation)
- `backend/test_auth_service_unit.py` → **10+ tests** (hàm hỗ trợ, trường hợp biên)

**Bảng lệnh Pytest:**

| Lệnh | Mục đích | Kết quả ví dụ |
|---------|---------|----------------|
| `pytest test_*_unit.py -v` | Chạy toàn bộ unit test (verbose) | `60 passed in 3.45s` |
| `pytest test_*_unit.py --cov=modules --cov-report=term-missing` | Chạy kèm báo cáo code coverage | Hiển thị % coverage mỗi module + dòng chưa được test |
| `pytest test_validators_unit.py::TestPhoneValidation -v` | Chạy riêng 1 class test | `15 passed in 1.2s` |
| `pytest test_validators_unit.py::TestPhoneValidation::test_vn_mobile_format -v` | Chạy riêng 1 method test | `1 passed in 0.5s` |

**Tóm tắt độ phủ test:**
- TestPhoneValidation: 15 tests (VN mobile format, E.164, invalid inputs)
- TestDOBValidation: 12 tests (date range, future dates, format validation)
- TestFullnameValidation: 10 tests (XSS prevention, unicode, length limits)
- TestEdgeCases: 13 tests (None values, empty strings, boundary conditions)
- TestHelperFunctions: 10 tests (phone normalization, timestamps)

---

### 2. Tích hợp với CI/CD Pipeline

**Các giai đoạn trong buildspec.yml:**

| Giai đoạn | Lệnh | Mục đích | Hành vi khi lỗi |
|-------|---------|---------|---------------------|
| **pre_build** | `pip install pytest pytest-cov flake8` | Cài đặt dependency cho test | FAIL toàn bộ build |
| | `flake8 backend/ --exclude=manual_test_*.py --exit-zero` | Lint code (kiểm tra style) | Chỉ cảnh báo (exit-zero) |
| | `pytest backend/test_*_unit.py -v --tb=short --strict-markers` | **Chạy unit test** | **HARD FAIL** → Chặn deploy |
| **build** | `docker build -t smartdocai backend/` | Build Docker image | FAIL toàn bộ build |
| | `docker tag smartdocai:latest $ECR_REPO_URI:latest` | Tag image để push ECR | FAIL toàn bộ build |
| **post_build** | `docker push $ECR_REPO_URI:latest` | Push image lên ECR | FAIL toàn bộ build |
| | `aws lambda update-function-code --function-name smartdocai --image-uri $ECR_REPO_URI:latest` | Deploy lên Lambda | FAIL toàn bộ build |

**Điểm nổi bật của pipeline:**
- Test chạy ở `pre_build` → chặn deploy nếu test fail
- Flake8 với `--exit-zero` → không fail build vì lỗi style
- Pytest `--strict-markers` → fail nếu dùng marker chưa khai báo
- `--tb=short` → traceback ngắn gọn, debug nhanh hơn

---

### 3. Debug lỗi CI/CD

**Lệnh AWS CLI để theo dõi build:**
- Xem các build gần đây: `aws codebuild list-builds-for-project --project-name smartdocai-be-build --max-items 10 --region us-east-1`
- Xem chi tiết build: `aws codebuild batch-get-builds --ids <build-id> --region us-east-1`

**Các lỗi CI/CD thường gặp & bài học kinh nghiệm:**
1. **Sai tên hàm giữa test và code thật** → luôn kiểm tra tên hàm trước khi viết test
2. **SyntaxError trong comment** → tránh nested docstring (Python parser hiểu nhầm thành code)
3. **Mock datetime phức tạp** → timezone-aware datetime cần setup thêm (pytest-freezegun)
4. **Assertion sai** → assertion phải khớp với hành vi thực tế của code, không phải giả định ban đầu
5. **Lợi thế của hard-fail CI/CD** → chặn deploy sớm, tránh bug lọt vào production

---