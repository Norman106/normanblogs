---
layout: post
title: "Presentation Guide: Logs & Metrics – Beyond the Basics"
author: "Norman Fwamba"
categories: [Observability, Software Engineering, DevOps]
tags: [Logs, Metrics, Observability, Presentation, Distributed Systems, Monitoring]
description: "A team workshop and presentation guide on moving from reactive logging to proactive observability with logs, metrics, and correlation IDs."
image: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRYHZY-IQl8zSapNEQ-HbfWVHjJcDKwgwN2WA&s"
---

# Presentation Guide: Logs & Metrics – Beyond the Basics

> **Presenter**: Norman Fwamba
> **Goal**: Move our team from "reactive logging" to "proactive observability."

---

## Slide 1: The "3 AM" Hook
**My Speech**: "Hey everyone, thanks for joining. I want to start with a story that I think most of us have lived through. It's 3:04 AM on a Tuesday. Your phone starts screaming at you—it's PagerDuty. The 'Checkout' service is failing. You’re bleary-eyed, staring at your monitor. 

Now, ask yourself: In that moment, do you open a dashboard, or do you start tailing logs? Usually, we panic and try both, but if we haven't set them up correctly, we're just staring at noise. Today, I want to talk about how we can use Logs and Metrics together so that next time this happens, we’re back in bed by 3:15."

**Content**:
- **Scenario**: The 3:04 AM on-call nightmare.
- **The Alert**: "Service 'Checkout' is failing."
- **The Question**: Dashboard (Metric) or Terminal (Logs)?
- **The Reality**: Without synergy, you're only seeing half the story.

---

## Slide 2: Metrics – The "What" (Our Vital Signs)
**My Speech**: "I like to think of Metrics as the vital signs of our system. Just like a doctor checks your pulse and blood pressure, metrics tell us *what* is happening. They are numbers, they are cheap to store, and they are incredibly fast. 

When I’m looking at metrics, I’m looking for the 'Four Golden Signals.' If our Latency spikes, or our Error rate climbs, I know exactly when the fire started. In a real-world scenario—say, a slow checkout—our P99 latency might jump from 200ms to 4 seconds. Searching 10 gigabytes of logs for the word 'slow' is a fool's errand. But looking at a graph? I can see that spike instantly."

**Content**:
- **Definition**: Aggregated numerical data points over time.
- **The Four Golden Signals**:
    1. **Latency**: How long requests take.
    2. **Traffic**: The demand on our system.
    3. **Errors**: The rate of failure.
    4. **Saturation**: How 'full' we are (CPU/Memory).
- **Key Takeaway**: Metrics tell you *something is wrong* immediately.

---

## Slide 3: Metric Pitfall – The Cardinality Explosion
**My Speech**: "Now, there’s a trap I’ve seen teams fall into—myself included. We get excited about metrics and want to track everything. We think, 'Hey, let’s add the User ID to this metric so we can see exactly who is having trouble!' 

Stop right there. That’s called a 'Cardinality Explosion.' If you have a million users and you add their ID as a label to a Prometheus metric, your monitoring server is going to have a heart attack. I once saw a team add email addresses to a metric; the server ran out of RAM and crashed right in the middle of a traffic spike. Keep your metrics high-level; save the unique details for the logs."

**Content**:
- **Cardinality**: The number of unique combinations of label values.
- **Good**: `status="200"`, `method="POST"`.
- **Bad**: `user_id="12345"`, `email="norman@example.com"`.
- **The Risk**: Monitoring system failure during critical incidents.

---

## Slide 4: Logs – The "Why" (Inside the Black Box)
**My Speech**: "If metrics are the pulse, logs are the surgery. They tell us *why* something happened. A log is a discrete event. It’s the story of a single request. 

