---
title : "CI/CD Automated Testing"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.5.6 </b> "
---

This section examines how unit tests are integrated into SmartDocAI's CI/CD pipeline: the pytest suite that runs automatically on every code push, the hard-fail configuration that blocks deployment if tests do not pass, and how to debug when a build fails. This layer of protection helps catch bugs before they reach production.

### 1. Pytest Test Suite

**Test files:**
- `backend/test_validators_unit.py` → **50+ tests** (phone, DOB, fullname validation)
- `backend/test_auth_service_unit.py` → **10+ tests** (helper functions, edge cases)

**Pytest command table:**

| Command | Purpose | Example Result |
|---------|---------|----------------|
| `pytest test_*_unit.py -v` | Run all unit tests (verbose) | `60 passed in 3.45s` |
| `pytest test_*_unit.py --cov=modules --cov-report=term-missing` | Run with a code coverage report | Shows % coverage per module + untested lines |
| `pytest test_validators_unit.py::TestPhoneValidation -v` | Run a single test class | `15 passed in 1.2s` |
| `pytest test_validators_unit.py::TestPhoneValidation::test_vn_mobile_format -v` | Run a single test method | `1 passed in 0.5s` |

**Test coverage summary:**
- TestPhoneValidation: 15 tests (VN mobile format, E.164, invalid inputs)
- TestDOBValidation: 12 tests (date range, future dates, format validation)
- TestFullnameValidation: 10 tests (XSS prevention, unicode, length limits)
- TestEdgeCases: 13 tests (None values, empty strings, boundary conditions)
- TestHelperFunctions: 10 tests (phone normalization, timestamps)

---

### 2. CI/CD Pipeline Integration

**Stages in buildspec.yml:**

| Stage | Command | Purpose | Behavior on failure |
|-------|---------|---------|---------------------|
| **pre_build** | `pip install pytest pytest-cov flake8` | Install test dependencies | FAILS the whole build |
| | `flake8 backend/ --exclude=manual_test_*.py --exit-zero` | Lint the code (style check) | Only a warning (exit-zero) |
| | `pytest backend/test_*_unit.py -v --tb=short --strict-markers` | **Run unit tests** | **HARD FAIL** → blocks deployment |
| **build** | `docker build -t smartdocai backend/` | Build the Docker image | FAILS the whole build |
| | `docker tag smartdocai:latest $ECR_REPO_URI:latest` | Tag the image to push to ECR | FAILS the whole build |
| **post_build** | `docker push $ECR_REPO_URI:latest` | Push the image to ECR | FAILS the whole build |
| | `aws lambda update-function-code --function-name smartdocai --image-uri $ECR_REPO_URI:latest` | Deploy to Lambda | FAILS the whole build |

**Pipeline highlights:**
- Tests run at the `pre_build` stage → blocks deployment if a test fails
- Flake8 with `--exit-zero` → does not fail the build due to style issues
- Pytest `--strict-markers` → fails if an undeclared marker is used
- `--tb=short` → concise traceback, faster debugging

---

### 3. Debugging CI/CD Failures

**AWS CLI commands to track builds:**
- View recent builds: `aws codebuild list-builds-for-project --project-name smartdocai-be-build --max-items 10 --region us-east-1`
- View build details: `aws codebuild batch-get-builds --ids <build-id> --region us-east-1`

**Common CI/CD issues & lessons learned:**
1. **Function name mismatch between the test and the real code** → always check the function name before writing a test
2. **SyntaxError inside a comment** → avoid nested docstrings (the Python parser mistakes them for code)
3. **Complex datetime mocking** → timezone-aware datetime requires extra setup (pytest-freezegun)
4. **Wrong assertion** → the assertion must match the code's actual behavior, not the initial assumption
5. **Advantage of hard-fail CI/CD** → blocks deployment early, preventing bugs from reaching production

---
