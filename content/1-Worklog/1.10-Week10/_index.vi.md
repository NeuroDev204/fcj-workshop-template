---
title: "Worklog Tuần 10"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---


### Mục tiêu tuần 10:

* Ở vai trò Phạm Văn Sỹ (CI/CD & Monitoring): xây dựng nền tảng CI/CD và Infrastructure-as-Code ban đầu cho PeriodIQ - SAM template, samconfig, controller kiểm tra health, và buildspec cho backend/frontend.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                     | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | -------------- |
| 4   | - Tạo SAM template (`template.yml`) và `samconfig.toml` <br> - Xây dựng endpoint kiểm tra health `HealthController.cs` <br> - Tạo `buildspec-backend.yml` cho stage CodeBuild backend | 25/06/2026   | 25/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 5   | - Thêm `frontend/.env.example` <br> - Sửa lỗi cú pháp trong `buildspec-backend.yml` <br> - Tạo `buildspec-frontend.yml` (test thành công)                                            | 26/06/2026   | 26/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 6   | - Cập nhật `buildspec-backend.yml`: thêm test report & CodeBuild report, artifact versioning <br> - Cập nhật `buildspec-frontend.yml`: thêm bước security scan                        | 27/06/2026   | 27/06/2026      | https://github.com/PeriolIQ/PeriodIQ |


### Kết quả đạt được tuần 10:

* Xây dựng SAM template (`template.yml`) và `samconfig.toml`, định nghĩa toàn bộ hạ tầng serverless của PeriodIQ dưới dạng Infrastructure as Code (Lambda, HTTP API, DynamoDB, S3, SQS, SNS).
* Xây dựng endpoint health-check `HealthController`, dùng sau này cho pipeline smoke test và giám sát uptime.
* Tạo `buildspec-backend.yml` (restore/build/test/package) và `buildspec-frontend.yml` (install/build/deploy), kiểm thử thành công từng phase build.
* Bổ sung test report và CodeBuild report, artifact versioning cho backend build; thêm bước security scan cho frontend build.

**Dịch vụ AWS đã học/sử dụng trong tuần:** AWS CloudFormation / AWS SAM, AWS CodeBuild, Amazon CloudWatch (health check).
