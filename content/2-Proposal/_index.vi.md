---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# PeriodIQ - Serverless Periodization Engine
## Hệ thống tự động sinh giáo án tập gym khoa học trên nền tảng AWS Serverless

### 1. Tóm tắt điều hành
PeriodIQ là một nền tảng web serverless hoạt động như một "huấn luyện viên tự động": người tập gym nhập các chỉ số cá nhân (cân nặng, trình độ, mục tiêu tập luyện, số ngày tập/tuần khả dụng), và một Rule Engine chạy trên AWS Lambda sẽ sinh ra giáo án 4 tuần theo chu kỳ khoa học (periodization) với trọng lượng tạ cụ thể (kg) cho từng bài tập. Thay vì dùng một mẫu giáo án chung chung tìm trên mạng, người dùng nhận được giáo án phù hợp với năng lực tập luyện thực tế, có theo dõi Personal Record (PR), và tự điều chỉnh qua từng tuần — bao gồm cả tuần giảm tải (deload) tự động để giảm nguy cơ chấn thương do kiệt sức thần kinh (CNS). Hệ thống được xây dựng hoàn toàn trên các dịch vụ AWS serverless (16 dịch vụ trải trên 8 tầng kiến trúc) và được phát triển, triển khai bởi một team 5 người, mỗi người phụ trách một nhóm dịch vụ riêng.

### 2. Tuyên bố vấn đề

**Vấn đề hiện tại**
Phần lớn người tập gym dựa vào các mẫu giáo án tĩnh, chung chung (tìm trên mạng, trong sách, hoặc từ huấn luyện viên) mà không tính đến trình độ cá nhân, sức mạnh thực tế (1RM/PR), khả năng hồi phục, hay số ngày tập khả dụng. Việc tăng tải dần (progressive overload) hiếm khi được theo dõi có hệ thống — người tập hoặc tập quá nhẹ (không tiến bộ) hoặc tập quá nặng (xếp liên tiếp các bài tập gây kiệt sức thần kinh cao, dễ chấn thương và burnout). Tự xây dựng một giáo án theo chu kỳ và điều chỉnh từng tuần đòi hỏi kiến thức chuyên môn mà hầu hết người mới và người tập trung cấp không có, trong khi các ứng dụng/coach chuyên nghiệp thường tốn kém và phức tạp.

**Giải pháp**
PeriodIQ tự động hoá việc lập giáo án theo chu kỳ dựa trên khoa học thông qua Rule Engine 3 bước:
1. **Volume Filter** — giới hạn tổng volume tập luyện mỗi tuần (số set) cho từng nhóm cơ, tránh tập quá tải một nhóm cơ.
2. **Conflict Resolution** — tránh xếp 2 bài tập có mức kiệt sức thần kinh (CNS) cao trong cùng một ngày, phân bổ đều stress thần kinh trong tuần.
3. **Progression Builder** — tăng tạ/số rep dần qua từng tuần dựa trên Personal Record và trình độ người dùng, và tự động chèn tuần deload (giảm volume) vào cuối chu kỳ 4 tuần.

Người dùng cũng ghi nhận tình trạng CNS hàng ngày (giấc ngủ, mức stress, đau cơ) và kết quả tập luyện thực tế (set/rep/kg/RPE), dữ liệu này được đưa ngược vào Rule Engine để giữ cho các giáo án sau luôn thực tế và an toàn. Admin quản lý thư viện bài tập, mẫu giáo án và tham số của các quy tắc thông qua một trang quản trị riêng.

**Lợi ích và giá trị mang lại**
- Giáo án cá nhân hoá, dựa trên cơ sở khoa học mà không cần huấn luyện viên thật.
- Tự động tăng tải và chèn tuần deload, giảm nguy cơ chấn thương do quá tải thần kinh.
- Thông báo email/nhắc lịch bất đồng bộ giúp giữ chân người dùng mà không làm chậm luồng xử lý chính.
- Hoàn toàn serverless (trả tiền theo mức dùng): không tốn chi phí hạ tầng nhàn rỗi, tự động scale theo lượng dùng, chi phí vận hành rất thấp với lượng người dùng nhỏ.
- Đồng thời là một project học tập thực hành cho cả team: 5 thành viên cùng xây dựng và vận hành một phần thật của kiến trúc AWS gồm 16 dịch vụ, bao gồm cả pipeline CI/CD và triển khai production thật.

