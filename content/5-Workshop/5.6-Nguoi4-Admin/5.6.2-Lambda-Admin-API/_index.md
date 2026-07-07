---
title : "Lambda Admin API"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2. </b> "
---

The second service in my role: 4 backend controllers (Exercise, Template, Rule, AdminStats) and the unit tests covering them.

The backend uses Clean Architecture with 5 projects. The entire API (including admin routes) is packaged into **a single Lambda** via `Amazon.Lambda.AspNetCoreServer.Hosting` — controllers route requests internally.

#### ExercisesController — Exercise master data

Admin-only endpoints (requires Cognito group `Admin`):

| Method | Route | Description |
|---|---|---|
| GET | `/api/exercises` | List all exercises |
| GET | `/api/exercises/{id}` | Get exercise detail |
| POST | `/api/exercises` | Create new exercise |
| PUT | `/api/exercises/{id}` | Update exercise |
| DELETE | `/api/exercises/{id}` | Delete exercise |

#### WorkoutTemplatesController — Workout templates

Same CRUD as `ExercisesController` but the `WorkoutTemplate` entity has a nested structure: `Days` → `Exercises` (each with `TargetSets`, `TargetRepRange`, `TargetIntensityPercentage`).

#### RuleDefinitionsController — Rule Engine rules

In addition to standard CRUD, this controller has a toggle endpoint:

```
PATCH /api/ruledefinitions/{id}/toggle
```

This flips `IsActive` without sending the full object — suited for the admin use case of quickly enabling/disabling rules from the dashboard.

#### AdminStatsController — System statistics

3 read-only endpoints serving the Admin Dashboard:

| Endpoint | Returns |
|---|---|
| `GET /api/admin/stats` | 8 counters: TotalExercises, TotalWorkoutTemplates, TotalRules, ActiveRules, TotalUsers, TotalWorkoutPlans, ActiveWorkoutPlans, TotalSessionLogs |
| `GET /api/admin/stats/popular-exercises` | Top 10 most-used exercises in WorkoutPlans (aggregated from nested data) |
| `GET /api/admin/stats/user-activity` | Active users and total sessions per ISO week (last 8) and per month (last 6) |

#### Unit tests — 31 tests across 4 controllers

```bash
cd PeriodIQ.Backend
dotnet test PeriodIQ.Api.Tests --verbosity normal
```

![31 tests pass, 0 fail — xUnit + Moq](/images/5-Workshop/5.6-Nguoi4-Admin/05-unit-tests-pass.png)

| Test file | Tests | Covers |
|---|---|---|
| `ExercisesControllerTests` | 8 | GetAll, GetById (found/not found), Create, Update (match/mismatch), Delete |
| `WorkoutTemplatesControllerTests` | 8 | Same as above |
| `RuleDefinitionsControllerTests` | 10 | CRUD + 2 tests for Toggle endpoint |
| `AdminStatsControllerTests` | 5 | ReturnsOk, AllZeroes, CountsActiveRulesOnly, CountsActivePlansOnly, AllEightCounters |

One notable technical detail: `AdminStatsController` returns an **anonymous type** (`new { TotalExercises = ..., ... }`). When testing across assembly boundaries, `dynamic` cannot access anonymous type properties (runtime binder exception). Fixed using reflection:

```csharp
private static int Prop(object obj, string name) =>
    (int)obj.GetType().GetProperty(name)!.GetValue(obj)!;
```