In the old days, we just threw strings into a file—'User failed to checkout.' That’s okay, but it’s hard to search. What I really want us to move toward is **Structured Logging**. When we log in JSON, we aren't just writing text; we're creating a database of events. I can query exactly how many users hit a 'timeout' error on the 'payments-v2' service without writing a single regex."

**Content**:
- **Definition**: Discrete records of events at a specific time.
- **The Shift**: From Unstructured Strings to Structured JSON.
- **JSON Example**:
```json
{
  "timestamp": "2026-05-14T10:00:01Z",
  "level": "ERROR",
  "event": "checkout_failed",
  "user_id": 5521,
  "error_type": "timeout",
  "trace_id": "abc-123-xyz"
}
```

---

## Slide 5: Real-Life Example – Hunting the "Heisenbug"
**My Speech**: "Let me give you a real example of where logs saved my skin. We had this 'Heisenbug'—a rare error that only happened to a few people. Our conversion rate dropped by maybe 0.5%. Not enough to trigger a major metric alert, but enough to be annoying.

Because we had structured logs, I searched for `event: onboarding_step_failed`. I found a pattern: users were clicking 'Next' at the exact millisecond their session token expired. The log showed: Token Expired, then Request Received, then a NullPointer. You can’t 'measure' a NullPointer as a trend on a graph; you have to see it in the logs to understand the logic failure."

**Content**:
- **Scenario**: A rare race condition in an onboarding form.
- **The Investigation**: Searching for specific event patterns.
- **The Lesson**: Logs reveal the logic; metrics reveal the impact.

---

## Slide 6: The Synergy – How We Should Actually Work
**My Speech**: "This is the most important part of today's talk. How do we actually use these together? Here is the workflow I want us to adopt. 

First, a **Metric Alert** wakes us up. We look at the **Dashboard** to see the scope—is it just one service? Is the database struggling? Then, and only then, do we dive into the **Logs**. We use the time window from the metric spike to filter our logs. We find the error, we see the stack trace, we fix the code, and then we watch the metric go back to normal. It’s a loop, not a choice."

**Content**:
1. **Detect**: Metric alert fires.
2. **Isolate**: Dashboard shows which service is peaking.
3. **Diagnose**: Logs reveal the specific error/stack trace.
4. **Verify**: Metrics return to baseline after the fix.

---

## Slide 7: The Bridge – Correlation IDs (Trace IDs)
**My Speech**: "If there’s one thing I want you to take away today, it’s this: we need Correlation IDs. Imagine finding an error in the 'Payment' service logs. You copy a 'Trace ID,' you paste it into our tracing tool, and boom—you see the entire journey of that request across all five of our microservices. 

It turns a needle-in-a-haystack search into a simple walk through the park. If our logs don't have a shared `trace_id`, we’re essentially flying blind in a distributed system."

**Content**:
- **Concept**: A unique ID generated at the gateway and passed to all services.
- **The Goal**: Unified visibility across the entire request lifecycle.
- **Implementation**: Middleware injection of `X-Request-ID`.

---

## Slide 8: Audit Checklist – Are We Observable?
**My Speech**: "I’ll leave you with this checklist. Next time you’re building a feature, ask yourself these five things. Do we have the golden signals? Are our logs in JSON? Do we have a trace ID? Are our alerts actually actionable, or are they just noise? And finally, can you find a stack trace for an error in under 60 seconds? If the answer is no, we still have work to do."

**Content**:
- [ ] Golden Signals on a Dashboard?
- [ ] Structured JSON Logs?
- [ ] Shared Trace IDs?
- [ ] Actionable (not noisy) Alerts?
- [ ] < 60s to find a Root Cause?

---

## Slide 9: Q&A
**My Speech**: "I want to open it up now. What’s the hardest bug you’ve had to fix lately? Do you think better logs or better metrics would have made it easier? Let’s talk about where we can improve our stack together."

**Content**:
- Open Floor for Discussion.
- Identifying the 'Hardest Bug' of the month.
- Collaborative improvement plan.
