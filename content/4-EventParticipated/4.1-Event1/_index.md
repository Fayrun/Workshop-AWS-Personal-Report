---
title: "Event 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Cloud Architect x Meet Up

### Purpose of the Event

- Create an environment where cloud engineers and AWS learners could exchange practical experiences.
- Introduce AWS Security Agent and discuss the growing importance of cloud security in modern software development.
- Share practical strategies for preparing AWS certification exams.
- Strengthen networking within the AWS Study Group and the cloud community.

### Speakers

The meetup featured four speakers. The following two sessions were the ones that left the strongest impression on me.

- **Mr. Long** – Security Solutions Architect, introduced **AWS Security Agent** and shared practical security considerations when developing cloud applications.
- **Mr. Huy** – AWS Certified Solutions Architect, shared his AWS certification journey together with useful exam preparation strategies.

### Key Takeaways

#### AWS Security Agent Session

- AWS Security Agent is capable of scanning an entire project to detect security risks beyond simple account misconfigurations. It can identify hardcoded credentials, vulnerable dependencies, overly permissive security configurations, missing encryption, and common application-level vulnerabilities.
- Instead of producing raw logs, the service provides an intuitive dashboard together with an automatically generated PDF report, making it easier for development teams to review and address security issues.
- One of the most valuable lessons was the importance of maintaining secure coding practices even when AI-assisted development is becoming increasingly common.

#### AWS Certification Experience Sharing

- AWS certification exams focus heavily on scenario-based questions that evaluate the ability to choose the most appropriate AWS services rather than programming skills.
- Practical advice such as time management, flagging difficult questions for later review, and paying attention to multiple-response questions can significantly improve exam performance.
- The speaker also emphasized the importance of understanding service use cases instead of simply memorizing AWS documentation.

### Knowledge Gained

- Security should be considered throughout the entire software development lifecycle rather than only before deployment.
- Automated security assessment tools can effectively complement manual code reviews by detecting issues that developers may overlook.
- Preparing for AWS certifications requires both technical knowledge and effective exam strategies based on real testing experiences.

### Application to My Project

- Applied stronger security practices by enabling KMS encryption for DynamoDB.
- Added automated pytest execution during the pre-build stage of the CI/CD pipeline to improve software quality before deployment.
- Started paying more attention to integrating security validation into the development workflow.

### Personal Reflection

This meetup gave me a better understanding of how security should be integrated into cloud applications instead of being treated as a separate task after development. The practical experiences shared by the speakers helped me recognize several areas where my own project could be improved, especially regarding security automation and deployment best practices. The event also motivated me to continue studying AWS services and prepare for AWS certification in the future.

#### Event Photos

![Event 1](/images/4-EventParticipated/4.1-Event1/Event-11-07-2026.jpeg)
