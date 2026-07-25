---
title : "Automating Lambda Deployment with CodePipeline"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.4.7 </b> "
---

### 1. Check Lambda Function
- Search for `Lambda` in the **AWS Console** search bar.
![find-lambda](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/find-lambda.png)
- Verify that the previously created **smartdocai Function** exists, then click on that function.
![function](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/function.png)
- Check the **Image URI**.
![image-URI](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/image-URI.png)

### 2. Create CodePipeline to Connect to Lambda
**Step 1: Create backend/buildspec.yml (CI/CD Automated Testing & Hard Fail Workflow)**
- In the **backend/** directory, create `buildspec.yml` integrating automated testing (Code Quality Linting with `flake8` and Automated Unit Tests with `pytest` for `test_validators_unit.py` and `test_auth_service_unit.py`). If any linting issues or test cases fail, CodeBuild immediately halts execution (Hard Fail) before packaging the Docker Image:

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

![file-buildspec](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/file-buildspec.png)

**Step 2: Create CodePipeline**
- Access **CodePipeline** and click the **Create pipeline** button.
![codepipeline](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/codepipeline.png)
- In the Choose creation option UI:
  - Select **Build custom pipeline**
  - Click **Next**
![choose-creation-option](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/choose-creation-option.png)
- In the Choose pipeline setting UI:
  - Pipeline name: Enter `smartdocai-be-pipeline`
  - Role name: Enter `AWSCodePipelineServiceRole-us-east-1-smartdocai-be-pipeline`
![choose-pipeline-setting](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/choose-pipeline-setting.png)
- In the Add source stage UI:
  - Source provider: Select **GitHub (via GitHub App)**
  - Connection: Select the **connection** created during CodePipeline connection to S3-Frontend
  - Repository: Select **repo TakunKenjo/SmartdocAI-AWS**
  - Default branch: Select **main**
  - Output artifact format: Select **CodePipeline default**
![add-source-stage1](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/add-source-stage1.png)
  - Scroll to the bottom of the page and select **Next**
![add-source-stage2](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/add-source-stage2.png)
- In the Add build stage - optional UI:
  - Build provider: Select **Other build providers**
  - Select: Select **AWS CodeBuild**
  - Project name: Click the **Create project** button
![add-build-stage](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/add-build-stage.png)

**Step 3: Create CodeBuild BE**
- In the Create build project UI:
  - Project name: Enter `smartdocai-be-build`
  - Project type: Select **Default project**
![create-build-project](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/create-build-project.png)
  - Operating system: Select **Ubuntu**
  - Runtime(s): Select **Standard**
  - Image: Select **aws/codebuild/standard:8.0**
![envi](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/envi.png)
  - Service role: Select **New service role**
  - Role name: Enter `codebuild-smartdocai-be-build-service-role`
![service-role](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/service-role.png)
  - Privileged: Check **Enable this flag…**
![privi](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/privi.png)
  - Build specifications: Select **Use a buildspec file**
  - Buildspec name - optional: Enter `backend/buildspec.yml`
![buildspec-name](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/buildspec-name.png)
  - Scroll to the bottom of the page and click **Continue to CodePipeline**
![continue-to-codepipeline](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/continue-to-codepipeline.png)

**Step 4: Continue Creating CodePipeline**
- In the Add build stage - optional UI:
  - Project name: Select the newly created **CodeBuild**: `smartdocai-be-build`
  - Scroll to the bottom of the page and click **Next**
![add-build-stage-final](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/add-build-stage-final.png)
- In the Add test stage - optional UI:
  - Click **Skip test stage**
![add-test-stage](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/add-test-stage.png)
- In the Add deploy stage UI:
  - Click **Skip deploy stage**
![add-deploy-stage](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/add-deploy-stage.png)
- In the Review UI:
  - Review Pipeline configuration details
![review-pipeline](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/review-pipeline.png)
  - Scroll to the bottom of the page and click **Create pipeline**

**Step 5: Grant ECR Permissions to CodeBuild IAM Role**
- Navigate to IAM > Roles:
  - Select `codebuild-smartdocai-be-service-role`
![IAM-role](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/IAM-role.png)
  - In the Permissions tab: Click **Add permissions**
![tab-permission](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/tab-permission.png)
  - Select **Attach policies**
![attach-policy](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/attach-policy.png)
- Search for: `AmazonEC2ContainerRegistryPowerUser`
  - Check **AmazonEC2ContainerRegistryPowerUser**
  - Click **Add permissions**
![add-permission](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/add-permission.png)
- Permission policy creation result
![permission-policy-success](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/permission-policy-success.png)

**Step 6: Add lambda:UpdateFunctionCode**
- After adding ECR, this role also requires: **lambda:UpdateFunctionCode**
- Click **Add permissions** and select **Create inline policy**
![create-inline-policy](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/create-inline-policy.png)
- In the Specify permissions page:
  - Select **JSON**
  - Enter code as shown below:
```json
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

![json](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/json.png)
  - Scroll to the bottom of the page and click **Next**
![json-next](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/json-next.png)
- In the Review and create UI:
  - Policy name: Enter `smartdocai-be-codebuild-ecr-lambda-policy`
  - Click **Create policy**
![create-inline-policy2](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/create-inline-policy2.png)
- Permission policy creation confirmation
![permission-policy-inline-success](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/permission-policy-inline-success.png)

**Step 7: Verify CodePipeline**
- Select `smartdocai-be-pipeline` and verify build pipeline success
![pipeline-be-success](/images/5-Workshop/5.4-Backend-deployment/5.4.7-automating-lambda-deployment-with-codePipeline/pipeline-be-success.png)
