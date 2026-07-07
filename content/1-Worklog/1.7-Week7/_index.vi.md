---
title: "Worklog Tuần 7"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---


### Mục tiêu tuần 7:

* Bắt đầu các bước triển khai ban đầu cho project PeriodIQ ở vai trò Phạm Văn Sỹ (CI/CD & Monitoring).
* Lên kế hoạch các resource CloudFormation/SAM và hình dạng buildspec cho backend/frontend.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | -------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | -------------- |
| 2   | - Xem xét pricing và giới hạn dịch vụ của AWS CodeBuild và Amazon CloudWatch để lên kế hoạch giám sát                            | 01/06/2026   | 01/06/2026      |                 |
| 3   | - Phác thảo danh sách resource CloudFormation/SAM ban đầu cần có (Lambda function, HTTP API, S3 bucket, IAM role)                 | 02/06/2026   | 02/06/2026      |                 |
| 4   | - Viết bản nháp đầu tiên cho `buildspec-backend.yml` và `buildspec-frontend.yml` (các phase install/pre_build/build/post_build)   | 03/06/2026   | 03/06/2026      |                 |
| 5   | - Test AWS SAM CLI ở local (`sam init` / `sam build`) với một function mẫu để xác nhận quy trình                                  | 04/06/2026   | 04/06/2026      |                 |
| 6   | - Làm việc trực tiếp tại văn phòng công ty <br> - Trao đổi với team về kế hoạch triển khai tổng thể                              | 05/06/2026   | 05/06/2026      |                 |


### Kết quả đạt được tuần 7:

* Xem xét pricing/giới hạn của AWS CodeBuild và CloudWatch để lên kế hoạch giám sát cho project.
* Phác thảo danh sách resource SAM ban đầu cho hạ tầng PeriodIQ.
* Phác thảo cấu trúc buildspec cho backend và frontend.
* Xác nhận quy trình AWS SAM CLI ở local với một function mẫu.
* Trao đổi với team tại văn phòng về kế hoạch triển khai tổng thể.
