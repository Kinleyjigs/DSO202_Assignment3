# Cost and Service Report (Task 2)

All prices are from the **AWS Pricing Calculator**, Region **Asia Pacific (Singapore)**, in USD, checked on **20 November 2026**.

- Normal month estimate: <https://calculator.aws/#/estimate?id=1f0f91c63aa6d5b858c506f71bed99884f1ada5b>
- Exam month estimate: <https://calculator.aws/#/estimate?id=4808cd75b90d373aa7dd677b778fb338818664c7>
- Exports and screenshots are in [`evidence/`](../evidence/). The inputs I used are in [`calculator-inputs.md`](calculator-inputs.md).

---

## 1. What it costs

### Monthly cost per service

The database is a 1-year Reserved Instance, paid **USD 1,299 upfront**. To compare months fairly, I spread that over 12 months (**USD 108.25 a month**).

| Service | Normal month | Exam month | What drives it |
|---|---:|---:|---|
| Fargate: 2 API tasks, always on | 35.98 | 35.98 | 2 tasks × 730 h (ARM) |
| Fargate: extra exam tasks | – | 11.58 | 470 task-hours on exam days |
| Load balancer (ALB) | 19.57 | 20.27 | Mostly the hourly charge |
| NAT gateways ×2 + public IPv4 ×4 | 101.92 | 103.70 | Hourly charge, even when idle |
| RDS PostgreSQL Multi-AZ (Reserved Instance) | 108.25 | 108.25 | USD 1,299 ÷ 12 |
| RDS storage 50 GB + RDS Proxy | 40.08 | 40.08 | 13.80 storage + 26.28 proxy |
| CloudFront (to India) | 20.98 | 63.25 | 100 GB / 250 GB; 8.4 M / 30 M requests |
| WAF | 14.04 | 27.00 | Grows with requests |
| S3 submissions (incl. 100 GB downloads) | 18.90 | 18.90 | 12.00 of this is data out |
| CloudWatch logs, metrics, alarms | 13.65 | 24.21 | 10 GB / 25 GB of logs |
| AWS Backup | 2.10 | 2.10 | Daily 35 d + monthly backups |
| KMS | 1.90 | 1.90 | 1 key + 300k requests |
| **Total** | **377.37** | **457.23** | |

A few tiny services aren't in the calculator because together they come to under USD 3 a month: Secrets Manager 0.65, Route 53 0.50, SES 0.50, ECR 0.20 and cross-AZ traffic about 1.00, all from the AWS price list. SQS, Lambda, SNS and Cognito stay inside the free tier at our usage.

### Year total and cost per student

| | USD |
|---|---:|
| 10 normal months (10 × 269.12) | 2,691.20 |
| 2 exam months (2 × 348.98) | 697.96 |
| Database reservation (upfront) | 1,299.00 |
| Small services not in the calculator (12 × ≈ 2.85) | ≈ 34 |
| **Total for the year** | **≈ 4,722** |
| **Average per month** | **≈ 394** (budget is 1,000) |
| **Per student per year** (8,000 students) | **≈ 0.59** |

### Hidden costs

These are the ones that are easy to forget. In a normal month they add up to **about USD 143, roughly 38% of the bill**.

| Hidden cost | Normal | Exam | Why it's easy to miss |
|---|---:|---:|---|
| NAT gateway hours + processing | 87.32 | 89.09 | You pay every hour, even with zero traffic |
| Data transfer out (CloudFront + S3) | 22.90 | 39.25 | Every GB that leaves AWS is billed |
| Public IPv4 addresses | 14.60 | 14.60 | USD 0.005 an hour per IP, since 2024 |
| CloudWatch logs | 13.65 | 24.21 | USD 0.70 for every GB the app logs |
| Backups | 2.10 | 2.10 | Grows each year (about USD 20 a month by year 5, when 60 monthly copies are kept) |
| KMS requests | 1.90 | 1.90 | Every encrypt or decrypt is a request |
| Cross-AZ traffic | ≈ 1.00 | ≈ 2.00 | The price of using two AZs |

---

## 2. The three most expensive parts

1. **Database: USD 148 a month (39%).** Multi-AZ means we pay for two database servers, but only one does any work. The standby is there for the AZ-failure and RPO targets. RDS Proxy adds USD 26.
2. **NAT gateways + public IPs: USD 102 a month (27%).** There are two, one per AZ, so losing an AZ doesn't cut off the other. They charge by the hour, so this cost is the same with 10 users or 3,000.
3. **API servers (Fargate + ALB): USD 56 a month (15%).** Two tasks and a load balancer run all the time. In exam months **CloudFront (USD 63)** jumps into the top three because of all the autosave requests.

**Key point:** the big costs come from **resilience** (two AZs, standby database), not from traffic. That's why an exam month costs only 21% more than a normal month, even though the load is 6–10 times higher.

---

## 3. My choices compared with alternatives

### Backend: Fargate vs EC2 vs Lambda

| | **Fargate (chosen)** | EC2 Auto Scaling | Lambda |
|---|---|---|---|
| Cost / month | 36 | ≈ 30 (2 × t4g.small) | ≈ 20–30 incl. API Gateway |
| Performance | Steady; pre-scaled before exams | Same, but slower to add servers (2–4 min) | Cold starts right when 3,000 students click Start |
| Resilience | Tasks in 2 AZs, replaced automatically | Same | Built in |
| Effort | Low: runs our existing container | High: we patch the servers | Medium: code changes |

