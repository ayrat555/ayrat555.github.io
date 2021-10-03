---
title: You don't need a background job framework in Rust
date: 2020-06-28
summary: My experience trying to find a suitable library for background job processing
categories: rust
---

![img](/images/2020-06-28-iceberg.jpg)

From the beginning of this year, I've been writing feed reader as a Telegram bot in Rust. Its functionality is simple: it allows a user to subscribe to RSS, Atom or JSON feeds and sends new feed items.

I released it last month and I'm happy to report that it acquired some users and it functions perfectly. It's deployed at [https://t.me/el_monitorro_bot](https://t.me/el_monitorro_bot). Its code is available on Github - [el_monitorro](https://github.com/ayrat555/el_monitorro). Hopefully, I'll write a post about in the near future.

But in this post, I'll describe how the bot handles the processing of background jobs.

### Background processing

#### The problem

Two main jobs that the bot have to perform:
- Sync feeds
- Deliver new feed items to users

Both of these actions should be performed periodically. I assumed that there is an established background processing framework in the Rust programming language that I can use. But it appears, there isn't. I spent a couple of weeks trying different not so popular background processing libraries which were buggy or not feature complete.

#### The solution

The common feature of the majority of these libraries is that they use Tokio tasks for concurrent processing. I started looking into Tokio and its Tokio tasks. Tokio documentation claims that tokio tasks are similar to Erlang processes. As a backend developer who spent the last couple of years writing Elixir fulltime, I came to realisation that this is exactly what I need.

I already had a Postgres database set up. The only thing left to do is periodically loop through feeds in the DB and concurrently sync them using Tokio tasks. I created a separate binary that does exactly that.

This is the entry point of the sync binary:

```rust
use dotenv::dotenv;
use el_monitorro;
use el_monitorro::sync::sync_job;

#[tokio::main]
async fn main() {
    dotenv().ok();
    env_logger::init();

    sync_job::sync_feeds().await;
}
```

It just calls `sync_jobs::sync_feeds()`. `sync_feeds` triggers syncing of all feeds every minute:

```rust
pub async fn sync_feeds() {
    let mut interval = time::interval(std::time::Duration::from_secs(60));
    loop {
        interval.tick().await;
        sync_all_feeds();
    }
}
```

The looping function looks like this:

```
   for id in &unsynced_feed_ids {
      tokio::spawn(sync_feed(*id));
   }
```

As you can see it just spawns a new Tokio task which processes the specified feed. In case of an error during the sync, it's written to logs and to the special column called `error` in the feeds table. The same idea is used for delivering messaged to users.

### Conclusion

I hope this post will be useful for people new to Rust looking into background job processing.
