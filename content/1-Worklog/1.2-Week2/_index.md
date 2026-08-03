---
title: "Week 2 Worklog"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

- Expand into more advanced Storage/Networking/Database services: S3, CloudFront, RDS.
- Learn EC2 Auto Scaling and data encryption with KMS.
- Start learning Amazon Cognito — the service that will become the authentication backbone of Smart Docs AI.

### Tasks to be carried out this week:

| Day | Task                                                                                                                                                                                                                                                                                                                                                                                                         | Start Date | Completion Date | Reference Material                                  |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | --------------------------------------------------- |
| 2   | - Learned Amazon S3: buckets, objects, storage classes (Standard/IA/Glacier), versioning<br>&emsp;+ Understood the differences between storage classes to pick the right one for each type of data (frequently accessed vs. long-term archival) - **Practice:** created a bucket, uploaded/downloaded objects, tested enabling versioning                                                                    | 06/29/2026 | 06/29/2026      | [AWS S3](https://000057.awsstudygroup.com/vi/)      |
| 3   | - Learned CloudFront: CDN, Origin, Distribution, cache behavior<br>&emsp;+ Understood how Edge Location caching reduces origin load and speeds up page loads for users far from the origin - **Practice:** created a CloudFront Distribution pointing to a static S3 bucket, compared load times before/after adding the CDN                                                                                 | 06/30/2026 | 06/30/2026      |                                                     |
| 4   | - Learned Amazon RDS: Multi-AZ, Read Replicas, automated backup/restore<br>&emsp;+ Compared RDS (managed, less operational overhead) against self-hosting a database on EC2 - Learned EC2 Auto Scaling: Launch Templates, Auto Scaling Groups, scaling policies <br>&emsp;+ Understood the scale-out/scale-in mechanism based on CPU/traffic metrics                                                         | 07/01/2026 | 07/01/2026      | [Amazon RDS](https://000005.awsstudygroup.com/vi/)  |
| 5   | - Learned AWS KMS: Customer Managed Key vs. AWS Managed Key, how at-rest data encryption works<br>&emsp;+ Took notes to apply later to Smart Docs AI's DynamoDB table - Started learning Amazon Cognito: User Pool, Identity Pool, User Pool Client <br>&emsp;+ Distinguished between User Pool (identity management, sign-up/sign-in) and Identity Pool (issuing temporary AWS resource access credentials) | 07/02/2026 | 07/02/2026      | [AWS Cognito](https://000081.awsstudygroup.com/vi/) |
| 6   | -**Practice:** created a test Cognito User Pool, tried a basic sign-up/OTP confirmation flow through the Console <br>&emsp;+ Tested this to get a feel for the UserStatus flow (UNCONFIRMED → CONFIRMED) — important groundwork for coding the real sign-up flow the following week                                                                                                                          | 07/03/2026 | 07/03/2026      |                                                     |
| 7   | - Reviewed everything learned over the first two weeks, consolidated it into personal notes <br> - Checked in with the team on frontend progress, clarified the upcoming backend work split (who owns auth, who owns RAG/document processing)                                                                                                                                                                | 07/04/2026 | 07/04/2026      |                                                     |

### Week 2 Achievements:

- Grasped the core concepts of S3 (storage classes, versioning), CloudFront (CDN, caching), and RDS (Multi-AZ, Read Replicas).
- Understood the EC2 Auto Scaling mechanism and how to use KMS for at-rest encryption.
- Built a foundational understanding of Cognito (User Pool/Identity Pool) and tested a basic sign-up/OTP flow through the Console.
- Agreed with the team on the backend work split for the coming weeks.