### 3. Kiến trúc giải pháp

Hệ thống được triển khai tại region AWS `ap-southeast-1`, chia thành 8 tầng: Edge (WAF + CloudFront + S3), Auth (Cognito), API (API Gateway), Core Engine (3 Lambda function), Data (DynamoDB), Async/Messaging (SQS + Lambda Worker + SNS/SES), Monitoring (CloudWatch) và CI/CD (CodePipeline + CodeBuild + CloudFormation/SAM).

![Kiến trúc AWS của PeriodIQ](/images/2-Proposal/aws_architecture.png)

**Luồng xử lý chính (Tạo giáo án):**
1. User/Admin truy cập ứng dụng (web + API) qua **CloudFront**, được bảo vệ ở tầng edge bởi **AWS WAF** (chặn bot/SQLi/XSS và giới hạn rate theo IP).
2. CloudFront phục vụ React SPA tĩnh từ **S3**, và định tuyến `/api/*` tới **API Gateway (HTTP API)**.
3. API Gateway xác minh JWT của request thông qua Authorizer **Cognito**.
4. API Gateway invoke Lambda tương ứng: **API Handler** (request của user), **Rule Engine** (sinh giáo án), hoặc **Admin API** (thao tác quản trị).
5. Rule Engine đọc profile người dùng, Personal Record, các lần check-in CNS gần nhất và các rule đang active, chạy tuần tự Volume Filter -> Conflict Resolution -> Progression Builder, rồi lưu giáo án 4 tuần vào **DynamoDB** (capacity on-demand + TTL cache).
6. API Handler đẩy job gửi email vào **Amazon SQS** để trả response ngay cho user mà không cần chờ gửi email.
7. **Lambda Worker**, được trigger từ SQS, gửi email giáo án / nhắc lịch tập qua **Amazon SNS/SES**.
8. **CloudWatch** thu thập Logs, Metrics và Alarms trên tầng Core Engine và Async để giám sát hệ thống.
9. Developer push code lên GitHub, kích hoạt **tầng CI/CD**: **CodePipeline** điều phối **CodeBuild** (build & test) và deploy hạ tầng qua **CloudFormation/SAM**.

### Dịch vụ AWS sử dụng (16 dịch vụ)
| # | Dịch vụ | Tầng | Vai trò |
|---|---------|------|---------|
| 1 | AWS WAF | Edge | Tường lửa ứng dụng web (WebACL, scope CLOUDFRONT) gắn vào CloudFront distribution |
| 2 | Amazon CloudFront | Edge | CDN và định tuyến, được WAF bảo vệ |
| 3 | Amazon S3 | Edge | Hosting React SPA (frontend tĩnh) |
| 4 | Amazon Cognito | Auth & Security | Xác thực User/Admin, cấp JWT |
| 5 | Amazon API Gateway (HTTP) | API | Nhận request, xác minh JWT |
| 6 | AWS Lambda - API Handler | Core Engine | Xử lý request từ user |
| 7 | AWS Lambda - Rule Engine | Core Engine | Sinh giáo án 4 tuần |
| 8 | AWS Lambda - Admin API | Core Engine | Xử lý thao tác admin |
| 9 | Amazon DynamoDB | Data | Lưu trữ + TTL cache (8 bảng, billing on-demand) |
| 10 | Amazon SQS | Async | Hàng đợi tác vụ bất đồng bộ |
| 11 | AWS Lambda - Worker | Async | Xử lý message từ SQS |
| 12 | Amazon SNS / SES | Async | Gửi email và thông báo |
| 13 | Amazon CloudWatch | Monitoring | Logs, Metrics, Alarms |
| 14 | AWS CodePipeline | CI/CD | Điều phối pipeline triển khai |
| 15 | AWS CodeBuild | CI/CD | Build và test mã nguồn |
| 16 | AWS CloudFormation / SAM | CI/CD | Infrastructure as Code |

