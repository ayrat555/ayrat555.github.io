---
title: Frankenstein
date: 2021-04-30
summary: Telegram bot API client for Rust
categories: rust
---

![frank](/images/2021-04-30-frank.png)


If you have been following me for a while, you may know that I created a Telegram bot in Rust for reading rss/atom/json feeds called [El Monitorro](https://github.com/ayrat555/el_monitorro). I launched it in May of 2020. So it's been running for one year already.

The only thing that caused the bot to fail is a telegram client library used by the bot. The library isn't maintained for a while. And I had to fork it and fix some issues I found.

Instead of patching a fork of a not maintained library, I decided to create a new telegram client for Rust. I called it [Frankenstein](https://github.com/ayrat555/frankenstein). In this very short post, I'll describe a couple of approaches for generating API client I tried.

## Generating a client

### OpenApi spec

[OpenApi specification](https://swagger.io/specification/) has become a de facto standard for defining/describing web APIs. Many web services provide this specification. And there is [the official openapi api generator](https://github.com/OpenAPITools/openapi-generator) which can be used to generate api clients from openapi specs.

I already used the openapi api generator to generate api clients so my first approach was to try using open api specification of the telegram bot api to generate the client. Telegram doesn't provide an openapi spec but fortunately, some users already generated it.

The openapi api generator can generate clients for almost every language but unfortunately, the generator for Rust is not feature-complete. And it generated unusable project with missing structs and functions.

### Parsing API docs

The main piece of work that has to be done is to create data structures for types described in the telegram bot api docs. Instead of writing these data structures myself, I decided to parse them from the docs page and generate Rust data structures from the parsed API.

And it worked like a charm. It didn't take me long to create an html parser with [kuchiki](https://github.com/kuchiki-rs/kuchiki) and then write a code generator from parsed data with [codegen](https://github.com/carllerche/codegen). Along with rust structs and enums, I generated getter and setter methods for struct fields.

The total number of generated lines of code is more than 13_000. I'm glad that I didn't have to write them myself :)

After generating structures, all I had to do is to write and test wrappers for each method of the telegram bot api.

## Links

- [El Monitorro](https://github.com/ayrat555/el_monitorro)
- [Frankenstein](https://github.com/ayrat555/frankenstein)
- [Frankenstein Creator](https://github.com/ayrat555/frankenstein_creator)
