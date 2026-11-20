# Online Exam Platform: Cost, Feasibility and Resilience Analysis

### Assignment 3 (DSO303)

This assignment takes the exam platform I built in Assignment 2 and works out what it costs to run in **Asia Pacific (Singapore), `ap-southeast-1`**, whether that fits the budget, and where it fails or slows down.

## 1. Summary

I priced my Assignment 2 exam platform in Singapore with the AWS Pricing Calculator. It costs about **USD 394 a month** on average (**≈ USD 4,722 a year**, or **USD 0.59 per student**), well under the USD 1,000 budget. Most of the cost comes from staying up when an AZ fails (Multi-AZ database, two NAT gateways), not from traffic.

While redesigning, I found three things that would slow down or break an exam: the WAF rate limit would block the shared campus IP, the API opened a new database connection for every request, and the marker Lambda had no concurrency limit. All three are fixed in the new design (see [`architecture/`](architecture/)).

Full report: [`docs/cost-report.md`](docs/cost-report.md).

## 2. Feasibility verdict

| Requirement | Target | Result | Met? |
|---|---|---|---|
| Budget | USD 1,000 / month (yearly average) | ≈ USD 394 / month | ✅ |
| Peak load | 3,000 students start one exam within 5 minutes | Pre-scaling + RDS Proxy + WAF fix | ✅ by design (load test: _TBD_) |
| RPO | 5 minutes | ≈ 0 (synchronous standby) | ✅ (failure test: _TBD_) |
| RTO | 1 hour | ≈ 1–2 min database failover | ✅ (failure test: _TBD_) |
| AZ loss | Survives the loss of one AZ | Everything in 2 AZs | ✅ |
| Retention | Files kept for 5 years | S3 lifecycle + monthly backups | ✅ |

## 3. AWS Pricing Calculator estimates

Prices in USD, checked on 20 November 2026.

| Load profile | Calculator link |
|---|---|
| Normal month (300–500 concurrent users): **USD 269.12/month + USD 1,299 upfront** | <https://calculator.aws/#/estimate?id=1f0f91c63aa6d5b858c506f71bed99884f1ada5b> |
| Exam month (3,000-student peak): **USD 348.98/month + USD 1,299 upfront** | <https://calculator.aws/#/estimate?id=4808cd75b90d373aa7dd677b778fb338818664c7> |

## 4. Demo video

_Link (5 minutes or less): one failure test and the load test._

## 5. Repository layout

```
architecture/        Revised diagram, single points of failure, change log against Assignment 2
docs/cost-report.md  Cost and service report (Task 2)
infra/               Terraform for the platform
infra/failure-tests/ Failure test scenarios, expected impact and fixes
evidence/            Pricing Calculator exports, CloudWatch screenshots
app/, lambdas/, frontend/, bootstrap/, scripts/   Platform code carried over from Assignment 2
```
