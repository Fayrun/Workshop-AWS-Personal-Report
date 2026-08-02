---
title: "LAMBDA TENANT ISOLATION MODE: DOES THE NEW FEATURE ACTUALLY FIX MULTI-TENANT DATA LEAK BUGS?"
date: 2026-01-15
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

<!-- # LAMBDA TENANT ISOLATION MODE: DOES THE NEW FEATURE ACTUALLY FIX MULTI-TENANT DATA LEAK BUGS? -->

## Introduction

Hi everyone, my team is building a document Q&A chatbot (RAG) that serves multiple users at once (multi-tenant): each user signs in through Cognito, uploads their own documents, and data must be strictly isolated between users. The backend runs on a single Lambda function (Docker container) that serves all users.

This is the most serious bug I've ever run into on this project: User A uploads a document, User B logs in and can see User A's document too — and worse, when User B deletes all of their own documents, User A's data gets wiped along with it. After investigating and fixing this bug by hand, I found out AWS had just announced (November 2025) a new feature that targets exactly this problem: **Lambda Tenant Isolation Mode**. This post walks through the real debugging process, explains the new feature, and — most importantly — analyzes why this feature does **NOT** automatically fix the bug I ran into, even though the name suggests it would.

## The Real Bug: Why Was Data Mixed Between Users?

Lambda reuses the execution environment (container) across multiple invocations to speed things up (avoiding cold starts) — but by default, that environment gets reused for _any_ request to the same function, regardless of which user the request came from. In my backend code, there were two places that unintentionally relied on "the container staying the same between requests":

- A global dict (`state`) storing `raw_documents`, `vector_store`, etc., shared across every request on the same container.
- A FAISS vector index using a fixed S3 path, `vectorstore/smartdoc_index` — with no `user_id` in the path, so every user read/wrote the same index file.

When two different users happened to get routed to the same warm container, they ended up sharing both the in-RAM data and the S3 data.

## What Does Lambda Tenant Isolation Mode Solve?

This new feature lets Lambda process invocations in separate execution environments per end-user/tenant, instead of sharing one environment across every request to the same function.

How it works: you pass an additional `tenant-id` parameter (via the `X-Amz-Tenant-Id` header when integrated with API Gateway) on each invocation. Lambda uses this ID to ensure the environment is only reused for the same tenant — you still get the benefit of warm starts, but without the risk of data in RAM/`/tmp` leaking across tenants.

## How I Fixed the Bug (Before Knowing About This Feature)

- `get_user_index_name(user_id)` — computes a per-user FAISS path: `f"{user_id}/{FAISS_INDEX_NAME}"`
- Every document/chat endpoint now reads fresh data directly from S3/FAISS on each request, no longer relying on the global `state` variable
- `/api/upload-url`: changed the S3 key to `uploads/{user_id}/{filename}` instead of `uploads/{filename}`

## Why Doesn't Tenant Isolation Mode Automatically Fix This Bug?

This is the part I want to emphasize, since it's easy to get asked "why not just wait for/use the built-in feature instead":

Tenant Isolation Mode only isolates at the **compute layer** — the execution environment, RAM, `/tmp`. It does not, and cannot, control how the application names its keys in S3 or DynamoDB. If I had only enabled Tenant Isolation Mode without fixing the shared S3 path (`vectorstore/smartdoc_index`), here's what would happen:

- The `state` variable in RAM would be correctly isolated — one user's cache would no longer leak into another user's, within the same run.
- But when Lambda (even running in a tenant-specific environment) reads the FAISS index back from S3 at the fixed path `vectorstore/smartdoc_index`, it would still read that exact same shared file — the persistent-data bug would still be there, because the root cause lives at the **data modeling layer (application logic)**, not the compute isolation layer.

In other words: Tenant Isolation Mode is a great additional layer of defense against similar issues in the future (e.g., a temp cache in `/tmp`, a global variable accidentally shared) — but it does **not replace** the need to design per-tenant data paths correctly at the application layer in the first place. A proper fix still has to happen where the bug actually lives.

## Verification Results

After the manual fix (before Tenant Isolation Mode was even released), I retested with two real Cognito users:

- Each user only sees their own files via `/api/files`
- Asking the AI about a "secret code" from the other user's document → the AI replies "not found in the document"
- User B deletes all their files → User A's data remains completely intact

## Conclusion

Multi-tenancy bugs are among the hardest classes of bugs to catch in serverless, because they only show up when two requests happen to land on the same warm container. Lambda Tenant Isolation Mode is a big step forward at the compute layer, but real data safety always starts at the **data modeling layer** — every per-tenant resource (S3 key, DynamoDB partition key, cache key, etc.) needs to explicitly include a tenant/user identifier, no matter how good the compute-layer isolation underneath it is.

## Article Link

Official AWS blog post on Lambda Tenant Isolation Mode:
https://aws.amazon.com/blogs/compute/building-multi-tenant-saas-applications-with-aws-lambdas-new-tenant-isolation-mode/