### Thiết kế thành phần
- **Edge & Frontend**: SPA React 19 + Vite, hosting trên S3, phục vụ qua CloudFront (dùng Origin Access Control) và được bảo vệ bởi các managed rule của AWS WAF (Common Rule Set + Bot Control) cùng rate limit theo IP.
- **Auth**: Cognito User Pool với 2 group `Users` và `Admins`; backend .NET validate JWT từ Cognito và map claim `sub` với `UserProfile.Id`.
- **Core Engine**: Xây dựng theo chiến lược "Monolithic Lambda" — chỉ một project ASP.NET Core Web API (`PeriodIQ.Api`) duy nhất, host qua `Amazon.Lambda.AspNetCoreServer.Hosting`, theo Clean Architecture nên logic Rule Engine và Worker nằm trong các service dùng chung ở `PeriodIQ.Core`, có thể tách thành Lambda riêng sau này mà không cần viết lại logic nghiệp vụ.
- **Data**: 8 bảng DynamoDB (master data, hồ sơ/theo dõi người dùng, giáo án đã sinh) dùng billing `PAY_PER_REQUEST`, có GSI cho các bảng cần query phức tạp.
- **Async/Messaging**: SQS tách các tác vụ thông báo chậm (email giáo án, nhắc lịch tập, chúc mừng PR) khỏi luồng request/response chính; Lambda Worker tiêu thụ hàng đợi và gửi qua SNS/SES.
- **Monitoring**: CloudWatch Logs, metric filter tuỳ chỉnh (thời gian chạy Rule Engine, số giáo án được tạo, số lỗi), và Alarm cho tỷ lệ lỗi/thời gian chạy Lambda, throttling của DynamoDB, giới hạn concurrency.
- **CI/CD**: CodePipeline (kích hoạt bởi GitHub webhook qua CodeStar Connections) chạy `buildspec-backend.yml` (restore/build/test/package qua SAM) và `buildspec-frontend.yml` (install/security scan/build/deploy lên S3 + invalidate CloudFront).

### 4. Triển khai kỹ thuật

**Cơ cấu team** — project được chia cho 5 thành viên, mỗi người phụ trách trọn vẹn 3-4 dịch vụ AWS (cả backend + frontend):
- **Lê Hoài Huân - Auth & User Profile**: Amazon Cognito, AWS WAF, Amazon CloudFront.
- **Trần Anh Tài - Rule Engine & Sinh giáo án**: AWS Lambda (API Handler + Rule Engine), Amazon S3 (hosting frontend).
- **Lê Hữu Duy Hoàng - Tiến trình & Async Notification**: Amazon SQS, Lambda Worker, Amazon SNS/SES.
- **Chương Tử Luân - Admin Panel & Data**: Lambda Admin API, Amazon DynamoDB, API Gateway.
- **Phạm Văn Sỹ (vai trò của tôi) - CI/CD & Monitoring**: AWS CodePipeline, AWS CodeBuild, AWS CloudFormation/SAM, Amazon CloudWatch.

**Tech Stack**
- Backend: .NET 10 (nâng cấp từ .NET 9 giữa project), ASP.NET Core Web API, Clean Architecture (5 project), Amazon DynamoDB, Amazon SQS, Serilog, Swagger/Scalar, AWS SAM/CloudFormation.
- Frontend: React 19, Vite 8, Tailwind CSS 4 + Mantine 9, TanStack React Query 5, Axios, React Router 7.

**Các giai đoạn triển khai**
1. Thiết kế kiến trúc và phân chia vai trò trong team (các tuần lập kế hoạch).
2. Thiết kế, review và chỉnh sửa sơ đồ kiến trúc AWS theo góp ý của team.
3. Triển khai backend và hạ tầng (Rule Engine, controller, schema DynamoDB, thiết lập IAM).
4. Xây dựng pipeline CI/CD (SAM template, buildspec, CodePipeline), dashboard giám sát, và triển khai production.

