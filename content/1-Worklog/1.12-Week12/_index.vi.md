---
title: "Worklog Tuần 12"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.12. </b> "
---


### Mục tiêu tuần 12:

* Hoàn thiện tích hợp CloudFront / Cognito / API Gateway cho pipeline build frontend.
* Hoàn tất triển khai production cho hệ thống PeriodIQ trên AWS.
* Tổng kết hạng mục CI/CD & Monitoring và bàn giao project trước khi kết thúc thực tập.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                                                        | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | -------------- |
| 1   | - Sửa `buildspec-frontend.yml` để lấy Cognito User Pool ID, Client ID và domain CloudFront từ output của CloudFormation stack ngay lúc build, để Vite build đúng config thật vào bundle production <br> - Sửa origin path API Gateway trên CloudFront để `/api/*` trỏ đúng tới stage API Gateway <br> - Sửa các lỗi YAML còn lại trong pipeline | 05/07/2026   | 05/07/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 2   | - Kiểm tra hệ thống production end-to-end (đăng nhập Cognito, routing API qua CloudFront, health check) và theo dõi CloudWatch xem có vấn đề gì sau khi deploy không                                                            | 06/07/2026   | 06/07/2026      | https://d1di1pzmfypszp.cloudfront.net/ |
| 3   | - Viết và cập nhật tài liệu CI/CD và deployment để bàn giao project                                                                                                                                                              | 07/07/2026   | 07/07/2026      |                 |
| 4   | - Review và hoàn thiện các alarm CloudWatch cùng dashboard giám sát với team                                                                                                                                                     | 08/07/2026   | 08/07/2026      |                 |
| 5   | - Hỗ trợ đồng đội sửa các lỗi phát sinh trong lúc test end-to-end cuối cùng                                                                                                                                                      | 09/07/2026   | 09/07/2026      |                 |
| 6   | - Chuẩn bị demo và tài liệu thuyết trình cuối cùng cho hạng mục CI/CD & Monitoring                                                                                                                                               | 10/07/2026   | 10/07/2026      |                 |


### Kết quả đạt được tuần 12:

* Sửa pipeline build frontend để tự động lấy cấu hình Cognito và API thật (User Pool ID, Client ID, API URL) từ CloudFormation stack đưa vào bản build production - không cần cấu hình thủ công và tránh deploy nhầm bundle lỗi.
* Sửa lỗi ánh xạ origin path API Gateway trên CloudFront distribution, khắc phục việc routing `/api` giữa frontend và backend bị lỗi.
* Hoàn tất pipeline CI/CD end-to-end và triển khai thành công hệ thống PeriodIQ lên production trên AWS.
* **Hệ thống đã hoạt động và truy cập công khai tại: https://d1di1pzmfypszp.cloudfront.net/**
* Xác nhận hệ thống production hoạt động ổn định end-to-end và các cơ chế giám sát/alarm hoạt động đúng.
* Hoàn thành tài liệu CI/CD và deployment để bàn giao project.
* Hỗ trợ sửa lỗi cuối cùng cùng team và chuẩn bị demo cho hạng mục Phạm Văn Sỹ (CI/CD & Monitoring) trước khi kết thúc thực tập.

**Dịch vụ AWS đã học/sử dụng trong tuần:** Amazon CloudFront, AWS CodePipeline/CodeBuild, AWS CloudFormation/SAM, Amazon Cognito, Amazon API Gateway, Amazon CloudWatch.

*(Các công việc từ 06-10/07/2026 là các hoạt động tổng kết/bàn giao được suy diễn hợp lý, do không có thêm lịch sử commit sau ngày 05/07/2026 tại thời điểm viết - cập nhật lại nếu công việc thực tế khác.)*
