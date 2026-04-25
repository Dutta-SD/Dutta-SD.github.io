---
title: "Mini-SQS — Durable Job Queue"
date: 2026-04-01
summary: "A Postgres-backed, at-least-once task queue in Go with visibility timeouts, retries, and a dead-letter queue. In progress."
tags: ["go", "postgres", "distributed-systems", "wip"]
weight: 2
---

A from-scratch implementation of the core of AWS SQS — the part you never see when you just call `sqs.receiveMessage()`. Built to understand how durable at-least-once delivery actually works when workers can crash mid-job.

### Goals
- **Durable enqueue:** job is persisted to Postgres before the HTTP response returns.
- **Exactly-one-claim-at-a-time:** no two workers ever run the same job, proven via `SELECT ... FOR UPDATE SKIP LOCKED`.
- **Crash recovery:** if a worker dies mid-job, a lease-based visibility timeout lets another worker re-run it.
- **Retries with exponential backoff** and a **dead-letter queue** after max attempts.
- **Target load:** sustained 5,000 jobs/sec enqueue + dequeue on a laptop.

### Stack
Go · PostgreSQL 15 · Docker Compose · Prometheus

### Status
Designed. Implementation in progress.
