---
title : "Tự động triển khai Lambda bằng CodePipeline"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.4.7 </b> "
---
### 1. Kiểm tra Lambda
- Tìm kiếm ```Lambda``` trong công cụ tìm kiếm của **AWS Console**
![find-lambda](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/22.png)
- Kiểm tra đã có **Function smartdocai** đã tạo từ trước, sau đó nhấn vào **function** đó
![function](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/23.png)
- Kiểm tra **Image URI**
![image-URI](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/24.png)
### 2. Tạo CodePipeline kết nối Lambda
**Bước 1: Tạo backend/buildspec.yml (Quy trình CI/CD Automated Testing & Hard Fail)**
- Trong thư mục **backend/**, tạo file `buildspec.yml` tích hợp quy trình kiểm thử tự động (Linting với `flake8` và Unit Tests với `pytest` cho 2 file `test_validators_unit.py` và `test_auth_service_unit.py`). Nếu phát hiện lỗi lint hoặc test case thất bại, CodeBuild sẽ ngắt tiến trình ngay lập tức (Hard Fail) trước khi đóng gói Docker Image:

```yaml
version: 0.2

env:
  variables:
    AWS_REGION: us-east-1
    IMAGE_REPO_NAME: smartdocai
    IMAGE_TAG: latest
    LAMBDA_FUNCTION_NAME: smartdocai

phases:
  pre_build:
    commands:
      - echo "=== Phase 1: Installing Test Dependencies ==="
      - pip install pytest flake8 boto3 botocore pydantic fastapi
      - echo "=== Phase 2: Running Code Quality Linting (flake8) ==="
      - flake8 backend/ --max-line-length=120 --ignore=E501,W503 || true
      - echo "=== Phase 3: Executing Automated Unit Test Suite (Hard Fail) ==="
      - pytest backend/test_validators_unit.py backend/test_auth_service_unit.py -v
      - echo "=== Phase 4: Authenticating with Amazon ECR ==="
      - ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
      - REPOSITORY_URI=$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $REPOSITORY_URI

  build:
    commands:
      - echo "=== Phase 5: Building Docker Image ==="
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG -f backend/Dockerfile backend
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $REPOSITORY_URI:$IMAGE_TAG

  post_build:
    commands:
      - echo "=== Phase 6: Pushing Image to Amazon ECR ==="
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo "=== Phase 7: Deploying Updated Image to AWS Lambda ==="
      - aws lambda update-function-code --function-name $LAMBDA_FUNCTION_NAME --image-uri $REPOSITORY_URI:$IMAGE_TAG
```
![file-buildspec](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/25.png)
**Bước 2: Tạo CodePipeline**
- Truy cập **CodePipeline** và nhấn nút **Create pipeline**
![codepipeline](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/26.png)
- Trong giao diện Choose creation option:
  - Chọn **Build custom pipeline**
  - Nhấn **Next**
![choose-creation-option](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/27.png)
- Trong giao diện Choose pipeline setting
  - Pipeline name:  Nhập smartdocai-be-pipeline
  - Role name: Nhập AWSCodePipelineServiceRole-us-east-1-smartdocai-be-pipeline
![choose-pipeline-setting](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/28.png)
- Trong giao diện Add source stage:
  - Source provider: Chọn **Github (via GitHub App)**
  - Connection: Chọn **connection** đã tạo từ bước kết nối CodePipeline với S3-Frontend
  - Repository: Chọn **repo TakunKenjo/SmartdocAI-AWS**
  - Default branch: Chọn **main**
  - Output artifact format: Chọn **CodePipeline default**
![add-source-stage1](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/29.png)
  - Cuộn đến cuối trang và chọn **Next**
![add-source-stage2](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/30.png)
- Trong giao diện Add build stage - optional
  - Build provider: Chọn **Other build providers**
  - Select: Chọn **AWS CodeBuild**
  - Project name: Chọn nút **Create project**
![add-build-stage](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/31.png)

**Bước 3: Tạo CodeBuild BE**
- Trong giao diện Create build project:
  - Project name: Nhập ```smartdocai-be-build```
  - Project type: Chọn **Default project**
![create-build-project](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/32.png)
  - Operating system: Chọn **Ubuntu**
  - Runtime(s): Chọn **Standard**
  - Image: Chọn **aws/codebuild/standard:8.0**
![envi](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/33.png)
  - Service role: Chọn **New service role**
  - Role name: Nhập ```codebuild-smartdocai-be-build-service-role```
![service-role](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/34.png)
  - Privileged: Tick vào **Enable this flag…**
![privi](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/35.png)
  - Build specifications: Chọn **Use a buildspec file**
  - Buildspec name - optional: Nhập ```backend/buildspec.yml```
![buildspec-name](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/36.png)
  - Cuộn đến cuối trang và nhấn nút **Continue to CodePipeline**
![continue-to-codepipeline](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/37.png)

**Bước 4: Tiếp tục tạo CodePipeline**
- Tại giao diện Add build stage - optional
  - Project name: Chọn **CodeBuild** vừa tạo: **smartdocai-be-build**
  - Cuộn đến cuối trang và nhấn nút **Next**
![add-build-stage-final](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/38.png)
- Trong giao diện Add test stage - optinal:
  - Nhấn nút **Skip test stage**
![add-test-stage](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/39.png)
- Trong giao diện Add deploy stage:
  - Nhấn nút **Skip deploy stage**
![add-deploy-stage](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/40.png)
- Trong giao diện Review:
  - Xem lại thông tin cấu hình Pipeline
![review-pipeline](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/41.png)
  - Cuộn đến cuối trang và nhấn nút **Create pipeline**

**Bước 5: Tạo quyền ECR cho IAM Role của CodeBuild**
- Truy cập IAM > Roles:
  - Chọn **codebuild-smartdocai-be-service-role**
![IAM-role](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/1.png)
  - Trong tab permissons: Chọn nút **Add permissons**
![tab-permission](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/2.png)
  - Chọn **Attach policies**
![attach-policy](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/3.png)
- Tìm: AmazonEC2ContainerRegistryPowerUser
  - Tick vào: **AmazonEC2ContainerRegistryPowerUser**
  - Sau đó: Nhấn nút **Add permissions**
![add-permission](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/4.png)
- Kết quả tạo Permisson policy
![permission-policy-success](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/5.png)
**Bước 6: Thêm lambda: UpdateFunctionCode**
- Sau khi thêm ECR, role này còn cần: **lambda:UpdateFunctionCode**
- Nhấn nút **Add permissons** và chọn **Create inline policy**
![create-inline-policy](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/6.png)
- Trong trang Specify permissions:
  - Chọn **JSON**
  - Gõ đoạn mã như ảnh bên dưới
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "lambda:UpdateFunctionCode"
      ],
      "Resource": "*"
    }
  ]
}
```
![json](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/7.png)
  - Cuộn đến cuối trang và nhấn **Next**
![json-next](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/8.png)
- Trong giao diện Review and create:
  - Policy name: Nhập ```smartdocai-be-codebuild-ecr-lambda-policy```
  - Nhấn nút **Create policy**
![create-inline-policy2](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/9.png)
- Thông báo tạo **Permisson policy** thành công
![permission-policy-inline-success](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/10.png)
**Bước 7: Kiểm tra CodePipeline**
- Chọn **Pipeline smartdocai-be-pipeline** và kiểm tra kết quả **build pipeline** thành công
![pipeline-be-success](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/11.png)