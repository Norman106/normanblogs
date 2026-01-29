---
layout: post
title: "Logs vs Metrics: A Practical Guide for Engineers"
author: "Norman Fwamba"
categories: [Observability, Software Engineering, DevOps]
tags: [Logs, Metrics, Monitoring, Observability, Prometheus, Grafana, System Reliability]
description: "A practical, engineer-focused guide that explains the differences between logs and metrics, when to use each, and how they work together to build reliable, observable systems."
image: "https://miro.medium.com/v2/resize:fit:1400/0*9NpLuMwnX30r6vDx"
---
![Code Refactoring](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRYHZY-IQl8zSapNEQ-HbfWVHjJcDKwgwN2WA&s)
# Logs vs Metrics: A Practical Guide for Engineers

Metrics tell you something is wrong. Logs tell you what is wrong. A practical guide on when to use each for effective observability.



I've lost count of how many times I've seen teams debate whether to add more logging or better metrics. The answer is almost always "both, but for different reasons." Here's how I think about it after years of debugging production systems.

## The Quick Distinction

Metrics tell you something is wrong. Logs tell you what is wrong.

That's the one-liner, but it glosses over the nuances that matter when you're building observability into a real system. Let's break it down.

## Logs: Your System's Black Box

Logs are event records. Every time something happens - a request comes in, a database query runs, an error gets thrown - you can write a log entry.

```
2026-01-16 14:23:45 [ERROR] Payment failed for user_id=12345, reason=card_declined
2026-01-16 14:23:46 [INFO] Retry initiated for transaction_id=abc123
```

I reach for logs when:

- A user reports a bug and I need to trace exactly what happened
- I'm debugging why a specific request failed
- Compliance requires an audit trail
- I need the full error message and stack trace

The downside? Logs are expensive. High-traffic systems can generate gigabytes per hour, and searching through them isn't instant. You're essentially doing text search across massive datasets.

If you're working with logs, understanding log levels is essential. Knowing when to use DEBUG vs INFO vs ERROR determines both the usefulness and the cost of your logging strategy.

### Structured vs Unstructured Logs

This is worth calling out separately. Unstructured logs look like this:

```
Payment failed for user 12345 because their card was declined
```

Structured logs look like this:

```json
{"level": "error", "event": "payment_failed", "user_id": 12345, "reason": "card_declined", "timestamp": "2026-01-16T14:23:45Z"}
```

Structured logs are searchable. You can filter by user_id, aggregate by reason, and build dashboards. Unstructured logs require regex and hope. I've seen teams waste days debugging issues that would have taken minutes with proper structured logging.

On Linux systems, syslog provides a standardized way to handle system-level logging, and understanding how it works helps when you're correlating application logs with system events.

## Metrics: The Vital Signs

Metrics are numbers over time. Request count, error rate, p99 latency, CPU usage - these are all metrics.

```
http_requests_total{status="200"} 145892
http_request_duration_seconds{quantile="0.99"} 0.25
```

I use metrics when:

- Setting up alerts (error rate > 1% for 5 minutes? Page someone)
- Building dashboards for the team
- Checking if a deploy made things better or worse
- Capacity planning - are we going to run out of headroom?

Metrics are cheap to store and fast to query. A time-series database like Prometheus can handle millions of data points and return results in milliseconds. The tradeoff is you lose the detail - you know how many errors happened, not which errors or why.

## The Four Golden Signals

Google's SRE book popularized the concept of four golden signals for monitoring:

- **Latency** - How long requests take
- **Traffic** - How many requests you're handling
- **Errors** - How many requests fail
- **Saturation** - How "full" your system is

If you're only going to track four things, track these. They cover most of what you need to know about whether your service is healthy. Once you have these as metrics, learning to query them effectively matters - for Prometheus users, understanding the rate function is fundamental.

## Key Differences at a Glance

| Aspect          | Logs                              | Metrics                          |
|-----------------|-----------------------------------|----------------------------------|
| Data type       | Text/structured events            | Numerical values                 |
| Cardinality     | High (unique per event)           | Low (aggregated)                 |
| Storage cost    | Higher                            | Lower                            |
| Query speed     | Slower (search-based)             | Faster (time-series)             |
| Best for        | "What happened?"                  | "How much/how often?"            |
| Retention       | Days to weeks typical             | Months to years typical          |

## The Real-World Workflow

Here's how this typically plays out during an incident:

1. Metrics alert fires - "Error rate exceeded 2% on the payments service"
2. Dashboard confirms - Yep, started 10 minutes ago, correlates with a deploy
3. Logs explain - Search for errors in that window, find "connection refused to payment gateway"
4. Root cause - New config didn't include the gateway's updated IP

Without metrics, you might not know there's a problem until users complain. Without logs, you'd know something's broken but have no idea what.

This workflow is at the heart of the logging vs monitoring discussion - they're not competing approaches, they're complementary.

## Common Mistakes I've Seen

- Logging everything at DEBUG level in production. Your storage costs will explode and you'll struggle to find the signal in the noise. Be intentional about what you log and at what level.
- High-cardinality metrics. Adding user_id as a label to every metric sounds useful until you have a million users and your Prometheus instance falls over. Keep label cardinality bounded.
- No correlation between logs and metrics. When an alert fires, you should be able to pivot directly to relevant logs. This means consistent timestamps, service names, and ideally request IDs that appear in both.
- Alerting on logs instead of metrics. "Alert me when this error message appears" sounds reasonable, but it's fragile. Error messages change, log parsing has edge cases. Alert on error rate metrics, then use logs to investigate.

## Where Traces Fit In

I'd be leaving out context if I didn't mention traces. In distributed systems, a single user request might touch ten services. Traces connect the dots, showing you the full journey of a request.

The modern observability stack - sometimes called the "three pillars" - combines:

- Metrics for alerting and dashboards
- Logs for detailed debugging
- Traces for understanding request flow

OpenTelemetry has become the standard for instrumenting all three, giving you a consistent way to collect and correlate telemetry data.

## Practical Recommendations

- Start with metrics for your SLIs. Define what "healthy" means for your service (latency under 200ms, error rate under 0.1%, availability above 99.9%) and build metrics to track those.
- Add structured logging at service boundaries. Log incoming requests, outgoing calls to dependencies, and errors. Include request IDs that flow through the system.
- Set retention policies early. Decide how long you need to keep logs (7 days? 30 days?) and metrics (90 days? 1 year?). This drives architecture and cost decisions.
- Invest in correlation. The ability to go from a metric alert to the relevant logs to a trace of the failing request is what separates "we debug fast" from "we debug eventually."

## The Cost Question

If budget is tight, invest in good metrics first. Add targeted logging for errors and critical paths. Expand from there as needed.

Many teams find that 80% of their logging cost comes from 20% of their log volume - usually verbose DEBUG logs that nobody looks at. Audit your logging periodically.

## Wrapping Up

Don't think of this as logs versus metrics. Think of it as logs **and** metrics, each doing what they're good at.

- Metrics for alerting and trends.
- Logs for debugging and audits.
- Traces for understanding distributed request flow.

Together, they give you the full picture.

The goal isn't perfect observability - it's observability that's good enough to debug issues quickly and confidently. Start simple, measure what matters, and expand based on the gaps you actually encounter.
```

