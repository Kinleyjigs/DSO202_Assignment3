# Pricing Calculator inputs

These are the values I entered at calculator.aws. Region: **Asia Pacific (Singapore)**. Checked on **20 November 2026**.

| Estimate | Link | Monthly | Upfront |
|---|---|---:|---:|
| Normal month | <https://calculator.aws/#/estimate?id=1f0f91c63aa6d5b858c506f71bed99884f1ada5b> | 269.12 | 1,299 |
| Exam month | <https://calculator.aws/#/estimate?id=4808cd75b90d373aa7dd677b778fb338818664c7> | 348.98 | 1,299 |

## Where the usage numbers come from

| Item | Normal | Exam | How I got it |
|---|---|---|---|
| API requests | 8.4 M | 30 M | Normal: 400 users × 2 req/min × 8 h × 22 days. Exam: plus 3,000 students saving every 30 s, 3 h a day, 20 days. |
| Extra exam tasks | 0 | 470 task-h | 6 extra tasks × 3.25 h × 20 days, plus a few auto-scaled ones |
| CloudFront out | 100 GB | 250 GB | The site is cached; API replies are small |
| Logs | 10 GB | 25 GB | API + Lambda logs |
| Submissions | 120 GB Standard, 120 GB IA, 1 TB Deep Archive | same | 8,000 students × ~6 files a year × 10 MB |

## What I entered

| Service | Normal month | Exam month (only the changes) |
|---|---|---|
| Fargate (baseline) | Linux, ARM, 2 tasks × 730 h, 0.5 vCPU, 1 GB | – |
| Fargate (exam burst) | – | ARM, 470 tasks × 1 h, 0.5 vCPU, 1 GB |
| Elastic Load Balancing | 1 ALB, 40 GB/month, 5 new conn/s, 60 s duration | 120 GB/month, 8 new conn/s |
| VPC | 2 NAT gateways × 10 GB/month; 4 in-use public IPv4 | NAT 25 GB/month each |
| RDS for PostgreSQL | db.t4g.medium, Multi-AZ, Reserved 1 yr All Upfront, gp3 50 GB, RDS Proxy yes, Database Insights no | – |
| CloudFront | India: 100 GB out, 8.4 M HTTPS requests | 250 GB, 30 M requests |
| WAF | 1 web ACL, 2 rules, 2 managed rule groups, 8.4 M requests | 30 M requests |
| S3 | Standard 120 GB (50k PUT, 200k GET), IA 120 GB, Deep Archive 1,000 GB, 100 GB data out | – |
| CloudWatch | 10 GB standard logs, 1-month retention, 6 alarms | 25 GB logs |
| AWS Backup | RDS: 20 GB data, +50%/year, 0.5% daily change, daily kept 35 days, monthly kept 12 months (the calculator's maximum) | – |
| KMS | 1 key, 300,000 requests | – |
