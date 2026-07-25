---
title : "Implementing Frontend API Gateway Integration"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.4.6 </b> "
---

### 1. Create a New Origin on CloudFront Pointing to API Gateway
- Log in to the AWS Console and navigate to CloudFront service.
- Select your **Distribution** (**ID: E5QRZJSUK0QTJ**).
![distribution-id](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/distribution-id.png)
- Select the **Origins** tab and click **Create origin**.
![tab-origin](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/tab-origin.png)
- Configure the following settings:
  - Origin domain: Select or enter the **API Gateway address d60866voq5.execute-api.us-east-1.amazonaws.com**
  - Protocol: Select **HTTPS only**.
  - Origin path: Enter `/prod`
  - Name: Enter `APIGatewayBackend`
![create-origin](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/create-origin.png)
- Keep other settings at default and click **Create origin**.
![create-origin-next](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/create-origin-next.png)

### 2. Create Behavior to Route /api/* Requests
- Switch to the **Behaviors** tab and click **Create behavior**.
![tab-behavior](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/create-behavior.png)
- Configure the following key parameters:
  - Path pattern: Enter `/api/*` (to intercept all requests sent to backend).
  - Origin and origin groups: Select the **Origin APIGatewayBackend** created in Step 1.
  - Viewer protocol policy: Select **Redirect HTTP to HTTPS.**
  - Allowed HTTP methods: Select **GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE** (Important: Must select this to support S3 file uploads, POST chat requests, and document deletions).
![create-behavior1](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/create-behavior1.png)
- Cache key and origin requests:
  - Select **Cache policy and origin request policy (recommended)**.
  - Cache policy: Select **CachingDisabled**.
  - Origin request policy: Select the managed AWS policy named **AllViewerExceptHostHeader**.
![create-behavior2](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/create-behavior2.png)
- Scroll to the bottom of the page and click **Create behavior**.
![create-behavior3](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/create-behavior3.png)

### 3. Create Invalidation to Purge Old Cache on CloudFront
- Switch to the **Invalidations** tab and click **Create invalidation**.
![tab-invalidation](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/tab-invalidation.png)
- Enter `/*` into the Object paths box and click **Create invalidation**.
![create-invalidation](/images/5-Workshop/5.4-Backend-deployment/5.4.6-implementing-frontend-API-gateway-integration/create-invalidation.png)
- Wait for CloudFront deployment to finish (approx. 1-3 minutes).
