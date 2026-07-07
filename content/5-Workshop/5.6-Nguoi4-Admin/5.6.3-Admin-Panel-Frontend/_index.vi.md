---
title : "Admin Panel Frontend"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---

Phần thứ ba trong vai trò của tôi: Admin Panel React — 4 trang (Dashboard, Exercises, Rules, Templates) tiêu thụ các bảng DynamoDB và Lambda Admin API ở hai trang trước.

#### Admin Dashboard (`/admin`)

Gọi `GET /api/admin/stats` qua TanStack Query, hiển thị 8 metric dưới dạng stat card và shortcut navigation đến các trang con.

![Admin Dashboard — 8 stat card và navigation shortcuts](/images/5-Workshop/5.6-Nguoi4-Admin/06-admin-dashboard.png)

#### Trang quản lý bài tập (`/admin/exercises`)

Bảng danh sách với search theo tên/nhóm cơ/thiết bị, CRUD qua modal form.

![Admin Exercises — danh sách 14 bài tập, modal tạo mới](/images/5-Workshop/5.6-Nguoi4-Admin/07-admin-exercises.png)

#### Trang quản lý bộ luật (`/admin/rules`)

Bảng danh sách với badge Category, toggle switch IsActive gọi thẳng `PATCH /toggle`, CRUD qua modal có JSON editor cho `RuleParametersJson`.

![Admin Rules — toggle IsActive, JSON editor](/images/5-Workshop/5.6-Nguoi4-Admin/08-admin-rules.png)

#### Trang quản lý template giáo án (`/admin/templates`)

Form phức tạp nhất: thêm/xóa ngày tập động, mỗi ngày thêm/xóa bài tập với 3 chỉ số (TargetSets, TargetRepRange, TargetIntensityPercentage).

![Admin Templates — dynamic form thêm ngày và bài tập](/images/5-Workshop/5.6-Nguoi4-Admin/09-admin-templates.png)

#### Luồng dữ liệu từ DynamoDB đến UI

```
DynamoDB (ap-southeast-1)
    → Lambda (ASP.NET Core)
    → Axios interceptor (tự gắn JWT)
    → TanStack Query (cache + refetch)
    → React component
```

JWT được lấy tự động từ Cognito session hiện tại qua interceptor trong `axiosConfig.js` — không cần truyền token thủ công trong bất kỳ component nào.
