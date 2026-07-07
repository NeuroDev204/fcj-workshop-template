---
title : "Lambda Admin API"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2. </b> "
---

Dịch vụ thứ hai trong vai trò của tôi: 4 controller backend (Exercise, Template, Rule, AdminStats) và các unit test bao phủ chúng.

Backend dùng kiến trúc Clean Architecture với 5 project. Toàn bộ API (kể cả admin) đóng gói thành **một Lambda duy nhất** qua `Amazon.Lambda.AspNetCoreServer.Hosting` — các controller phân luồng request nội bộ.

#### ExercisesController — Master data bài tập

Endpoint admin-only (yêu cầu Cognito group `Admin`):

| Method | Route | Mô tả |
|---|---|---|
| GET | `/api/exercises` | Danh sách tất cả bài tập |
| GET | `/api/exercises/{id}` | Chi tiết 1 bài tập |
| POST | `/api/exercises` | Thêm bài tập mới |
| PUT | `/api/exercises/{id}` | Cập nhật bài tập |
| DELETE | `/api/exercises/{id}` | Xóa bài tập |

#### WorkoutTemplatesController — Mẫu giáo án

Tương tự `ExercisesController` nhưng entity `WorkoutTemplate` có cấu trúc nested phức tạp hơn: `Days` → `Exercises` (mỗi exercise có `TargetSets`, `TargetRepRange`, `TargetIntensityPercentage`).

#### RuleDefinitionsController — Bộ luật Rule Engine

Ngoài CRUD chuẩn, controller này có thêm endpoint toggle:

```
PATCH /api/ruledefinitions/{id}/toggle
```

Endpoint này lật trạng thái `IsActive` của một rule mà không cần gửi toàn bộ object — phù hợp với usecase admin bật/tắt rule nhanh từ dashboard.

#### AdminStatsController — Thống kê hệ thống

3 endpoint chỉ đọc, phục vụ Admin Dashboard:

| Endpoint | Trả về |
|---|---|
| `GET /api/admin/stats` | 8 counter: TotalExercises, TotalWorkoutTemplates, TotalRules, ActiveRules, TotalUsers, TotalWorkoutPlans, ActiveWorkoutPlans, TotalSessionLogs |
| `GET /api/admin/stats/popular-exercises` | Top 10 bài tập được dùng nhiều nhất trong WorkoutPlan (tổng hợp từ nested data) |
| `GET /api/admin/stats/user-activity` | Số user active và tổng buổi tập theo 8 tuần gần nhất và 6 tháng gần nhất (dùng ISO week) |

#### Unit test — 31 test cho 4 controller

```bash
cd PeriodIQ.Backend
dotnet test PeriodIQ.Api.Tests --verbosity normal
```

![31 test pass, 0 fail — xUnit + Moq](/images/5-Workshop/5.6-Nguoi4-Admin/05-unit-tests-pass.png)

| File test | Số test | Bao gồm |
|---|---|---|
| `ExercisesControllerTests` | 8 | GetAll, GetById (found/not found), Create, Update (match/mismatch), Delete |
| `WorkoutTemplatesControllerTests` | 8 | Như trên |
| `RuleDefinitionsControllerTests` | 10 | CRUD + 2 test cho Toggle endpoint |
| `AdminStatsControllerTests` | 5 | ReturnsOk, AllZeroes, CountsActiveRulesOnly, CountsActivePlansOnly, AllEightCounters |

Một điểm kỹ thuật đáng ghi nhận: `AdminStatsController` trả về **anonymous type** (`new { TotalExercises = ..., ... }`). Khi test qua assembly boundary, `dynamic` không truy cập được property của anonymous type (runtime binder exception). Giải pháp dùng reflection:

```csharp
private static int Prop(object obj, string name) =>
    (int)obj.GetType().GetProperty(name)!.GetValue(obj)!;
```
