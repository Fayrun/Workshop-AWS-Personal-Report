---
title: "Baseline Architecture for Amazon Bedrock in an AWS Landing Zone Environment"
date: 2026-01-25
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

> **Written by: Phong** (a member of our team). This post is reproduced in this personal report as a shared reference for the whole team.

## 1. System Purpose and the Deployment Problem

When adopting Generative AI (GenAI) at enterprise scale, organizations often run into major challenges around risk governance, security, and compliance. If development teams are allowed to freely deploy Amazon Bedrock within a single, loosely controlled AWS account, the business becomes exposed to data leaks, network policy violations, and unmanageable costs.

The problem, then, is to build a solid baseline architecture — infrastructure that lets engineering teams innovate and experiment with AI quickly, while still staying within a secure, centrally monitored boundary with strict data isolation.

## 2. The Solution: Multi-Account Governance With AWS Landing Zone

The core solution is to avoid running GenAI workloads inside a single monolithic account, and instead distribute them across a multi-account environment using AWS Control Tower (Landing Zone).

The system clearly separates accounts by purpose (e.g., a Sandbox for experimentation, Dev/Test, and Production).

This approach limits the "blast radius." If a test environment is misconfigured, accounts holding production data or core business services remain completely safe and unaffected.

## 3. Isolated Network Architecture and Fine-Grained Access Management

This baseline architecture establishes strict protection layers at both the network and identity level:

**Layer 1: Private Networking**

Instead of calling the Amazon Bedrock API over the public internet, the architecture uses AWS PrivateLink (VPC Endpoints).

All data flow (from the application to the LLM) travels over AWS's internal backbone network. Combined with AWS Transit Gateway in a Hub-and-Spoke model, the business gains full control over the data's path, completely preventing the risk of data exfiltration over the internet.

**Layer 2: Identity & Access Management**

AWS IAM Identity Center is used to grant developers temporary access.

The system applies role-based (RBAC) and attribute-based (ABAC) access control, enforcing the principle of least privilege. An engineer on Project A will only be able to see and use the models/resources specifically authorized for that project.

## 4. Centralized Operations and Monitoring

To maintain compliance without slowing down development, the system relies on automated governance mechanisms:

- **Automated Guardrails (SCPs):** At the AWS Organizations level, Service Control Policies (SCPs) are applied. For example, an SCP can prohibit model exfiltration by users, or restrict calls to only approved Foundation Models (FMs).
- **Centralized Logging:** The complete history of API calls — including both the prompt input and the model's response (via Model Invocation Logging) — is encrypted and sent directly to an isolated Log Archive account. The security team can monitor this in real time using Amazon CloudWatch and CloudTrail, without ever needing access to a developer's account.

## 5. Bedrock's Deep, Built-In Data Safety Features

Beyond the infrastructure-level security provided by the Landing Zone, this architecture also inherits the strong default security features of Amazon Bedrock itself:

- **Absolute Privacy:** Enterprise prompt data, responses, and fine-tuning data are stored locally, encrypted with AWS KMS, and are never used to retrain Amazon's or any third party's foundation models.
- **Bedrock Guardrails Integration:** Infrastructure-level security is combined with Bedrock's built-in Guardrails to automatically block prompts requesting personally identifiable information (PII), protecting the integrity of sensitive customer data right at the application layer.

## Reference

https://aws.amazon.com/vi/blogs/architecture/amazon-bedrock-baseline-architecture-in-an-aws-landing-zone/

## Link to the post in the AWS Study Group

https://www.facebook.com/groups/awsstudygroupfcj/permalink/2207763766655250
