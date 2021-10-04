---
title: El Monitorro - Feed reader in Rust
date: 2020-07-31
summary: Feed reader as a Telegram bot
categories: rust

redirect_from:
   - /rust/2020/07/31/el-monitorro/
---

![el-monitorro](/images/2020-07-31-el-monitorro.png)

I've been using rss/atom feeds for a long time. I remember I was subscribing to my favourite blogs on LiveJournal (It's so old you may not know about it) and news feeds in the first decade of this century (the 21st century if you're reading this post in the future). I think at that period it was a peak of popularity for this technology.

More than a decade later in 2019, it's a peak of popularity for another kind of service - messengers. I'm no different from an average contemporary so I'm an avid user of messengers and my messenger is Telegram. Nowadays I don't use traditional social networks like Facebook or VK (Russian Facebook copycat) and do all my messaging in Telegram.

I started finding myself leaving Telegram only to check news sites. So I decided why not to bring news to Telegram. Telegram provides tools for writing bots. From telegram bots intro: "Bots are third-party applications that run inside Telegram. Users can interact with bots by sending them messages". Most news sites provide JSON, ATOM or RSS feeds. I checked existing solutions and there are already a couple of Feed readers for Telegram out there. But I wasn't satisfied with any of them, none of them provided instant updates of feeds.  So my idea was to create a feed reader for Telegram bot that provided instant updates. I called it "El Monitorro".

A side note about another feed reader bot called 'the feed reader bot'. Its feed refresh rate before I released my bot was 5 minutes and it was Premium feature. After I released my bot, which has a refresh rate of 1 minute and it's free of charge, 'the feed reader bot' added new Elite level with a refresh rate of 30 seconds. What do you think about squeezing it out of business by setting an update rate to 20 seconds? LOL :)

In this post, I'll give a brief explanation about the structure of the El Monitorro bot.

### Structure

The bot consists of four parts:

- Command frontend. It's used to communicate with the bot via commands.
- Sync process. It periodically syncs data feeds.
- Delivery process. It periodically delivers unread feed items to users.
- Cleaner process. It cleans stale data from the DB.

All these processes are connected to the same Postgres DB with tables:
- feeds - RSS, ATOM or JSON feeds
- feed_items - feed items
- telegram_chats - users of the bot
- telegram_subscriptions - it connects feeds to telegram_chats

#### Command frontend

It provides an interface for users to interact with the bot using commands:

- `/start` - show the bot's description and contact information
- `/subscribe` - subscribe to feed
- `/unsubscribe` - unsubscribe from feed
- `/list_subscriptions` - list your subscriptions
- `/help` - show available commands
- `/set_timezone` - set the timezone
- `/get_timezone` - get the timezone

#### Sync process

Every minute it synchronizes every feed in the DB that has at least one subscription. It saves fetched items to `feed_items` table. Some feeds get unaccessible after period of time, they are removed if they unaccessible for 24 hours.

Initially, I set the number of subscriptions limit to 5, but after some time I received messages from users asking to increase it. So now it's 20.

#### Delivery process

Every minute it delivers feed items that weren't delivered to the user yet.

It decides which items user did not read based on `publication_date` field in the `feed_items` table. `telegram_subscriptions` table has `last_delivered_at` field, the delivery process sends all feed items that have `publication_date` > `last_delivered_at` and then updates `last_delivered_at` to the latest `publication_date`. There is one issue that appears in that approach. If a feed provider uses `publication_date` not for feed item publication date but another purpose, for example, for the product release date, user may not receive all messages. I encountered this issue in Metacritic feeds, they use `publication_date` for the product release date.

#### Cleaner process

I created the cleaning process a couple of months later after I released the bot. I discovered even for the relatively small amount of users (~ 100), `feed_items` table contains around ~100_000 records. So for 1000 users, it'll have around one million records (I think it will be smaller, because users will have overlapping subscriptions, but you got the idea).

Every 12 hours the cleaner process removes all feeds without subscriptions and for remaining feeds, it only leaves the last 30 feed items. After the first run of this process, the number of records in `feed_items` changed from 100_000 to ~ 4_000.

### Planned features

Over the last two months users kept asking for two relatively large features for El Monitorro:

- ability to use it in Telegram channels
- ability to customize messages

I'm planning to implement them in the upcoming months.

### Conclusion

The bot was released at the end of May 2020.

It's available at [https://t.me/el_monitorro_bot](https://t.me/el_monitorro_bot). Its code available on [GitHub](https://github.com/ayrat555/el_monitorro).
