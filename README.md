# Online Exam Platform: Cost, Feasibility and Resilience Analysis

### Assignment 3 (DSO303)

This assignment takes the exam platform I built in Assignment 2 and works out what it costs to run in **Asia Pacific (Singapore), `ap-southeast-1`**, whether that fits the budget, and where it fails or slows down.

## 1. Summary

_To be completed._

## 2. Feasibility verdict

| Requirement | Target | Result | Met? |
|---|---|---|---|
| Budget | USD 1,000 / month (yearly average) | _TBD_ | _TBD_ |
| Peak load | 3,000 students start one exam within 5 minutes | _TBD_ | _TBD_ |
| RPO | 5 minutes | _TBD_ | _TBD_ |
| RTO | 1 hour | _TBD_ | _TBD_ |
| AZ loss | Survives the loss of one AZ | _TBD_ | _TBD_ |
| Retention | Files kept for 5 years | _TBD_ | _TBD_ |

## 3. AWS Pricing Calculator estimates

Prices in USD, checked on _date_.

| Load profile | Calculator link |
|---|---|
| Normal month (300–500 concurrent users) | _link_ |
| Exam month (3,000-student peak) | _link_ |
| Optimised | _link_ |

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
