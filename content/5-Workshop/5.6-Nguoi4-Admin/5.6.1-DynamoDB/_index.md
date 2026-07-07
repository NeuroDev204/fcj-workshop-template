---
title : "Amazon DynamoDB"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1. </b> "
---

The first of the three services in my role: full schema design, table creation, GSIs, and seed data for the 8 DynamoDB tables behind PeriodIQ.

#### List running tables

```bash
aws dynamodb list-tables --region ap-southeast-1
```

![8 DynamoDB tables running in ap-southeast-1](/images/5-Workshop/5.6-Nguoi4-Admin/01-dynamodb-list-tables.png)

9 tables are listed (including `Progress` required by Lê Hữu Duy Hoàng). The 8 tables within my design scope:

| Group | Tables | Purpose |
|---|---|---|
| Master data | `Exercise`, `WorkoutTemplate`, `RuleDefinition` | Admin-managed, read by Rule Engine |
| User profile | `UserProfile`, `PersonalRecordHistory`, `DailyCnsStatus` | User personal data |
| Generated plans | `WorkoutPlan`, `WorkoutSessionLog` | Generated plans and session logs |

#### Design principles

All 8 tables follow the same model:
- **Partition key**: `Id` (String) — UUID for all entities; `UserProfile` uses Cognito `sub` directly so it maps to the JWT claim without any lookup
- **Billing**: `PAY_PER_REQUEST` — no upfront throughput estimates, fits a startup's uneven workload
- **Base fields**: `Id`, `CreatedAt`, `UpdatedAt` — inherited from `BaseEntity` in the Domain layer

```bash
aws dynamodb describe-table --table-name Exercise --region ap-southeast-1 \
  --query "Table.{Keys:KeySchema,BillingMode:BillingModeSummary.BillingMode}"
```

![Exercise table — PAY_PER_REQUEST, partition key Id](/images/5-Workshop/5.6-Nguoi4-Admin/02-dynamodb-exercise-table.png)

#### GSIs on tables that need complex queries

5 of the 8 tables have Global Secondary Indexes to support querying by `UserId`:

```bash
aws dynamodb describe-table --table-name UserProfile --region ap-southeast-1 \
  --query "Table.GlobalSecondaryIndexes[*].{Index:IndexName,Key:KeySchema}"
```

![UserProfile GSI Email-index — query user by email](/images/5-Workshop/5.6-Nguoi4-Admin/03-dynamodb-userprofile-gsi.png)

| Table | GSI | Used for |
|---|---|---|
| `UserProfile` | `Email-index` (PK: Email) | Lookup user by email at login |
| `PersonalRecordHistory` | `UserId-ExerciseId-index` (PK: UserId, SK: ExerciseId) | Fetch all PRs for a user + exercise |
| `DailyCnsStatus` | `UserId-DateLog-index` (PK: UserId, SK: DateLog) | Fetch last 7 CNS days for a user |
| `WorkoutPlan` | `UserId-StartDate-index` (PK: UserId, SK: StartDate) | Fetch plans by user, sorted by date |
| `WorkoutSessionLog` | `UserId-CompletedAt-index` (PK: UserId, SK: CompletedAt) | Fetch session history sorted by time |

All GSIs use `UserId` as Partition Key because every application query starts with "get data for user X" — there is no end-user usecase that needs a full table scan.

#### Checking record counts per table

```bash
foreach ($table in @("Exercise","WorkoutTemplate","RuleDefinition","UserProfile","PersonalRecordHistory","DailyCnsStatus","WorkoutPlan","WorkoutSessionLog")) {
  $count = aws dynamodb scan --table-name $table --region ap-southeast-1 --select COUNT --query "Count" --output text
  Write-Host "$table`: $count"
}
```

![Record counts in all 8 tables after seeding](/images/5-Workshop/5.6-Nguoi4-Admin/04-dynamodb-record-counts.png)

> **CLI Note:** all `aws dynamodb` commands on this page call directly into account `345798130457`, region `ap-southeast-1` — not a sandbox. The 8 tables, 5 GSIs, and seed data existed before this section was written.
