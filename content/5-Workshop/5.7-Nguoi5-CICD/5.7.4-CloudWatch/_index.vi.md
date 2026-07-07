---
title : "Amazon CloudWatch"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.7.4. </b> "
---

Phần này ghi lại hệ thống giám sát CloudWatch đã được cấu hình sẵn cho stack `periodiq-dev` - các log group mà Lambda function ghi vào, và các alarm theo dõi chúng.

#### Log group Lambda

```bash
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/periodiq-"
```

![Log group Lambda cho các function production](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/01-real-log-groups.png)

`periodiq-api-dev` và `periodiq-worker-dev` đều đã đặt retention 14 ngày; `periodiq-admin-api-dev` chưa đặt (retention `None` nghĩa là log được giữ vô thời hạn).

#### Các alarm

```bash
aws cloudwatch describe-alarms --query "MetricAlarms[?contains(AlarmName,'periodiq')].[AlarmName,StateValue]"
```

![8 alarm CloudWatch cho hệ thống production](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/02-real-alarms.png)

Có 8 alarm bao phủ các kịch bản lỗi của kiến trúc: lỗi 5xx của API, độ trễ API, lỗi invocation Lambda (API + Worker riêng biệt), concurrency Lambda, throttling DynamoDB, và message trong DLQ.

#### Một phát hiện đáng chú ý: alarm DLQ đang ở trạng thái `ALARM`

Kiểm tra trạng thái hiện tại từng alarm phát hiện ra điều đáng chú ý - `periodiq-dlq-messages-dev` không chỉ được cấu hình, nó **đang báo động**:

```bash
aws cloudwatch describe-alarms --alarm-names periodiq-dlq-messages-dev
aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/345798130457/periodiq-workout-processing-dlq-dev \
  --attribute-names ApproximateNumberOfMessages
```

![Phát hiện alarm DLQ - 9 message bị kẹt trong dead-letter queue](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/03-real-dlq-alarm-finding.png)

* Alarm vượt ngưỡng vào lúc `02/07/26 09:47` với **9 message** đang nằm trong `periodiq-workout-processing-dlq-dev`.
* Queue chính (`periodiq-workout-processing-dev`) hiện đang có **0** message - nên hiện tại không có gì mới đang lỗi, nhưng 9 message cũ đó (mỗi cái đã thất bại xử lý 3 lần, theo `maxReceiveCount` của queue) vẫn đang nằm chưa xử lý trong DLQ, và alarm chưa trở về OK vì chưa có datapoint mới báo "đã xuống dưới ngưỡng".

Đây chính xác là kiểu điều mà hệ thống giám sát ở [Worklog Tuần 11-12](../../../1-worklog/) được xây dựng để phát hiện: một tín hiệu cho thấy một số job thông báo giáo án đã thất bại lặp lại và cần kiểm tra thủ công (khả năng cao là qua `aws sqs receive-message` trên DLQ) thay vì bị lờ đi âm thầm.

> **CLI Note:** mọi giá trị trên trang này - tên/retention log group, trạng thái alarm, độ sâu queue - đều lấy từ lệnh `aws logs` / `aws cloudwatch` / `aws sqs` gọi trực tiếp trên account đang chạy. Phát hiện về DLQ không phải do tôi dàn dựng cho workshop này - đó là một tình trạng đã tồn tại sẵn trong hệ thống đang được giám sát.

#### Cùng dữ liệu đó, qua dashboard admin tôi đã xây dựng

Với vai trò Phạm Văn Sỹ, một phần của hạng mục CI/CD & Monitoring là xây dựng trang "Giám sát hệ thống" trong admin panel để hiển thị dữ liệu CloudWatch này cho con người xem, thay vì phải gọi CLI. Trang đang chạy tại `https://d1di1pzmfypszp.cloudfront.net/admin/monitoring`:

![Dashboard giám sát admin - cùng 8 alarm, alarm DLQ hiển thị "Đang cảnh báo"](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/04-admin-ui-overview.png)

Dashboard báo cáo đúng trạng thái như CLI: **1 alarm cảnh báo / 7 bình thường**, và card `Dlq Messages` được đánh dấu **"Đang cảnh báo"** - khớp với trạng thái `ALARM` đã xác nhận qua `describe-alarms` ở trên.

![Biểu đồ metric CloudWatch - invocations Lambda theo thời gian, API p95 duration](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/05-admin-ui-metrics-charts.png)

Cuộn xuống thêm cho thấy biểu đồ metric CloudWatch trực tiếp (invocations, API p95 duration) và một donut trạng thái alarm (8 alarm, 1 đang cảnh báo, 7 bình thường).

![Vòng tròn uptime và đúng 3 log group Lambda kèm retention, khớp chính xác với output CLI](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/06-admin-ui-uptime-loggroups.png)

Card "Log groups (Lambda) · retention" liệt kê đúng 3 log group và giá trị retention giống hệt lệnh `describe-log-groups` ở trên trang này: `periodiq-admin-api-dev` (Không hết hạn), `periodiq-api-dev` (14 ngày), `periodiq-worker-dev` (14 ngày).

![Dải lịch sử health-check - thanh xanh là check thành công, thanh đỏ là thất bại](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.4-CloudWatch/07-admin-ui-healthcheck-history.png)

Dải "Lịch sử kiểm tra" ở cuối trang là lịch sử health-check phía client - mỗi thanh là một lần gọi `/api/health`, xanh là OK và đỏ là lỗi - đây cũng chính là nguồn tính ra phần trăm uptime của dashboard (94.4-94.6% tại thời điểm chụp các ảnh này).
