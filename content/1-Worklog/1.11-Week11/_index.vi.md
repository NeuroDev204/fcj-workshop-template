---
title: "Worklog Tuần 11"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---


### Mục tiêu tuần 11:

* Xây dựng và ổn định pipeline CI/CD (AWS CodePipeline + CodeBuild) cho PeriodIQ.
* Nâng cấp backend lên .NET 10 và sửa các lỗi Lambda custom runtime phát sinh.
* Xây dựng dashboard giám sát CI/CD ở phía frontend.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                                    | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | -------------- |
| 1   | - Tạo file định nghĩa pipeline CI/CD `cicd-pipeline.yml` (AWS CodePipeline điều phối các stage CodeBuild backend/frontend)                                                                                   | 28/06/2026   | 28/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 2   | - Viết và cập nhật tài liệu README của project                                                                                                                                                                | 29/06/2026   | 29/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 3   | - Cập nhật `template.yml` để sửa các lỗi hạ tầng                                                                                                                                                              | 30/06/2026   | 30/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 4   | - Kết nối repo GitHub qua AWS CodeStar Connections và chạy pipeline <br> - Nâng cấp backend lên .NET 10 (sửa đường dẫn cài SDK, pin `global.json`, sửa bootstrap assembly name cho custom runtime) <br> - Chuyển `buildspec-frontend.yml` sang dùng pnpm <br> - Cập nhật dashboard admin | 01/07/2026   | 01/07/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 5   | - Sửa lỗi gọi API "deploy history" ở trang admin <br> - Thiết kế lại dashboard giám sát <br> - Hiển thị 4 giai đoạn build của pipeline kèm log trực tiếp trong lúc build <br> - Thêm biểu đồ vào trang monitoring và sửa lỗi crash Lambda | 02/07/2026   | 02/07/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 7   | - Thêm biểu đồ giám sát và cập nhật README cho Lê Hoài Huân <br> - Sửa các unit test bị lỗi <br> - Sửa cấu hình region cho AWS WAF WebACL                                                                          | 04/07/2026   | 04/07/2026      | https://github.com/PeriolIQ/PeriodIQ |


### Kết quả đạt được tuần 11:

* Xây dựng pipeline CI/CD CodePipeline/CodeBuild (`cicd-pipeline.yml`), điều phối trọn vẹn các stage build/deploy backend và frontend.
* Nâng cấp toàn bộ backend lên .NET 10, xử lý các vấn đề Lambda custom runtime (`provided.al2023`), cài đặt .NET 10 SDK và bootstrap assembly name.
* Kết nối repo GitHub với CodePipeline qua AWS CodeStar Connections và xác minh pipeline chạy thành công end-to-end.
* Xây dựng dashboard giám sát CI/CD ở frontend: hiển thị tiến trình build theo từng stage, log build trực tiếp và biểu đồ giám sát.
* Sửa các unit test bị lỗi và sửa cấu hình region của AWS WAF WebACL.

**Dịch vụ AWS đã học/sử dụng trong tuần:** AWS CodePipeline, AWS CodeBuild, AWS CodeStar Connections, AWS CloudFormation/SAM, AWS Lambda (custom runtime), AWS WAF, Amazon CloudWatch.
