---
title : "Amazon DynamoDB"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1. </b> "
---

Dịch vụ đầu tiên trong ba dịch vụ thuộc vai trò của tôi: thiết kế toàn bộ schema, tạo bảng, GSI và seed data cho 8 bảng DynamoDB đứng sau PeriodIQ.

#### Liệt kê bảng đang chạy

```bash
aws dynamodb list-tables --region ap-southeast-1
```

![8 bảng DynamoDB đang chạy trong ap-southeast-1](/images/5-Workshop/5.6-Nguoi4-Admin/01-dynamodb-list-tables.png)

9 bảng được liệt kê (bao gồm `Progress` do Lê Hữu Duy Hoàng yêu cầu). 8 bảng thuộc phạm vi thiết kế của tôi:

| Nhóm | Bảng | Vai trò |
|---|---|---|
| Master data | `Exercise`, `WorkoutTemplate`, `RuleDefinition` | Admin quản lý, Rule Engine đọc |
| User profile | `UserProfile`, `PersonalRecordHistory`, `DailyCnsStatus` | Dữ liệu cá nhân người dùng |
| Generated plans | `WorkoutPlan`, `WorkoutSessionLog` | Kết quả sinh giáo án và nhật ký tập |

#### Nguyên tắc thiết kế

Tất cả 8 bảng dùng cùng một mô hình:
- **Partition key**: `Id` (String) — UUID cho mọi entity, riêng `UserProfile` dùng Cognito `sub` để khớp JWT claim trực tiếp
- **Billing**: `PAY_PER_REQUEST` — không cần ước lượng throughput trước, phù hợp với workload không đều của startup
- **Base fields**: `Id`, `CreatedAt`, `UpdatedAt` — kế thừa từ `BaseEntity` trong Domain layer

```bash
aws dynamodb describe-table --table-name Exercise --region ap-southeast-1 \
  --query "Table.{Keys:KeySchema,BillingMode:BillingModeSummary.BillingMode}"
```

![Exercise table — PAY_PER_REQUEST, partition key Id](/images/5-Workshop/5.6-Nguoi4-Admin/02-dynamodb-exercise-table.png)

#### GSI trên các bảng cần query phức tạp

5 trong 8 bảng có Global Secondary Index để hỗ trợ query theo `UserId`:

```bash
aws dynamodb describe-table --table-name UserProfile --region ap-southeast-1 \
  --query "Table.GlobalSecondaryIndexes[*].{Index:IndexName,Key:KeySchema}"
```

![UserProfile GSI Email-index — query user theo email](/images/5-Workshop/5.6-Nguoi4-Admin/03-dynamodb-userprofile-gsi.png)

| Bảng | GSI | Dùng để |
|---|---|---|
| `UserProfile` | `Email-index` (PK: Email) | Tìm user theo email khi đăng nhập |
| `PersonalRecordHistory` | `UserId-ExerciseId-index` (PK: UserId, SK: ExerciseId) | Lấy toàn bộ PR của 1 user cho 1 bài tập |
| `DailyCnsStatus` | `UserId-DateLog-index` (PK: UserId, SK: DateLog) | Lấy 7 ngày CNS gần nhất của 1 user |
| `WorkoutPlan` | `UserId-StartDate-index` (PK: UserId, SK: StartDate) | Lấy giáo án theo user, sort theo ngày |
| `WorkoutSessionLog` | `UserId-CompletedAt-index` (PK: UserId, SK: CompletedAt) | Lấy lịch sử buổi tập sort theo thời gian |

Lý do tất cả GSI dùng `UserId` làm Partition Key: toàn bộ query của ứng dụng đều bắt đầu từ "lấy dữ liệu của user X" — không có usecase nào cần scan toàn bảng để phục vụ người dùng cuối.

#### Kiểm tra số record trong từng bảng

```bash
for table in Exercise WorkoutTemplate RuleDefinition UserProfile PersonalRecordHistory DailyCnsStatus WorkoutPlan WorkoutSessionLog; do
  count=$(aws dynamodb scan --table-name $table --region ap-southeast-1 --select COUNT --query "Count" --output text)
  echo "$table: $count"
done
```

![Số lượng record trong 8 bảng sau khi seed data](/images/5-Workshop/5.6-Nguoi4-Admin/04-dynamodb-record-counts.png)

> **CLI Note:** toàn bộ lệnh `aws dynamodb` trên trang này đều gọi trực tiếp vào account `345798130457`, region `ap-southeast-1` — không phải sandbox. 8 bảng, 5 GSI, và dữ liệu seed đã tồn tại từ trước khi viết phần này.
