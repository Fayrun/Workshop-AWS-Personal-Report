---
title: "EventBridge Scheduler: When Should You 'Upgrade' From EventBridge Rule?"
date: 2026-07-28
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

<!-- # EVENTBRIDGE SCHEDULER: WHEN SHOULD YOU "UPGRADE" FROM EVENTBRIDGE RULE? -->

## Introduction

Hi everyone, my team is building a document Q&A chatbot (RAG): users upload PDF/DOCX files and then chat directly with an AI based on the content of those documents. The whole system runs serverless on AWS — API Gateway + Lambda (packaged as Docker) for the backend, Cognito for authentication, DynamoDB + S3 for storage, and Amazon Bedrock for the LLM/Embeddings layer.

While building this out, I needed a mechanism to automatically clean up Cognito accounts that signed up but never confirmed their email (UNCONFIRMED) — if left alone, they just pile up as "junk" in the User Pool. The solution I picked was an **EventBridge Rule** running on `rate(1 minute)` to trigger a Lambda that checks for and deletes expired users.

While digging deeper to write this post, I discovered AWS actually has a dedicated service just for scheduling — **Amazon EventBridge Scheduler** — which is far more powerful than a traditional EventBridge Rule. This post walks through the differences between the two options, and more importantly: why I still chose Rule for my current use case, rather than jumping straight to Scheduler just because it's "newer."

## What Is EventBridge Scheduler?

Amazon EventBridge Scheduler is a serverless service for creating, running, and managing scheduled tasks at scale — one-time or recurring — across more than 270 AWS services and over 6,000 API actions, without having to manage any infrastructure yourself.

Previously, if you wanted to schedule a task, the most common option was an EventBridge Rule with a `cron()` or `rate()` expression. That works fine, but it comes with a few limitations:

- A maximum of **300 rules per region per account** — not great if you need a separate schedule for thousands of customers (e.g., every tenant in a SaaS product needing its own reminder schedule).
- Only supports around **20 target types**.
- No built-in retry with exponential backoff, dead-letter queue (DLQ), or time window to spread out request load.
- No support for one-time schedules — recurring only.

EventBridge Scheduler solves all of the above: it supports up to **1 million schedules per account** (instead of 300 rules per region), throughput up to thousands of TPS, connects to **270+ AWS services and 6,000+ API actions** (instead of Rule's ~20 targets), supports one-time schedules (`at()`) alongside recurring ones, includes a time window to spread out request load, comes with built-in retry + dead-letter queue (retries up to 185 times over 24 hours by default), and fully supports time zones/DST instead of just UTC like Rule.

## The Actual Architecture in My Project

The backend Lambda is designed as an event-branching function: the same function handles both HTTP requests (via API Gateway/Mangum) and scheduled events from EventBridge, based on `event.get("source") == "aws.events"`. This is a minimal-footprint choice — no need for a separate Lambda just to run a cron job.

## Why Did I Choose EventBridge Rule Instead of Scheduler?

I think this is the most important part, since it's an easy question to get pushback on — "why not just use the newer, more 'correct' option?":

- **There's exactly one type of recurring task** in the whole system — cleaning up unconfirmed users. There's no need for per-user/per-tenant scheduling, so I never come close to the 300-rule limit or need Scheduler's 1-million-schedule capacity.
- **No need for complex retry/DLQ** — if one cleanup run fails, the next run (1 minute later) automatically runs again and still cleans up any expired users correctly, so no dedicated retry mechanism is needed.
- **No need for one-time schedules** — this is a task that recurs forever via `rate()`; Scheduler's headline features (one-time schedules, time windows) don't provide any benefit for this use case.
- **Cost & operational complexity:** Scheduler is still free under the Free Tier and not meaningfully more expensive, but adding a new service to the architecture means one more thing to learn and maintain — at the current demo/internship scale, that trade-off isn't worth it.

In other words: Scheduler isn't "better" than Rule in some absolute sense — it's better for the specific problem it was built to solve (large-scale scheduling, diverse targets, sophisticated retry/DLQ needs). If this project later grows into a real SaaS product with thousands of tenants, each needing their own schedule (e.g., trial-expiration reminders, auto-cleanup based on each customer's own retention policy) — that's exactly the point where it would make sense to switch to Scheduler.

## Conclusion

EventBridge Scheduler is a genuinely powerful upgrade over EventBridge Rule, especially for multi-tenant SaaS systems that need scheduling at scale. But choosing a technology should be based on the actual problem at hand, not on which service is "newer." For a simple recurring task like cleaning up unconfirmed users, EventBridge Rule remains the minimal, sensible choice.

## Article Link

Official AWS blog post on Amazon EventBridge Scheduler:
https://aws.amazon.com/blogs/compute/introducing-amazon-eventbridge-scheduler/

## Link to the post in the AWS Study Group

https://www.facebook.com/groups/awsstudygroupfcj/permalink/2226683848096575/
