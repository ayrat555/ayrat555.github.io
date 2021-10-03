---
title:      Cronenberg
date:       2018-02-27
summary:    Simple cron command entry parser
categories: rust
---

![time](/images/2018-02-27-time.jpg)

In [my previous post](/rust/2018/01/28/in-rust-i-trust/) I wrote about the plan of how I am going to learn the Rust language. One of the the plans's points was to write a couple of simple libraries.

So here comes the first library that I wrote in Rust - Cronenberg.

### Background

A couple of weeks ago I finally found some time to dive into emacs's [org-mode](https://orgmode.org/) and I found it very useful. For those who don't know what org-mode is, it is a mode for keeping notes, maintaining TODO lists, and doing project planning with a fast and effective plain-text system.

I personally use it for planning my day, keeping track of my work tasks and gym programs. I synchronize org-mode files with my Android phone using open source [Orgzly](http://www.orgzly.com/) app (by the way, there are a couple of apps for ios too).

Org-mode has an amazing deadlines and scheduling feature. I got an idea to write a simple Rust app that will notify me about scheduled events in my org-mode files. [The project](https://github.com/ayrat555/doomsday) has a working title of 'Doomsday'. I use macOS at work and a linux distro at home so I decided to use cron for scheduling system notifications.

I encountered the first problem when I was writing interaction with crontab files: rust ecosystem has a library for parsing cron command entries but it uses nightly version of Rust and an older version of [nom](https://github.com/Geal/nom) (parser combinator framework). I decided to write a simple cron parser myself.

### Parsing cron command entries

`cronenberg` provides two core components

* `TimeItem`: An enum that represents cron command time or date field

```rust
pub enum TimeItem {
    AllValues,
    SingleValue(u8),
    MultipleValues(Vec<u8>),
    Interval((u8, u8)),
}
```

* `CronItem`: A struct that represents cron command entry, for example, `* * 5-7 1,2,5 8 sudo rm -rf /`

```rust
pub struct CronItem {
    pub minute: TimeItem,
    pub hour: TimeItem,
    pub day_of_month: TimeItem,
    pub month: TimeItem,
    pub day_of_week: TimeItem,
    pub command: String,
}
```

#### Usage example

```rust

let s = "* * 5-7 1,2,5 8 sudo rm -rf /";
assert_eq!(
    CronItem::from_str(s).unwrap(),
    CronItem {
        minute: AllValues,
        hour: AllValues,
        day_of_month: Interval((5, 7)),
        month: MultipleValues(vec![1, 2, 5]),
        day_of_week: SingleValue(8),
        command: String::from("sudo rm -rf /"),
    }
);

let cron_item = CronItem {
    minute: MultipleValues(vec![1, 10]),
    hour: Interval((1, 4)),
    day_of_month: Interval((1, 11)),
    month: MultipleValues(vec![1, 2, 5]),
    day_of_week: AllValues,
    command: String::from("sudo rm -rf /"),
};
assert_eq!("1,10 1-4 1-11 1,2,5 * sudo rm -rf /", cron_item.to_string());
```


I used [nom](https://github.com/Geal/nom) library for parsing. Although I heard good things about this crate, I'm not particularly fond of it. It extensively uses macros and learning this library was like learning a new programming language.

Here's example of parsing cron time item:
```rust
named!(command<&str, &str>,
       do_parse!(
           com: take_until!(COMMAND_TERMINATOR) >>
           tag!(COMMAND_TERMINATOR)             >>
           (com)
       )
);
```

### Roadmap

I'm planning to add more features to `cronenberg` in the future:
* words for days of the week: `Sun,...,Mon`
* words for months: `Jan,..,Dec`
* steps: `*/5`


The library has a GitHub repository - [cronenberg](https://github.com/ayrat555/cronenberg).

![cronenberg_quote](/images/2018-02-27-cronenberg.jpg)
