---
title: Performance testing with Datadog
date: 2021-01-17
summary: Performance testing with Datadog
categories: common
---

![cover](/images/2021-01-17-datadog.jpeg)

There are situations in a lifetime of a project when all main features work well enough and it's time to make them work reliably well.

One approach is to use defensive performance tests. By "defensive performance tests"  I mean a test will fail if during its run some parameters exceed the specified values. For example, you don't want some part of your code to take more than the time it takes now, i.e. you don't want performance degradation.

In this post, I'll give a brief overview of implementing such tests using Datadog and CI service.

### Datadog Moniors

Datadog is a service which collects performance parameters from web applications. Collected data can be used for visualization, analyzation, investigation of outages, alerts and other things.

Datadog has many different features but today we are interested in metric monitors. Because we will use them for our performance tests.

A metric monitor provides alerts and notifications if a specific metric is above or below a certain threshold.

We will need the following configurable conditions:
- Alert threshold - the value used to trigger an alert notification.
- Definition - possible values are `on average`, `at least once`, `at all times` and `in total`. We will use `at least` once - if any single value in the generated series crosses the threshold, then an alert is triggered. But based on different situations, another value can be used.

When a monitor is triggered it emits an event telling what exactly went wrong. And its state change from normal to failed.

Let's get to the main idea.

### Implementation

Datadog provides Rest API which allows interacting with Datadog. For our tests we will need two endpoints:
- `api/v1/events` - the event stream can be queried and filtered by time, priority, sources and tags.
- `monitor/bulk_resolve` - it resolves monitor.

So we should implement the following steps:

1. We should create monitors in datadog that will monitor specific metrics
2. During the test run, our project will send metrics to datadog
3. After the test, we should query datadog for events triggered by our monitors
4. If we find events triggered by any of our monitors, we will fail the test, otherwise the test is a success.
5. This step is required when a monitor failed on the previous step. Failed monitors don't emit any events. So we will have to resolve it so on the next test run, we can use it.

Another thing to consider is tags. If you're running several ci runs simultaneously, the monitor can fail for one run but not fail for another (because it's already failed). Using tags solves this problem: you'll have to generate a unique tag for each run, submit metrics using this tag and query events containing this tag.
