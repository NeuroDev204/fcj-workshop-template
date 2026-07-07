---
title : "Admin Panel Frontend"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---

The third piece of my role: the React admin panel — 4 pages (Dashboard, Exercises, Rules, Templates) consuming the DynamoDB tables and Lambda Admin API from the previous two pages.

#### Admin Dashboard (`/admin`)

Calls `GET /api/admin/stats` via TanStack Query, displays 8 metrics as stat cards and shortcut navigation to sub-pages.

![Admin Dashboard — 8 stat cards and navigation shortcuts](/images/5-Workshop/5.6-Nguoi4-Admin/06-admin-dashboard.png)

#### Exercise management (`/admin/exercises`)

Table with search by name/muscle group/equipment, full CRUD via modal form.

![Admin Exercises — list of 14 exercises, create modal](/images/5-Workshop/5.6-Nguoi4-Admin/07-admin-exercises.png)

#### Rule management (`/admin/rules`)

Table with Category badges, IsActive toggle switch calling `PATCH /toggle` directly, CRUD modal with a JSON editor for `RuleParametersJson`.

![Admin Rules — toggle IsActive, JSON editor](/images/5-Workshop/5.6-Nguoi4-Admin/08-admin-rules.png)

#### Workout template management (`/admin/templates`)

The most complex form: dynamically add/remove training days, each day with dynamically add/remove exercises with 3 fields (TargetSets, TargetRepRange, TargetIntensityPercentage).

![Admin Templates — dynamic form for days and exercises](/images/5-Workshop/5.6-Nguoi4-Admin/09-admin-templates.png)

#### Data flow from DynamoDB to UI

```
DynamoDB (ap-southeast-1)
    → Lambda (ASP.NET Core)
    → Axios interceptor (auto-attach JWT)
    → TanStack Query (cache + refetch)
    → React component
```

The JWT is fetched automatically from the current Cognito session via the interceptor in `axiosConfig.js` — no manual token passing in any component.