### 5. Lộ trình & Mốc triển khai
- **Tuần 1-5**: Chương trình AWS Study Group (học cá nhân, các lab AWS nền tảng).
- **Tuần 6**: Họp khởi động team - xem xét kiến trúc, phân chia vai trò theo nhóm dịch vụ, thiết lập quyền truy cập AWS chung cho cả team (IAM Group/User/access key).
- **Tuần 7-9**: Thiết kế và chỉnh sửa sơ đồ kiến trúc AWS theo góp ý của team; bắt đầu triển khai backend.
- **Tuần 10**: Xây dựng nền tảng CI/CD - SAM template, `samconfig.toml`, controller kiểm tra health, buildspec cho backend/frontend.
- **Tuần 11**: Xây dựng và ổn định pipeline CodePipeline/CodeBuild; nâng cấp backend lên .NET 10; xây dựng dashboard giám sát CI/CD.
- **Tuần 12**: Hoàn thiện tích hợp CloudFront/Cognito/API Gateway và hoàn tất triển khai production.

### 6. Ước tính ngân sách
PeriodIQ chạy trên kiến trúc serverless trả tiền theo mức dùng hoàn toàn, không tốn chi phí hạ tầng nhàn rỗi. Với quy mô triển khai nhỏ (một số ít người dùng, lượng request thấp), các yếu tố chi phí chính gồm:
- **AWS Lambda**: tính phí theo số lần gọi + thời gian chạy; không đáng kể ở lượng traffic thấp (phần lớn nằm trong Free Tier 1 triệu request/tháng).
- **Amazon DynamoDB**: billing `PAY_PER_REQUEST` trên 8 bảng - chi phí tỷ lệ theo số lần đọc/ghi thực tế, không phải theo capacity đặt trước.
- **Amazon API Gateway (HTTP API)**: tính phí theo request; loại API Gateway rẻ nhất hiện có.
- **Amazon CloudFront + S3**: chi phí tối thiểu với bundle SPA nhỏ và lượng request thấp.
- **Amazon SQS / SNS / SES**: không đáng kể ở lượng message thấp (đều có free tier hào phóng).
- **AWS CodePipeline / CodeBuild**: chi phí cố định nhỏ cho mỗi pipeline hoạt động cộng với số phút compute của CodeBuild mỗi lần build.

*(Không đính kèm con số ước tính cố định từ AWS Pricing Calculator vì chi phí thực tế phụ thuộc vào lượng truy cập cuối cùng; kiến trúc được chọn có chủ đích để giữ chi phí biên gần như bằng 0 ở quy mô nhỏ.)*

### 7. Đánh giá rủi ro
- **Tính sai CNS/volume dẫn đến giáo án không an toàn** (ảnh hưởng trung bình, xác suất thấp) - giảm thiểu bằng rule Volume Filter và Conflict Resolution, cùng cơ chế check-in CNS hàng ngày có thể tự động kích hoạt deload.
- **Lambda cold start / lỗi custom runtime** (ảnh hưởng trung bình, xác suất trung bình, đặc biệt sau khi nâng cấp .NET 10) - giảm thiểu bằng cách pin phiên bản SDK, sửa bootstrap assembly name cho custom runtime, và kiểm thử tự động qua pipeline CI/CD.
- **Pipeline CI/CD bị drift** (ví dụ stack mất đồng bộ với resource đã deploy thực tế) - giảm thiểu bằng cách coi CloudFormation/SAM là nguồn chân lý duy nhất và deploy lại (xoá + deploy lại) khi phát hiện drift.
- **Tích hợp frontend/backend bị lỗi sau khi deploy** (ví dụ sai origin path CloudFront, config Cognito cũ bị bake cứng vào bundle frontend) - giảm thiểu bằng cách lấy config Cognito/API thật từ output của CloudFormation stack ngay lúc build, thay vì hardcode.

### 8. Kết quả kỳ vọng
- Một ứng dụng serverless hoàn chỉnh, đã triển khai công khai, sinh giáo án tập luyện cá nhân hoá dựa trên khoa học - đang hoạt động tại **https://d1di1pzmfypszp.cloudfront.net/**.
- Một kiến trúc tham chiếu và pipeline CI/CD có thể tái sử dụng cho các project serverless .NET-on-Lambda trong tương lai.
- Kinh nghiệm thực hành end-to-end trên toàn bộ stack dịch vụ AWS (auth, API, compute, data, messaging, monitoring, CI/CD) cho cả 5 thành viên trong team.