**Why Fargate:** about the same price as EC2 with no servers to look after, and no cold starts at exam time.

### Database: RDS Multi-AZ vs Aurora Serverless v2 vs DynamoDB

| | **RDS Multi-AZ (chosen)** | Aurora Serverless v2 | DynamoDB |
|---|---|---|---|
| Cost / month | 122 (RI + storage) | ≥ 146 (USD 0.20 per ACU-hour, min 2 × 0.5 ACU); ≈ 300 at real load | ≈ 10–25 |
| Performance | Enough for about 100 saves/s with RDS Proxy | Scales up by itself | Practically unlimited |
| Resilience | Standby takes over in 1–2 min, no data lost | Takes over in < 30 s | Built in |
| Effort | None | Low | **High**: redesign the tables, rewrite the API |

**Why RDS:** the cheapest option that keeps our relational data model and meets the RPO and RTO.

### Private traffic: NAT gateway vs VPC endpoints

| | **2 NAT gateways (chosen)** | VPC interface endpoints | 1 NAT gateway |
|---|---|---|---|
| Cost / month | 87 + IPs | 7 endpoints × 2 AZs × USD 0.013/h = **133** | 44 |
| Resilience | Each AZ has its own way out | Good | **Single point of failure** |
| Effort | None | Medium | None |

**Why 2 NATs:** the cheapest option that still survives losing an AZ. I did add the **free S3 gateway endpoint**, so S3 traffic skips the NAT.

### Paying: On-Demand vs Reserved vs Savings Plan

| | On-Demand | **1-year Reserved, all upfront (chosen for RDS)** | Compute Savings Plan |
|---|---|---|---|
| RDS per month | 148.19 | **108.25** | doesn't apply to RDS |
| Saving | – | **27%** | ≈ 20% on Fargate |
| Risk | None | Locked to this size for a year | Low |

**Why:** the database runs 24/7, so reserving it is an easy win. A Savings Plan for the 2 Fargate tasks would save about USD 7 more a month. I've left it as a next step.

### 5-year file storage: S3 Standard vs Intelligent-Tiering vs Glacier

| | S3 Standard only | Intelligent-Tiering | **Lifecycle to Deep Archive (chosen)** |
|---|---|---|---|
| Cost / month for 1.24 TB | ≈ 31 | ≈ 12 + monitoring fee | **≈ 7** |
| Opening old files | Instant | Instant | Up to 12 h after year 1 |

**Why:** old submissions are only needed for appeals, so waiting 12 hours is fine.

---

## 4. Singapore vs Mumbai

| | **Singapore** | Mumbai |
|---|---|---|
| Normal month (same design) | **377** | ≈ 337 (about 11% cheaper) |
| Biggest differences | – | Fargate ARM about 40% cheaper; RDS reservation USD 1,084 vs 1,299 |
| Distance from Thimphu | ≈ 3,100 km | ≈ 2,000 km |
| Latency from Bhutan (my test) | **_ ms_** | **_ ms_** |

Mumbai prices are from the AWS price list for `ap-south-1`. NAT is assumed to cost about the same.

**What it means:** Mumbai is a bit cheaper and closer. However:
- the brief asks for Singapore;
- CloudFront serves the website from India **in either case**, so only the autosave calls feel the extra distance, and a background save that's a few tens of ms slower isn't noticeable;
- the saving (about USD 480 a year) doesn't change the verdict.

**Singapore is fine. Mumbai would be a sensible second choice.**

---

## 5. Feasibility verdict

| Target | Result | Met? |
|---|---|---|
| **Budget:** USD 1,000/month average | **≈ USD 394/month**, 39% of the budget | ✅ |
| **Peak:** 3,000 students in 5 min | 8 tasks pre-scaled (auto scaling up to 20), RDS Proxy, WAF fix for the campus IP | ✅ by design; proven in the load test (Task 3) |
| **RPO:** 5 min | ≈ 0 for an AZ failure (synchronous standby); ≤ 5 min with point-in-time restore | ✅ |
| **RTO:** 1 hour | About 1–2 min for database failover | ✅ |
| **Lose one AZ** | Everything runs in 2 AZs | ✅ |
| **Keep files 5 years** | S3 lifecycle + monthly backups | ✅ |

**Verdict: yes, it's affordable and it works.** It costs about **USD 394 a month**, well under the USD 1,000 budget. Most of that is the price of being resilient, not traffic, so even double the users would still fit.

---

## 6. Optimisations and what they save

Every number here comes from the same AWS prices the calculator uses.

| # | Optimisation | Before | After | Saved / month | Saved / year |
|---|---|---:|---:|---:|---:|
| O1 | RDS **Reserved Instance** (1 yr, all upfront) instead of On-Demand | 148.19 | 108.25 | **39.94** | **479** |
| O2 | Turned off **Database Insights (Advanced)**; free Standard is enough | 18.25 | 0.00 | **18.25** | **219** |
| O3 | Pre-scale **only on exam days**, not every weekday as in Assignment 2 | 10.57 | 0.00 | **10.57** | **106** (10 normal months) |
| O4 | **ARM (Graviton)** Fargate instead of x86 | 44.98 | 35.98 | **9.00** | **108** |
| | **Total** | | | **≈ 77.76** | **≈ 912** |

**Not applied yet, but possible:** a Compute Savings Plan on Fargate (≈ USD 7 a month), and the CloudFront free tier (1 TB + 10 M requests a month, ≈ USD 21 in normal months).
