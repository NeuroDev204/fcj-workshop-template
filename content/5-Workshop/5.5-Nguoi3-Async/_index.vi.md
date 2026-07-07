---
title : "Lê Hữu Duy Hoàng - Tiến trình & Async Notification"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

Đây là vai trò của chính tôi trong dự án **PeriodIQ**. Trách nhiệm của tôi là xử lý toàn bộ luồng dữ liệu **Bất đồng bộ (Asynchronous)** và **Theo dõi tiến trình tập luyện (Progression Tracking)** của người dùng, từ lúc họ hoàn thành buổi tập cho đến khi hệ thống gửi thông báo.

Dưới đây là 3 dịch vụ AWS cốt lõi mà tôi đã thiết lập và tích hợp:

| Dịch vụ AWS | Vai trò |
|---|---|
| **Amazon SQS** | Đóng vai trò là hàng đợi (Message Queue) để tiếp nhận các tác vụ nặng (như tạo giáo án xong cần gửi mail) nhằm giải phóng API phản hồi ngay lập tức cho User. |
| **AWS Lambda (Worker)** | Function chạy ngầm, tự động được Trigger mỗi khi có tin nhắn mới rơi vào SQS. Chịu trách nhiệm xử lý logic hậu kỳ. |
| **Amazon SES** | Dịch vụ gửi Email tự động. Lambda Worker sẽ gọi SES để gửi nội dung giáo án 4 tuần hoặc nhắc nhở lịch tập cho người dùng. |

Theo sự phân công, tôi tập trung mạnh vào phát triển **Backend (.NET 10)** cho các luồng xử lý này:
- **SQS Queue & Lambda Worker:** Viết `SqsMessageQueueService` để API đẩy tin nhắn, và `SqsHandler` để Lambda đọc tin nhắn rồi gọi AWS SES gửi email.
- **Log buổi tập (Workout Session):** Xây dựng `WorkoutSessionLogsController` để ghi nhận kết quả tập thực tế của User từ phòng gym.
- **Tính toán XP & Streak (Gamification):** Phát triển logic cộng điểm kinh nghiệm (XP) và tính toán chuỗi ngày tập liên tiếp (Streak) ngay khi User log buổi tập thành công.
- **DynamoDB Progress:** Cấu hình và tương tác với bảng DynamoDB (lưu trữ Progress) để cập nhật liên tục tiến trình của User.

#### Nội dung chi tiết các phần Backend đã thực hiện:

1. [Kiến trúc Bất đồng bộ với SQS & Lambda Worker](5.5.1-sqs-worker/)
2. [Thiết kế API Log Buổi Tập & Hệ thống Gamification (XP/Streak)](5.5.2-gamification/)
