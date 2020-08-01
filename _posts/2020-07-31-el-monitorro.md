---
layout: post
title: El Monitorro - Feed reader in Rust
date: 2020-07-31
summary: Feed reader as a Telegram bot
categories: rust
---

I've been using rss/atom feeds for a long time. I remember I was subscribing to my favourite blogs on LiveJournal (It's so old you may not know about it) and news feeds in the first decade of this century (the 21st century if you're reading this post in the future). I think at that period it was a peak of popularity for this technology.

More than a decade later in the 2019, it's a peak of popularity for another kind of service - messengers. I'm no different from an average contemporary so I'm an avid user of messengers and my messenger is Telegram. Nowadays I don't use traditional social networks like Facebook or VK (Russian facebook copycat) and do all my messaging in Telegram.

I started finding myself leaving Telegram only to check news sites. So I decided why not to bring news to Telegram. Telegram provides tools for writing bots. From telegram bots intro: "Bots are third-party applications that run inside Telegram. Users can interact with bots by sending them messages". Most news sites provide JSON, ATOM or RSS feeds. I checked existing solution and there are already a couple of Feed readers for telegrams out there. But I wasn't satisfied with any of them, none of them provided instant updates of feeds.  So my idea was to create RSS reader for Telegram bot that provided instant updates. I called it "El Monitorro".

A side note about another feed reader bot 'the feed reader bot'. Its feed refresh rate before I released my bot was 5 minutes and it was Premium feature. After I released my bot, which has refresh rate of 1 minute and it's free of charge, 'the feed reader bot' added new Elite level with refresh rate of 30 seconds. What do you think about squeezing it out of business by setting update rate to 20 seconds? LOL :)

In this post I'll give a brief explanation about the structure of the El Monitorro bot.

### Structure

The bot consists of four parts:

- Command frontend. It's used to communicate with the bot via commands.
- Sync process. It periodically syncs data feeds.
- Delivery process. It periodically delivers unread feed items to users.
- Cleaner process. It cleans stale data from the DB.

All these processes are connected to the same postges DB with tables:
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



#### Delivery process

#### Cleaner process

### Conclustion







- sync
- deliver
- bot
- cleaner

planned updates


--- conclusion
