---
title : "Creating API Gateway"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.4.5 </b> "
---

### 1. Declare PowerShell Variables


![image12.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image12.png)


### 2. Command to Create New REST API


![image13.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image13.png)


Fetch root resource ID (`/`) of API:


![image14.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image14.png)


### 3. Create Cognito Authorizer (Protect API with JWT Token)
![image15.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image15.png)

### 4. Create Proxy Resource  and Method 




#### 4.1. Create Resource `{proxy+}`:


![image16.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image16.png)


#### 4.2. Assign Method `ANY` with Cognito Authorizer:


![image17.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image17.png)


### 5. Integrate API Gateway with AWS Lambda (AWS_PROXY Integration)


![image18.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image18.png)


### 6. Grant Permissions for API Gateway to Invoke Lambda Function


![image19.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image19.png)


### 7. Deploy API Gateway to Stage "prod"


![image20.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image20.png)


Result:


![image21.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image21.png)


### 8. Configure CORS on API Gateway

To prevent browser CORS blocking on Preflight Requests (`OPTIONS`), run command to create OPTIONS method:


![image22.png](/images/5-Workshop/5.4-Backend-deployment/5.4.5-creating-API-gateway/image22.png)
