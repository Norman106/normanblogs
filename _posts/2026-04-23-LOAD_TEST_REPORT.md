---
layout: post
title: "Search Engine Load Test Report: ParadeDB vs Elasticsearch"
author: "Norman Fwamba"
categories: [Performance Testing, Search Systems, DevOps]
tags: [Load Testing, k6, Elasticsearch, PostgreSQL, ParadeDB, BM25, Performance Benchmarking, .NET, Docker, Scalability]
description: "A detailed performance benchmark comparing ParadeDB (PostgreSQL BM25) and Elasticsearch under load, highlighting speed, scalability, cost, and operational trade-offs to guide production search architecture decisions."
image: "https://www.paradedb.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fhero.91ca554a.png&w=2048&q=75&dpl=dpl_E3obiTZwLkk9A1s5AuCFGPyfaYgk"
---

# Search Engine Load Test Report
## ParadeDB (PostgreSQL BM25) vs Elasticsearch
### Proof of Concept — Performance Benchmark

---

> **Prepared by:** Norman Fwamba
> **Date:** 22 April 2026
> **Test Tool:** k6 v1.7.1
> **Application:** SearchPOC (.NET 9, EF Core, Docker)
> **Environment:** Local (Windows 11, Docker Desktop)

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [What Are We Comparing?](#2-what-are-we-comparing)
3. [How the System Works](#3-how-the-system-works)
4. [Test Setup & Methodology](#4-test-setup--methodology)
5. [ParadeDB — Detailed Results](#5-paradedb--detailed-results)
6. [Elasticsearch — Detailed Results](#6-elasticsearch--detailed-results)
7. [Head-to-Head Comparison](#7-head-to-head-comparison)
8. [What the Numbers Mean (Plain English)](#8-what-the-numbers-mean-plain-english)
9. [Infrastructure & Operational Comparison](#9-infrastructure--operational-comparison)
10. [Cost Analysis](#10-cost-analysis)
11. [Risk Assessment](#11-risk-assessment)
12. [Recommendation](#12-recommendation)
13. [Appendix — Technical Details](#13-appendix--technical-details)

---

## 1. Executive Summary

We conducted a rigorous load test comparing two search technologies:

- **ParadeDB** — a modern search extension built on top of PostgreSQL (the world's most popular open-source database)
- **Elasticsearch** — the industry-standard dedicated search engine, widely used by Netflix, LinkedIn, and GitHub

Both were tested under identical conditions: up to **100 simultaneous users**, over a **3-minute test window**, performing real-world operations (creating records and searching them).

### Key Findings at a Glance

| What We Measured | ParadeDB | Elasticsearch | Winner |
|---|---|---|---|
| Average search speed | 194 ms | **33 ms** | Elasticsearch |
| Speed at 95th percentile | 651 ms | **101 ms** | Elasticsearch |
| Requests per second | 44.9 | **55.7** | Elasticsearch |
| Error rate | **0%** | **0%** | Tie |
| Data write (create) avg | 264 ms | **47 ms** | Elasticsearch |
| Passed all thresholds | No | **Yes** | Elasticsearch |

> **Bottom Line:** Elasticsearch is significantly faster than ParadeDB under load — roughly **5.8x faster** on average search queries and **6.4x faster** at peak (p95). Both had zero errors, meaning both are reliable. The difference is purely in speed and capacity.

---

## 2. What Are We Comparing?

### 2.1 ParadeDB

**Think of it like this:** ParadeDB is like upgrading your existing filing cabinet with a supercharged search tab. Instead of replacing your whole storage system, you just make the search capability much smarter.

Technically:
- ParadeDB is **PostgreSQL** (a traditional database) with a special search index called **BM25** added on top
- BM25 (Best Match 25) is a well-known algorithm for ranking text search results by relevance
- All your data lives in **one place** — the same database that stores everything else
- No extra server, no extra service, no extra cost

### 2.2 Elasticsearch

**Think of it like this:** Elasticsearch is like hiring a dedicated librarian whose only job is to index and find books. Incredibly fast at searching, but it's a separate person you need to manage, pay, and keep in sync with the main library.

Technically:
- Elasticsearch is a **standalone search engine** built specifically for fast full-text search
- It stores a copy of your data in its own optimised format
- Runs as a **separate service** alongside your main database
- Used by some of the largest companies in the world for high-volume search

---

## 3. How the System Works

The POC application (SearchPOC) was built to test both technologies fairly under the same conditions.

```
User Request
     |
     v
.NET 9 Web API (SearchPOC)
     |
     +------------------+
     |                  |
     v                  v
 ParadeDB           Elasticsearch
 (Port 5435)        (Port 9200)
 PostgreSQL +        Dedicated
 BM25 Index          Search Index
```

**Data flow:**
1. When a bank record is **created**, it is saved to PostgreSQL (ParadeDB) AND simultaneously sent to Elasticsearch
2. When a **search is performed**, the request goes to either ParadeDB or Elasticsearch depending on the mode selected
3. Both return a list of matching bank records ranked by relevance

**Data model tested:**

| Field | Type | Description |
|---|---|---|
| ID | UUID | Unique identifier |
| Name | Text | Bank name (e.g., "Standard Bank-482931") |
| Active | Boolean | Whether bank is currently active |
| DateCreated | DateTime | When the record was added |

---

## 4. Test Setup & Methodology

### 4.1 Test Environment

| Component | Details |
|---|---|
| Machine | Windows 11, Local Developer Machine |
| Application | .NET 9 Minimal API, `dotnet run` |
| ParadeDB | Docker container, port 5435 |
| Elasticsearch | Docker v8.13.0, port 9200 |
| Load Test Tool | k6 v1.7.1 (Grafana) |
| Test Duration | 3 minutes per engine |
| Max Virtual Users | 100 |

### 4.2 Load Profile (Ramp-Up Pattern)

Both tests used identical stages:

| Stage | Duration | Users | Purpose |
|---|---|---|---|
| 1 — Warm-up | 30 seconds | 10 | Let the system settle |
| 2 — Ramp-up | 60 seconds | 50 | Gradual increase |
| 3 — Peak Load | 60 seconds | 100 | Maximum stress |
| 4 — Cool-down | 30 seconds | 0 | Graceful shutdown |

### 4.3 What Each Virtual User Did

Every simulated user repeated this loop continuously:

1. **CREATE** a new bank record with a random name
2. Wait 0.5 seconds
3. **SEARCH** for banks using a keyword (e.g., "Standard", "FNB", "Discovery")
4. Wait 1 second
5. Repeat

### 4.4 Performance Thresholds (Pass/Fail Criteria)

| Metric | Acceptable Level | Meaning |
|---|---|---|
| p(95) search latency | < 600ms | 95% of searches must complete in under 0.6 seconds |
| p(99) search latency | < 1000ms | 99% of searches must complete in under 1 second |
| Success rate | > 95% | At least 95 out of every 100 requests must succeed |
| Error rate | < 5% | Fewer than 1 in 20 requests can fail |

---

## 5. ParadeDB — Detailed Results

### 5.1 Summary

> ParadeDB completed **4,058 search iterations** with **zero errors** but **breached both latency thresholds** under peak load.

### 5.2 Core Metrics

| Metric | Value |
|---|---|
| Total Iterations | 4,058 |
| Total HTTP Requests | 8,116 |
| Throughput | 44.94 requests/second |
| Error Rate | 0.00% |
| Success Rate | 100.00% |
| Data Received | 22 MB (122 kB/s) |
| Data Sent | 1.1 MB (6.2 kB/s) |

### 5.3 Search Latency Breakdown

| Percentile | Time | Notes |
|---|---|---|
| Minimum | 15.61 ms | Fastest single search |
| Average | 194.85 ms | Typical search time |
| Median (p50) | 117.49 ms | Half of searches faster than this |
| p(90) | 441.42 ms | 90% of searches faster than this |
| **p(95)** | **651.65 ms** | **FAILED — threshold was 600ms** |
| **p(99)** | **1,130 ms** | **FAILED — threshold was 1,000ms** |
| Maximum | 2,010 ms | Slowest single search (2 seconds) |

### 5.4 Write (Create) Latency Breakdown

| Percentile | Time |
|---|---|
| Minimum | 21.72 ms |
| Average | 264.10 ms |
| Median | 114.30 ms |
| p(90) | 700.78 ms |
| p(95) | 1,000 ms |
| Maximum | 2,680 ms |

### 5.5 Threshold Results

| Threshold | Target | Actual | Status |
|---|---|---|---|
| p(95) search < 600ms | 600ms | 651.65ms | FAILED |
| p(99) search < 1000ms | 1,000ms | 1,130ms | FAILED |
| Success rate > 95% | 95% | 100% | PASSED |
| Error rate < 5% | 5% | 0% | PASSED |

### 5.6 Behaviour Under Load

ParadeDB started reasonably well at low user counts but began slowing down noticeably as users climbed past 50. At 100 concurrent users, some queries exceeded 2 seconds. This is because ParadeDB uses the same database engine (PostgreSQL) for both storing data AND searching — these two operations compete for the same CPU and memory when load is high.

---

## 6. Elasticsearch — Detailed Results

### 6.1 Summary

> Elasticsearch completed **5,041 search iterations** with **zero errors** and **passed all performance thresholds** comfortably.

### 6.2 Core Metrics

| Metric | Value |
|---|---|
| Total Iterations | 5,041 |
| Total HTTP Requests | 10,082 |
| Throughput | 55.74 requests/second |
| Error Rate | 0.00% |
| Success Rate | 100.00% |
| Data Received | 30 MB (168 kB/s) |
| Data Sent | 1.4 MB (7.7 kB/s) |

### 6.3 Search Latency Breakdown

| Percentile | Time | Notes |
|---|---|---|
| Minimum | 10.01 ms | Fastest single search |
| Average | **33.48 ms** | Typical search time |
| Median (p50) | **22.25 ms** | Half of searches faster than this |
| p(90) | **57.65 ms** | 90% of searches faster than this |
| **p(95)** | **101.23 ms** | **PASSED — well within 600ms** |
| **p(99)** | **198.96 ms** | **PASSED — well within 1,000ms** |
| Maximum | 405.62 ms | Slowest single search |

### 6.4 Write (Create) Latency Breakdown

| Percentile | Time |
|---|---|
| Minimum | 18.78 ms |
| Average | **47.23 ms** |
| Median | **32.92 ms** |
| p(90) | 85.77 ms |
| p(95) | 123.52 ms |
| Maximum | 469.90 ms |

### 6.5 Threshold Results

| Threshold | Target | Actual | Status |
|---|---|---|---|
| p(95) search < 600ms | 600ms | 101.23ms | PASSED |
| p(99) search < 1000ms | 1,000ms | 198.96ms | PASSED |
| Success rate > 95% | 95% | 100% | PASSED |
| Error rate < 5% | 5% | 0% | PASSED |

### 6.6 Behaviour Under Load

Elasticsearch remained consistent throughout all load stages. Even at 100 concurrent users, the search latency barely moved — a hallmark of a system specifically engineered for search. The inverted-index architecture means searches never compete with writes for core resources.

---

## 7. Head-to-Head Comparison

### 7.1 Search Speed

| Metric | ParadeDB | Elasticsearch | ES Advantage |
|---|---|---|---|
| Average | 194.85 ms | 33.48 ms | **5.8x faster** |
| Median | 117.49 ms | 22.25 ms | **5.3x faster** |
| p(90) | 441.42 ms | 57.65 ms | **7.7x faster** |
| p(95) | 651.65 ms | 101.23 ms | **6.4x faster** |
| p(99) | 1,130 ms | 198.96 ms | **5.7x faster** |
| Maximum | 2,010 ms | 405.62 ms | **5.0x faster** |

### 7.2 Write Speed

| Metric | ParadeDB | Elasticsearch | ES Advantage |
|---|---|---|---|
| Average | 264.10 ms | 47.23 ms | **5.6x faster** |
| Median | 114.30 ms | 32.92 ms | **3.5x faster** |
| p(95) | 1,000 ms | 123.52 ms | **8.1x faster** |
| Maximum | 2,680 ms | 469.90 ms | **5.7x faster** |

### 7.3 Throughput

| Engine | Requests/Second | Iterations Completed | Data Received |
|---|---|---|---|
| ParadeDB | 44.94 | 4,058 | 122 kB/s |
| Elasticsearch | **55.74** | **5,041** | **168 kB/s** |
| Difference | +24% ES | +24% ES | +38% ES |

### 7.4 Reliability

| Metric | ParadeDB | Elasticsearch |
|---|---|---|
| Error Rate | 0.00% | 0.00% |
| Failed Checks | 0 / 12,174 | 0 / 15,123 |
| Success Rate | 100% | 100% |

> Both engines are equally reliable — zero errors recorded. The difference is speed, not stability.

### 7.5 Overall Scorecard

| Category | ParadeDB | Elasticsearch |
|---|---|---|
| Search Speed | 2/5 | 5/5 |
| Write Speed | 2/5 | 5/5 |
| Throughput | 3/5 | 4/5 |
| Reliability | 5/5 | 5/5 |
| Threshold Compliance | 2/4 | 4/4 |
| Operational Simplicity | 5/5 | 2/5 |
| Cost | 5/5 | 3/5 |
| **Overall** | **24/35** | **28/35** |

---

## 8. What the Numbers Mean (Plain English)

### Is 195ms fast or slow?

Human perception guidelines from the UX industry:

| Response Time | User Experience |
|---|---|
| Under 100ms | Feels instant — like flipping a light switch |
| 100 – 300ms | Fast — user barely notices |
| 300 – 1,000ms | Noticeable delay — user waits |
| Over 1,000ms | Frustrating — user thinks something is broken |

| Engine | Average Search | User Experience |
|---|---|---|
| Elasticsearch | 33ms | Feels instant |
| ParadeDB | 195ms | Noticeable but acceptable |
| ParadeDB at p(95) | 652ms | Clearly slow to users |
| ParadeDB at p(99) | 1,130ms | Frustrating experience |

### What does "100 virtual users" mean in the real world?

100 simultaneous virtual users roughly corresponds to a system under moderate business load — imagine 100 employees or customers all clicking "Search" at the same moment. For a banking or financial application, this is a realistic mid-day peak scenario.

### Why did ParadeDB slow down but Elasticsearch didn't?

**ParadeDB** uses PostgreSQL's internal processes for search. When many users search simultaneously, PostgreSQL must divide its CPU, memory, and disk I/O between search queries, writing new records, and maintaining indexes. This competition for shared resources causes latency to grow under load.

**Elasticsearch** is a dedicated search engine. Its entire architecture is built around answering many simultaneous search queries fast. It uses a technique called an **inverted index** — think of it like the index at the back of a textbook — which allows it to find results almost instantly regardless of how many users are searching at the same time.

---

## 9. Infrastructure & Operational Comparison

### 9.1 Architecture Complexity

| Factor | ParadeDB | Elasticsearch |
|---|---|---|
| Number of services to run | 1 (PostgreSQL only) | 2 (PostgreSQL + Elasticsearch) |
| Data synchronisation needed | No | Yes — must sync all writes |
| Risk of data being out of sync | None | Yes — if sync fails, search is stale |
| Setup complexity | Low | Medium |
| Maintenance burden | Low | Medium-High |

### 9.2 Data Consistency Risk

With Elasticsearch, every time a record is created, updated, or deleted in the main database, the application must **also** update Elasticsearch. If this sync step fails:
- The database has correct, up-to-date data
- Elasticsearch has stale or wrong data
- Search results could return outdated information

During this POC, Elasticsearch returned **0 results** in initial testing due to a data synchronisation gap. This is a real operational risk that requires careful engineering to manage.

ParadeDB has **no synchronisation risk** — there is only one system to maintain.

### 9.3 Team Skill Requirements

| Requirement | ParadeDB | Elasticsearch |
|---|---|---|
| Database expertise needed | Standard PostgreSQL skills | PostgreSQL + Elasticsearch admin |
| Monitoring | Standard DB monitoring | DB monitoring + ES cluster monitoring |
| Backup strategy | Single database backup | Database backup + ES snapshots |
| Scaling approach | Scale DB vertically | Scale ES cluster independently |

---

## 10. Cost Analysis

### 10.1 Infrastructure Cost (Cloud Estimate)

| Item | ParadeDB | Elasticsearch |
|---|---|---|
| Main database server | Required | Required |
| Search service | Not needed | Additional instance ($100–$400/month) |
| Licensing | Free (open source) | Free (basic) / Paid (advanced) |
| Additional DevOps time | Minimal | Moderate |

### 10.2 Summary

- ParadeDB is the **lower cost** option — no additional infrastructure
- Elasticsearch adds meaningful infrastructure cost but delivers significantly better performance
- For a production banking system serving hundreds of users, the performance gains of Elasticsearch typically justify the additional cost

---

## 11. Risk Assessment

### 11.1 ParadeDB Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Slow search under high load | High (observed) | Medium | Caching, index tuning |
| BM25 index limitations at scale | Medium | Medium | Test with full production data volumes |
| Smaller community/support | Low | Medium | Monitor project maturity |

### 11.2 Elasticsearch Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Data sync failure | Medium | High | Retry logic, message queues, monitoring |
| Increased operational complexity | Certain | Medium | DevOps training, documentation |
| Higher infrastructure cost | Certain | Low-Medium | Budget planning |
| Stale search results during outages | Medium | Medium | Monitoring, sync lag alerts |

---

## 12. Recommendation

### Recommended: Elasticsearch

Based on the load test results and overall analysis, **Elasticsearch is the recommended search technology** for production use.

### Reasons

**1. Performance is decisive.**
Elasticsearch is 5–8x faster than ParadeDB across every metric measured. At 100 concurrent users, ParadeDB's 95th-percentile response exceeded the acceptable threshold. Elasticsearch sat comfortably at 101ms — with enormous headroom for growth.

**2. Both are equally reliable.**
Neither system produced a single error in the entire test. So reliability is not a differentiator — speed is the deciding factor.

**3. Elasticsearch scales with your business.**
As user counts grow from 100 to 500 to 1,000+, Elasticsearch is designed to scale horizontally by adding more nodes. ParadeDB performance would continue to degrade under increasing load.

**4. Industry-proven at scale.**
Elasticsearch powers search for Netflix, GitHub, LinkedIn, Wikipedia, and thousands of enterprise systems worldwide. It is battle-tested at scales far beyond what this application requires.

### When ParadeDB Would Be Better

ParadeDB is the right choice if:
- The application has **low search traffic** (under 20–30 concurrent users)
- **Operational simplicity** is more important than raw performance
- The team lacks capacity to maintain two separate services
- **Budget is very tight** and performance at current scale is acceptable
- You want a **quick MVP** without extra infrastructure complexity

### Adoption Roadmap

| Phase | Action |
|---|---|
| Immediate | Use Elasticsearch for the production search implementation |
| Short-term | Implement robust sync (retry queues, dead-letter handling, error alerts) |
| Medium-term | Add Kibana dashboards for real-time search monitoring |
| Long-term | Re-evaluate ParadeDB in 12–18 months as it matures |

---

## 13. Appendix — Technical Details

### 13.1 Test Script Configuration (Both Tests)

```javascript
stages: [
  { duration: "30s", target: 10  },  // warm-up
  { duration: "60s", target: 50  },  // ramp-up
  { duration: "60s", target: 100 },  // peak load
  { duration: "30s", target: 0   },  // cool-down
]
```

### 13.2 Application Endpoints Tested

| Endpoint | Method | Purpose |
|---|---|---|
| `/bank` | POST | Create a new bank record |
| `/search-hybrid?q={term}&useElastic=false` | GET | Search via ParadeDB BM25 |
| `/search-hybrid?q={term}&useElastic=true` | GET | Search via Elasticsearch |

### 13.3 ParadeDB BM25 Index Definition

```sql
CREATE INDEX bank_search_idx
ON bank
USING bm25 ("Name", "Active")
WITH (key_field='Id');
```

### 13.4 Docker Container Configuration

| Container | Image | Port |
|---|---|---|
| paradedb | paradedb/paradedb | 5435 |
| elasticsearch | elasticsearch:8.13.0 | 9200 |

### 13.5 Raw k6 Output Summary

**ParadeDB:**
```
create_latency.......: avg=264.1ms  min=21.72ms med=114.3ms  max=2.68s  p(90)=700.78ms p(95)=1s
parade_search_latency: avg=194.85ms min=15.61ms med=117.49ms max=2.01s  p(90)=441.42ms p(95)=651.65ms
http_req_failed......: 0.00%   0 out of 8116
http_reqs............: 8116    44.94/s
iterations...........: 4058    22.47/s
```

**Elasticsearch:**
```
create_latency.......: avg=47.23ms min=18.78ms med=32.92ms max=469.9ms  p(90)=85.77ms  p(95)=123.52ms
es_search_latency....: avg=33.48ms min=10.01ms med=22.25ms max=405.62ms p(90)=57.65ms  p(95)=101.23ms
http_req_failed......: 0.00%   0 out of 10082
http_reqs............: 10082   55.74/s
iterations...........: 5041    27.87/s
```

---

*Report generated from real k6 load test executions on 22 April 2026.*
*Test scripts: `k6-test/paradedb-load-test.js` | `k6-test/elasticsearch-load-test.js`*
*Raw results: `k6-test/paradedb-results.json` | `k6-test/elasticsearch-results.json`*
