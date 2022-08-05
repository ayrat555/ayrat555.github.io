---
title: Fang: Async background processing for rust
date: 2022-05-07
summary: Async background processing for rust with tokio and postgres
categories: rust
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /images/2022-08-08-factory.png
---

## Introduction

Even though the first stable version of Rust was released in 2015, there are still some holes in its ecosystem for solving common tasks. One of which is background processing.

In software engineering background processing is a common approach for solving several problems:

- Carry out periodic tasks. For example, deliver notifications, update cached values.
- Defer expensive work so your application stays responsive while performing calculations in the background

Most programming languages have go-to background processing frameworks/libraries. For example:

- Ruby - [sidekiq](https://github.com/mperham/sidekiq). It uses Redis as a job queue.
- Python - [dramatiq](https://github.com/Bogdanp/dramatiq). It uses RabbitMQ as a job queue.
- Elixir - [oban](https://github.com/sorentwo/oban). It uses a Postgres DB as a job queue.

The async programming (async/await) can be used for background processing but it has several major disadvantages if used directly:

- It doesn't give control to the number of tasks that are being executed at any given time. So a lot of spawned tasks can overload a thread/threads that they're started on.
- It doesn't provide any monitoring which can be useful to investigate your system and find bottlenecks
- Tasks are not persistent. So all enqueued tasks are lost on every application restart

To solve these shortcomings of the async programming we implemented the async processing in [the fang library](https://github.com/ayrat555/fang).

## Threaded Fang

Fang is a background processing library for rust. The first version of Fang was released exactly one year ago. Its key features were:

- Each worker is started in a separate thread
- A Postgres table is used as the task queue

This implementation was written for a specific use case - [el monitorro bot](https://github.com/ayrat555/el_monitorro). This specific implementation of background processing was provided by time. Each day it processes more and more feeds every minute (the current number is more than 3000). Some users host a bot on their infrastructure.

You can find out more about the threaded processing in fang in [this blog post](https://www.badykov.com/rust/fang/).

## Async Fang

<blockquote>
  <p>
Async provides significantly reduced CPU and memory overhead, especially for workloads with a large amount of IO-bound tasks, such as servers and databases. All else equal, you can have orders of magnitude more tasks than OS threads, because an async runtime uses a small amount of (expensive) threads to handle a large amount of (cheap) tasks
  </p>
  <footer><cite title="Async book">From the Rust's Async book</cite></footer>
</blockquote>

For some lightweight background tasks, it's cheaper to run them on the same thread using async instead of starting one thread per worker.


The threaded processing uses the [diesel](https://github.com/diesel-rs/diesel) ORM which blocks the thread


## Usage

```rust
use fang::serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
#[serde(crate = "fang::serde")]
pub struct MyTask {
    pub number: u16,
}

impl MyTask {
    pub fn new(number: u16) -> Self {
        Self { number }
    }
}
```

```rust
use fang::async_trait;
use fang::typetag;
use fang::AsyncRunnable;
use std::time::Duration;

#[async_trait]
#[typetag::serde]
impl AsyncRunnable for MyTask {
    async fn run(&self, queue: &mut dyn AsyncQueueable) -> Result<(), Error> {
        let new_task = MyTask::new(self.number + 1);
        queue
            .insert_task(&new_task as &dyn AsyncRunnable)
            .await
            .unwrap();

        log::info!("the current number is {}", self.number);
        tokio::time::sleep(Duration::from_secs(3)).await;

        Ok(())
    }
}

```

```rust
use fang::asynk::async_queue::AsyncQueue;

let max_pool_size: u32 = 2;
let mut queue = AsyncQueue::builder()
    .uri("postgres://postgres:postgres@localhost/fang")
    .max_pool_size(max_pool_size)
    .duplicated_tasks(true)
    .build();
```

```rust
use fang::asynk::async_worker_pool::AsyncWorkerPool;
use fang::NoTls;

let mut pool: AsyncWorkerPool<AsyncQueue<NoTls>> = AsyncWorkerPool::builder()
    .number_of_workers(10_u32)
    .queue(queue.clone())
    .build();

pool.start().await;
```

```rust
let task = MyTask::new(0);

queue
    .insert_task(&task1 as &dyn AsyncRunnable)
    .await
    .unwrap();
```
