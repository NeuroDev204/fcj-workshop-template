---
title : "API Log Buổi Tập & Hệ thống Gamification"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

Để một ứng dụng Fitness (thể hình) có thể giữ chân người dùng lâu dài, việc chỉ cung cấp giáo án là chưa đủ. Người dùng cần có cảm giác thành tựu và được ghi nhận công sức sau mỗi giọt mồ hôi rơi xuống phòng tập. Đó là lý do tôi thiết kế và tích hợp trực tiếp **Hệ thống Gamification** (Trò chơi hóa) vào sâu bên trong luồng xử lý Log buổi tập của Backend.

Hệ thống này xoay quanh hai chỉ số cốt lõi:
- **XP (Kinh nghiệm):** Điểm thưởng tích lũy sau mỗi buổi tập hoàn thành.
- **Streak (Chuỗi ngày):** Số ngày tập luyện liên tiếp không bị ngắt quãng.

![Giao diện nhập kết quả buổi tập (Log Workout)](/images/5-Workshop/5.5-Nguoi3-Async/09-workout-logging.jpg)
![Chi tiết buổi tập (Phần 1)](/images/5-Workshop/5.5-Nguoi3-Async/11-workout-detail-1.jpg)
![Chi tiết buổi tập (Phần 2)](/images/5-Workshop/5.5-Nguoi3-Async/12-workout-detail-2.jpg)

### 1. Luồng xử lý tại API Log Buổi Tập

Khi người dùng hoàn tất bài tập cuối cùng và nhấn "Hoàn thành buổi tập" trên giao diện React, Frontend sẽ gửi một HTTP POST Request chứa Payload toàn diện (bao gồm danh sách bài tập, số Sets, Reps thực tế, khối lượng tạ, và mức độ RPE) về Backend.

Tại tầng Core của kiến trúc Clean Architecture, tiến trình này được tiếp nhận và xử lý bởi `WorkoutSessionLogService`. Tiến trình này không chỉ ghi nhận dữ liệu thô vào DynamoDB (để Rule Engine đo đạc độ mỏi hệ thần kinh trung ương - CNS) mà còn kích hoạt ngay lập tức thuật toán tính toán Gamification.

![Dashboard hiển thị XP và ngọn lửa Streak rực rỡ](/images/5-Workshop/5.5-Nguoi3-Async/10-gamification-dashboard.jpg)

### 2. Thuật toán xử lý Gamification tại Backend

Thay vì sử dụng cronjob tính toán chậm trễ, tôi quyết định cập nhật XP và Streak theo thời gian thực (Real-time). Dữ liệu này được lưu trữ trong Entity `Progress.cs` của DynamoDB, bao gồm các trường chính như `XP`, `CurrentStreak`, `LongestStreak`, và `LastWorkoutDate`.

![Bảng Progress trong DynamoDB](/images/5-Workshop/5.5-Nguoi3-Async/14-dynamodb-progress-table.jpg)

Cơ chế tính toán Gamification hoạt động theo các quy tắc sau:
- **Tăng Streak:** Nếu người dùng tập liên tục (ngày tập gần nhất là hôm qua) hoặc đây là buổi tập đầu tiên, hệ thống sẽ cộng thêm 1 ngày vào chuỗi Streak hiện tại.
- **Phạt Reset Streak:** Nếu người dùng bỏ tập quá 1 ngày, chuỗi Streak hiện tại sẽ lập tức bị reset về 1 để rèn luyện tính kỷ luật.
- **Kỷ lục cá nhân:** Luôn so sánh và lưu lại chuỗi ngày tập dài nhất (Longest Streak) mà họ từng đạt được.
- **Cộng XP:** Bất kể Streak thế nào, mỗi buổi tập hoàn thành đều được cộng cứng 50 XP để khích lệ tinh thần.

Đoạn code trong `WorkoutSessionLogService.cs` thực thi quy tắc này một cách gọn gàng:

```csharp
var today = DateTime.UtcNow.Date;
var lastDate = progress.LastWorkoutDate?.Date;

if (lastDate == null || lastDate == today.AddDays(-1))
{
    progress.CurrentStreak += 1;
}
else if (lastDate < today.AddDays(-1))
{
    progress.CurrentStreak = 1;
}

if (progress.CurrentStreak > progress.LongestStreak)
{
    progress.LongestStreak = progress.CurrentStreak;
}

progress.XP += 50; 
progress.LastWorkoutDate = DateTime.UtcNow;

await _dynamoContext.SaveAsync(progress);
```

### 3. Tác động đến trải nghiệm người dùng

Nhờ kiến trúc xử lý liền mạch tại Backend, ngay khoảnh khắc API trả về HTTP Status `200 OK`, dữ liệu `Progress` mới nhất đã sẵn sàng. 

Khi Frontend tự động điều hướng người dùng về trang Dashboard, màn hình sẽ lập tức nhảy số XP mới, kèm theo biểu tượng ngọn lửa rực rỡ đại diện cho chuỗi Streak hiện tại. Phản hồi tức thời này (Immediate Feedback) tạo ra một cú hích Dopamine cực mạnh trong não bộ, khơi dậy cảm giác thỏa mãn và vô tình "gây nghiện", tạo động lực to lớn để họ xách balo lên và tiếp tục đi tập vào ngày hôm sau!
