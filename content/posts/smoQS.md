---
title: "smoQS: A Distributed Queue + Job Scheduler"
date: 2026-05-28
draft: false
math: true
tags: ["queue", "project"]
summary: "smoQS: A Distributed Queue + Job Scheduler"
---

I finally started on this journey of creating a Distributed Scheduler. This blog post will be kindof like a changelog.

## Language: Java

Going with **Java**. The honest reason is that I already wanted to learn Java more deeply — concurrency utilities in `java.util.concurrent`, the memory model, GraalVM, all of it. A queue/scheduler is the right kind of project for that: it's nothing *but* concurrency primitives and lifecycle management. So the language choice and the learning goal lined up.

**Tradeoffs:**

- *Win:* deep concurrency toolbox. `BlockingQueue`, `DelayQueue`, `ReentrantLock`, `CompletableFuture` — all battle-tested.
- *Loss:* verbose vs. Go or Kotlin. Build tooling (Gradle) is heavy.
- *Rejected alternatives:* Go or Rust would have meant learning the language *and* building the project at the same time — added burden I didn't want. Java I already know reasonably well, so I can focus on the design.

## Transport: REST

The broker exposes a **REST API**. Clients submit jobs; workers reserve, ack, and extend leases. Plain request/response.

**Tradeoffs:**

- *Win:* trivial to test (`curl`), every language has a client, no protocol-specific tooling needed.
- *Loss:* no streaming. A worker that wants near-zero dispatch latency will have to long-poll, which is workable but uglier than gRPC streaming or a raw socket.
- *Deferred:* could add a streaming endpoint later if the polling overhead actually shows up. Not worth it on day one.

## Web framework: Quarkus

Spring Boot was the obvious starting point but felt too heavy for a project this size. I do want opinions and structure — I just didn't want Spring's specific weight. That left a few real options:

| Framework  | Startup    | DI               | GraalVM native | Opinionated | Notes                                    |
| ---------- | ---------- | ---------------- | -------------- | ----------- | ---------------------------------------- |
| Spring Boot| ~3–5s      | Yes (runtime)    | Painful        | Heavily     | Too much weight                          |
| Javalin    | sub-second | None             | Easy           | No          | Would need Dagger bolted on for DI       |
| Micronaut  | ~1s        | Yes (compile)    | Good           | Yes         | Solid, smaller community                 |
| Quarkus    | ~1s        | Yes (CDI/ArC)    | First-class    | Yes         | Designed around native-image from day 1  |

Going with **Quarkus**. CDI is built in (compile-time, native-image-friendly), the dev loop has live reload, and the GraalVM tooling was first-class from the start — the framework was *designed* around native-image, not retrofitted to it. That kills the separate "DI library" decision: I don't need Dagger, because Quarkus's own ArC fills the same role.

I briefly considered Javalin + Dagger as a "lighter" stack, but it's basically reinventing what Quarkus already bundles, with worse defaults.

**Tradeoffs:**

- *Win:* opinionated stack with batteries included — DI, config, REST (RESTEasy Reactive), test annotations, native-image build, all integrated.
- *Loss:* more abstraction to learn upfront vs. a thin router; if I fight a Quarkus convention later that's a real cost.
- *Why not Micronaut:* Quarkus has more momentum, better docs, and Red Hat's investment in the GraalVM path. Close call, but the tooling tipped it.

## Runtime: OpenJDK 17 now, GraalVM later

Sticking with **OpenJDK 17** for the build-out. Standard tooling, fast iteration, easy to debug.

**GraalVM native image** is the planned migration once the broker stabilises — for the startup-time and memory-footprint wins. Quarkus makes this swap cheap: the same code builds either as a JVM jar or as a native binary.

**Tradeoffs:**

- *Win (GraalVM):* sub-100ms startup, lower RSS, single binary distribution.
- *Loss (GraalVM):* slow native-image compile (minutes), worse debugging story (no JFR, harder profiling), occasional reflection / dynamic-proxy gotchas. Not worth fighting during active development.

## Stack so far

- **Language:** Java 17
- **Framework:** Quarkus (with CDI for DI, RESTEasy for REST)
- **Runtime:** OpenJDK 17 now, GraalVM native image later
