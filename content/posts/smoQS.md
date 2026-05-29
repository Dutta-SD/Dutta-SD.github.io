---
title: "A Distributed Job Scheduler: smoQS"
date: 2026-05-28
draft: false
math: true
tags: ["queue", "project"]
summary: "A Distributed Job Scheduler: smoQS"
---

I finally started on this journey of creating a Distributed Scheduler. This blog post will be kindof like a changelog.

After evaluating the options, we went ahead with **Java** as the implementation language — a natural fit given the ecosystem around concurrent programming, the JVM's mature threading model, and the availability of battle-tested libraries for networking and persistence.
