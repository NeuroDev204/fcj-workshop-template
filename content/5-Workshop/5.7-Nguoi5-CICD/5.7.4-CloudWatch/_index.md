---
title : "Amazon CloudWatch"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.7.4. </b> "
---

This covers the real CloudWatch monitoring already configured for the `periodiq-dev` stack - the log groups the Lambda functions write to, and the alarms that watch them.

#### Real Lambda log groups

```bash
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/periodiq-"
```

![Real Lambda log groups for the production functions](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/01-real-log-groups.png)

`periodiq-api-dev` and `periodiq-worker-dev` both have a 14-day retention policy set; `periodiq-admin-api-dev` doesn't have one set yet (retention `None` means logs are kept indefinitely).

#### Real alarms

```bash
aws cloudwatch describe-alarms --query "MetricAlarms[?contains(AlarmName,'periodiq')].[AlarmName,StateValue]"
```

![8 real CloudWatch alarms for the production system](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/02-real-alarms.png)

There are 8 alarms covering the architecture's failure modes: API 5xx errors, API latency, Lambda invocation errors (API + Worker separately), Lambda concurrency, DynamoDB throttling, and DLQ messages.

#### A real finding: the DLQ alarm is actually in `ALARM` state

Checking each alarm's current state turned up something genuinely worth flagging - `periodiq-dlq-messages-dev` isn't just configured, it's **currently alarming**:

```bash
aws cloudwatch describe-alarms --alarm-names periodiq-dlq-messages-dev
aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/345798130457/periodiq-workout-processing-dlq-dev \
  --attribute-names ApproximateNumberOfMessages
```

![Real DLQ alarm finding - 9 messages stuck in the dead-letter queue](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/03-real-dlq-alarm-finding.png)

* The alarm crossed its threshold on `02/07/26 09:47` with **9 messages** visible in `periodiq-workout-processing-dlq-dev`.
* The main queue (`periodiq-workout-processing-dev`) currently shows **0** messages - so nothing new is failing right now, but those 9 older messages (each one having failed processing 3 times, per the queue's `maxReceiveCount`) are still sitting unprocessed in the DLQ, and the alarm hasn't cleared because it hasn't gotten a fresh "back below threshold" datapoint.

This is exactly the kind of thing the monitoring in [Worklog Week 11-12](../../../1-worklog/) was built to surface: a real signal that some workout-plan notification jobs failed repeatedly and need manual inspection (likely via `aws sqs receive-message` against the DLQ) rather than something to silently ignore.

> **CLI Note:** every value on this page - log group names/retention, alarm states, queue depths - comes from live `aws logs` / `aws cloudwatch` / `aws sqs` calls against the real account. The DLQ finding wasn't set up for this workshop; it's a real, pre-existing condition in the monitored system.

#### The same data, through the admin dashboard I built

As Phạm Văn Sỹ, part of the CI/CD & Monitoring role was building a "Giám sát hệ thống" (System Monitoring) page in the admin panel that surfaces this exact CloudWatch data to a human, instead of requiring `aws` CLI calls. It's live at `https://d1di1pzmfypszp.cloudfront.net/admin/monitoring`:

![Admin monitoring dashboard - same 8 alarms, DLQ alarm shown as "Đang cảnh báo"](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/04-admin-ui-overview.png)

The dashboard reports the exact same state as the CLI: **1 alarm cảnh báo / 7 bình thường**, and the `Dlq Messages` card is flagged **"Đang cảnh báo"** - matching the `ALARM` state confirmed via `describe-alarms` above.

![CloudWatch metrics charts - Lambda invocations over time, API p95 duration](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/05-admin-ui-metrics-charts.png)

Scrolling further shows live CloudWatch metric charts (invocations, API p95 duration) and an alarm-state donut (8 alarms, 1 alerting, 7 normal).

![Uptime ring and the same 3 Lambda log groups with retention, matching the CLI output exactly](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/06-admin-ui-uptime-loggroups.png)

The "Log groups (Lambda) · retention" card lists the identical 3 log groups and retention values as the `describe-log-groups` call earlier on this page: `periodiq-admin-api-dev` (Không hết hạn / no expiry), `periodiq-api-dev` (14 ngày), `periodiq-worker-dev` (14 ngày).

![Health-check history bars - green bars are passing checks, red bars are failures](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/07-admin-ui-healthcheck-history.png)

The "Lịch sử kiểm tra" (check history) strip at the bottom is a client-side health-check history - each bar is one `/api/health` call, green for OK and red for failed - which is also where the dashboard's own uptime percentage (94.4-94.6% at the time of these screenshots) comes from.
