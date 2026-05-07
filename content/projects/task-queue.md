---
title: "Priority Task Queue"
description: "Redis-backed job queue with priority scheduling and dead letter handling"
date: 2026-01-08
draft: false
tags: ["python", "redis", "task-queue", "async"]
image: "/images/projects/task-queue.png"
links:
  - name: "GitHub"
    url: "https://github.com/keshavbansal015/task-queue"
  - name: "PyPI"
    url: "https://pypi.org/project/priority-task-queue"
---

A robust task queue system supporting priority levels, scheduled execution, and automatic retries with exponential backoff.

## Features

- **Priority Queues**: 10 priority levels with weighted consumption
- **Scheduled Tasks**: Execute jobs at specific times or with delays
- **Retries**: Configurable retry policies with exponential backoff
- **Dead Letter Queue**: Failed jobs moved to DLQ after max retries
- **Monitoring**: Web dashboard for queue metrics and job inspection

## Quick Start

```python
from task_queue import Queue, Task

queue = Queue(redis_url="redis://localhost:6379")

@queue.register
def send_email(to: str, subject: str, body: str):
    # Send email logic
    pass

# Submit high-priority task
queue.submit(
    Task(send_email, "user@example.com", "Welcome", "..."),
    priority=9
)
```

## Architecture

Uses Redis sorted sets for priority ordering and lists for FIFO semantics within priority levels. Workers use optimistic locking for safe job claiming.

## Production Usage

Processing 500K+ tasks daily across 50+ worker processes with 99.9% success rate.
