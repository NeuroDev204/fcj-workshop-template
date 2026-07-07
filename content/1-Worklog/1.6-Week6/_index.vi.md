---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---


### Mục tiêu tuần 6:

* Tham gia buổi họp khởi động team cho project cuối cùng (PeriodIQ) và cùng phân chia công việc giữa 5 thành viên theo nhóm dịch vụ AWS phụ trách.
* Bắt đầu làm quen với bộ công cụ AWS CodePipeline / CodeBuild / SAM cho hạng mục Phạm Văn Sỹ (CI/CD & Monitoring).

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                  | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | -------------- |
| 2   | - Họp team: xem xét kiến trúc hệ thống PeriodIQ (thiết kế serverless 7 tầng trên AWS) và chia project thành 5 nhóm vai trò, mỗi người phụ trách 3-4 dịch vụ AWS | 25/05/2026   | 25/05/2026      |                 |
| 3   | - Đọc tài liệu kỹ thuật của project (giải thích kiến trúc, hướng dẫn schema DynamoDB) để hiểu toàn bộ hệ thống trước khi lên phạm vi cho hạng mục CI/CD & Monitoring <br> - Thiết lập quyền truy cập AWS cho cả team: tạo IAM Group theo từng vai trò và IAM User riêng cho mỗi người trong 5 thành viên, gắn permission policy phù hợp với vai trò từng người, và tạo access key cho từng người | 26/05/2026   | 26/05/2026      |                 |
| 4   | - Tìm hiểu AWS CodePipeline, AWS CodeBuild và AWS SAM, cách chúng phù hợp với monorepo .NET Lambda + React frontend                            | 27/05/2026   | 27/05/2026      |                 |
| 5   | - Phác thảo luồng pipeline CI/CD ban đầu (source -> build -> test -> deploy) cho cả backend và frontend                                        | 28/05/2026   | 28/05/2026      |                 |
| 6   | - Thiết lập môi trường phát triển local (AWS CLI, SAM CLI, .NET SDK), clone repo và chạy thử build lần đầu                                     | 29/05/2026   | 29/05/2026      |                 |


### Kết quả đạt được tuần 6:

* Xem xét tổng thể kiến trúc PeriodIQ: Cognito/WAF/CloudFront (edge & auth), API Gateway, Lambda (API Handler/Rule Engine/Admin API), DynamoDB, SQS/SNS/SES, CloudWatch và CodePipeline/CodeBuild/SAM.
* Cùng phân chia team thành 5 vai trò theo nhóm dịch vụ AWS:
  * Lê Hoài Huân - Auth & User Profile: Amazon Cognito, AWS WAF, Amazon CloudFront.
  * Trần Anh Tài - Rule Engine & Sinh giáo án: AWS Lambda (API Handler + Rule Engine), Amazon S3 (hosting frontend).
  * Lê Hữu Duy Hoàng - Tiến trình & Async Notification: Amazon SQS, Lambda Worker, Amazon SNS/SES.
  * Chương Tử Luân - Admin Panel & Data: Lambda Admin API, Amazon DynamoDB, API Gateway.
  * **Phạm Văn Sỹ (vai trò của tôi) - CI/CD & Monitoring:** AWS CodePipeline, AWS CodeBuild, AWS CloudFormation/SAM, Amazon CloudWatch.
* Nhận vai trò Phạm Văn Sỹ: chịu trách nhiệm về Infrastructure as Code, pipeline build/deploy và giám sát hệ thống cho toàn bộ project.
* Nghiên cứu tài liệu project và bộ công cụ AWS CodePipeline/CodeBuild/SAM để chuẩn bị cho hạng mục CI/CD & Monitoring.
* Với vai trò Phạm Văn Sỹ, thiết lập quyền truy cập AWS chung cho cả team: tạo IAM Group theo từng vai trò và IAM User riêng cho mỗi người trong 5 thành viên, gắn permission policy giới hạn đúng theo các dịch vụ AWS mà từng người phụ trách, và tạo access key cho từng người để cả team có thể bắt đầu làm việc độc lập.
* Phác thảo luồng pipeline CI/CD ban đầu và thiết lập môi trường phát triển local (AWS CLI, SAM CLI, .NET SDK).

*(Chi tiết công việc đầy đủ của từng vai trò được ghi trong README của project - mục "Phân công theo thành viên".)*

**Kỹ năng đạt được:** Lên kế hoạch và phân chia công việc theo nhóm dịch vụ; hiểu toàn diện một kiến trúc tham chiếu serverless đầy đủ trên AWS; quản lý định danh IAM (Group, User, Policy, Access Key) cho tài khoản AWS dùng chung nhiều người; kiến thức cơ bản về AWS SAM CLI; thiết lập môi trường phát triển local cho .NET serverless.
