---
title: "A Distributed Job Scheduler: smoQS"
date: 2026-05-28
draft: false
math: true
tags: ["queue", "project"]
summary: "A Distributed Job Scheduler: smoQS"
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

## Web framework: Javalin

Started sketching this on Spring Boot out of habit. Killed it within an hour — way too much weight for what is essentially a router around a queue. Quarkus and Micronaut were tempting (good startup numbers, built-in DI), but "framework with opinions" was the exact thing I was running away from.

Ended up with **Javalin**. Thin wrapper over Jetty, no annotation magic, no auto-configuration. You instantiate a `Javalin` object and register routes on it. That's the whole story.

**Tradeoffs:**

- *Win:* minimal surface area; the queue stays the protagonist.
- *Loss:* no built-in DI, no opinionated config layer, no test slices — I have to wire those myself.
- *Risk:* smaller community than Spring/Quarkus. If I hit something obscure I'm reading source code, not Stack Overflow.

## Dependency injection: Dagger

Javalin doesn't ship with DI. Fine for hello-world, but I want a container for the object graph — job store, dispatcher, lease manager, handlers all need to find each other. So: Guice or Dagger.

| DI library | Wiring        | Reflection      | GraalVM         |
| ---------- | ------------- | --------------- | --------------- |
| Guice      | Runtime       | Heavy           | Painful         |
| Dagger     | Compile-time  | None            | Native-friendly |

Picking **Dagger**.

**Tradeoffs:**

- *Win:* compile-time wiring means errors at `javac` time, not at first request. Plays well with GraalVM.
- *Loss:* `@Module` / `@Component` ceremony, slower edit-compile loop because of annotation processing.
- *Why not Guice:* I plan to go GraalVM native image later, and Guice's runtime reflection means hand-writing reflection-config JSON for every injected class. I've done that before and didn't enjoy it.

## Runtime: OpenJDK 17 now, GraalVM later

Sticking with **OpenJDK 17** for the build-out. Standard tooling, fast iteration, easy to debug.

**GraalVM native image** is the planned migration once the broker stabilises — for the startup-time and memory-footprint wins, especially if I ever want to run multiple broker instances on a small box.

**Tradeoffs:**

- *Win (GraalVM):* sub-100ms startup, lower RSS, single binary distribution.
- *Loss (GraalVM):* slow native-image compile (minutes), worse debugging story (no JFR, harder profiling), reflection / dynamic-proxy gotchas. Not worth fighting during active development.
- *Why 17 over 21:* I want to stay on an LTS that's broadly supported in libraries today; 21 is fine but I don't need virtual threads here, since the broker's concurrency model is queue-driven, not thread-per-request.

## Stack so far

- **Language:** Java 17
- **Web framework:** Javalin
- **DI:** Dagger
- **Runtime:** OpenJDK 17 now, GraalVM native image later
