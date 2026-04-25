---
layout: page
title: Mini-Kafka — Log-Structured Broker (Planned)
description: A single-broker, multi-partition append-only message log with consumer groups and offset tracking.
img: assets/img/3.jpg
importance: 3
category: in progress
---

A small-but-real Kafka-like broker: append-only segment files per partition, offset indexes for O(1) reads, consumer groups with static assignment, and retention by size + age.

### Goals
- **Persistent append-only log** per partition with segment rolling at 1 GB.
- **Offset index** files mapping logical offsets to byte positions for fast random reads.
- **Consumer groups** where each partition is owned by exactly one consumer in the group; rebalance on join/leave.
- **Retention** driven by total size or age, without deleting segments that are actively being read.
- Benchmark: **100k msgs/sec** sustained on a single broker locally.

### Stack
Go · raw TCP framing · Docker Compose

### Status
Planned — Phase 1 (single broker, localhost producers/consumers) will follow the durable job queue. Phase 2 is multi-machine; Phase 3 (replication) is a stretch.
