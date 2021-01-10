---
layout: post
title: Performance testing with Datadog
date: 2021-01-12
summary: Performance testing with Datadog
categories: common
---

There are situations in a lifetime of a project when its all main features work well enough and it's time to make them work reliably well.

One approach is to use defensive perfomance tests. By "defensive perfomance tests"  I mean a test will fail if during its run some parameters exceed the specified values. For example, you don't want some part of your code to take more than the time it takes now, i.e. you don't want performance degradation.

In this post, I'll give a brief overview about implementing such tests using Datadog and CI service.

### Datadog

- What is datadog
- Long running performacne test

### Implementation

- Thresholds
- Datadog apis
- Datadog tags
- Resolving alerts

### Conclusion
