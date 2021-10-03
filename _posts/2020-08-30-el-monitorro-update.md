---
title: The last big features of El Monitorro
date: 2020-08-30
summary: El Monitorro Update
categories: common
---

![el-monitorro](/images/2020-08-30-bull.jpg)

If you've been following me, you know that for the last months I've been working on the feed reader bot for Telegram called El Monittorro. It's a toy project that I used to get some practice with the Rust programming language. After I launched the project in May of 2020, it gained around 100 users with over 250 feed subscriptions. I think it's a pretty good number considering the only promotion post I did was a Reddit post in the Telegram subreddit.

In this post I'll announce a couple of features I developed recently.

### Message templates

Initially, when I released the bot, all messages had the same hardcoded format:

```
Feed item name

Feed item date

Feed item link
```

Then I started receiving messages from users asking to change the format and each message contained specific requests. So I decided to add message templates which would allow a user to describe the format of messages he wants to receive. I added four commands:

- /set_template feed_url template - set template to the subscription
- /get_template feed_url - get template for the subscription
- /set_global_template template - set global template for all subscriptions
- /get_global_template - get global template

A template supports the following fields:

- bot_feed_name - the name of the feed
- bot_feed_link - url of the feed
- bot_item_name - the name of the item
- bot_item_link - url of the item
- bot_item_description - description of the item
- bot_date - publication date of the feed
- bot_space - defines a space character
- bot_new_line - defines a new line character

For example, if you want to receive messages in the format of:

```
Feed name

Feed item name

Feed item description

Feed item date

Feed item link
```

You should use the following template - `bot_feed_namebot_new_linebot_new_linebot_item_namebot_new_linebot_new_linebot_item_descriptionbot_new_linebot_new_linebot_datebot_new_linebot_new_linebot_item_link`


### Channels support

Another frequently requested feature was channels support. Initially, the bot supported only groups and private conversations. Now users can create news/announcement channels based on data feeds. For example, you can create an unofficial BBC news channel using RSS feed provided by the BBC. Or you can create a channel with memes fetching posts from memes subreddit on Reddit (Reddit also provided RSS feeds).

### Conclusion

I think these features will be the last big features for El Monitorro and from this point on I'll be only maintaining the project fixing bugs if they occur. You may ask "why?". In my opinion, the bot has all the features for an average user and it seems there is no demand for anything else. For the context, I implemented support of timezones, support for channels and message templates because I received requests for these features from multiple users.

If you're interested in the implementation details, you might want to check my [previous post](/rust/2020/07/31/el-monitorro/) and its [GitHub repo](https://github.com/ayrat555/el_monitorro). Also, I created a simple [landing page](https://www.badykov.com/el_monitorro/) for the bot which copies README from the repo so web crawlers can index it.
