---
title : "Kiến trúc Bất đồng bộ với SQS, Lambda & SES"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

Trong thiết kế hệ thống phần mềm hiện đại, đặc biệt là kiến trúc Microservices và Serverless, một trong những "anti-pattern" phổ biến nhất là thực hiện các tác vụ tốn thời gian (như tạo file PDF, xử lý hình ảnh, hoặc giao tiếp với dịch vụ gửi Email bên thứ ba) ngay bên trong luồng xử lý đồng bộ (synchronous) của API. Việc này không chỉ chặn luồng (block thread) của server mà còn làm tăng độ trễ (latency) của toàn bộ request, dẫn đến trải nghiệm người dùng kém vì họ phải nhìn biểu tượng "loading" quay vòng vòng.

Để giải quyết bài toán này cho dự án PeriodIQ, tôi đã ứng dụng **Kiến trúc Hướng sự kiện (Event-Driven Architecture)**, sử dụng bộ ba dịch vụ AWS là **Amazon SQS**, **AWS Lambda**, và **Amazon SES** nhằm tách rời hoàn toàn (decouple) tiến trình gửi email thông báo và giáo án ra khỏi API chính.

### 1. Khởi tạo và đẩy Message vào hàng đợi (Amazon SQS)

**Amazon SQS (Simple Queue Service)** đóng vai trò là một bộ đệm (buffer) cực kỳ bền bỉ. Khi người dùng hoàn tất việc thiết lập thông số và bấm "Tạo giáo án" trên Frontend (đồng thời phải bật tùy chọn nhận Email thông báo), API Backend sẽ đảm nhận việc phân tích dữ liệu và ghi giáo án vào DynamoDB. 

![Cài đặt thông báo qua Email](/images/5-Workshop/5.5-Nguoi3-Async/08-notification-setting.jpg)

Thay vì gửi email ngay lập tức, API chỉ đơn giản là đóng gói một sự kiện (ví dụ: `WorkoutPlanCreatedEvent` chứa `UserId` và `PlanId`) và đẩy nó vào SQS Queue.

Quá trình cấu hình Queue trên AWS Console được tôi thực hiện cẩn thận để đảm bảo tính sẵn sàng cao:

![Khởi tạo SQS Queue](/images/5-Workshop/5.5-Nguoi3-Async/01-sqs-create-queue.jpg)

Dưới đây là danh sách các Queue SQS mà tôi đã thiết lập để tiếp nhận các loại message khác nhau của hệ thống:

![Danh sách SQS Queue](/images/5-Workshop/5.5-Nguoi3-Async/02-sqs-list-queues.jpg)

Ở phía tầng Application (Backend .NET), tôi triển khai một Interface `IMessageQueueService` để đóng gói logic tương tác với AWS SDK. Đoạn code dưới đây minh họa cách API đẩy dữ liệu vào Queue chỉ trong vài mili-giây:

```csharp
public async Task SendMessageAsync<T>(T message)
{
    var jsonMessage = JsonSerializer.Serialize(message);
    var request = new SendMessageRequest
    {
        QueueUrl = _queueUrl,
        MessageBody = jsonMessage
    };
    await _sqsClient.SendMessageAsync(request);
}
```
Nhờ cơ chế này, API lập tức trả về HTTP Status `200 OK` cho Frontend. Người dùng có thể tiếp tục lướt web hoặc xem dashboard ngay lập tức mà không hề nhận ra tiến trình gửi email vẫn đang được xử lý ngầm ở phía sau.

### 2. Kích hoạt Lambda Worker xử lý ngầm

Bản thân SQS chỉ là nơi chứa tin nhắn. Để "tiêu thụ" (consume) những tin nhắn này, tôi xây dựng một **AWS Lambda Function** đóng vai trò là một Worker độc lập.

Sức mạnh thực sự của kiến trúc Serverless thể hiện ở chỗ: Tôi không cần phải thuê một máy chủ EC2 chạy 24/7 chỉ để chờ đọc tin nhắn từ SQS. Thay vào đó, tôi cấu hình **SQS Event Source Mapping** (SQS Trigger) gắn trực tiếp vào Lambda. Mỗi khi có tin nhắn mới rơi vào hàng đợi, hệ thống AWS sẽ tự động đánh thức (invoke) Lambda Worker để xử lý. Nếu có hàng ngàn tin nhắn đổ về cùng lúc, Lambda sẽ tự động scale-out (nhân bản) lên hàng ngàn instance song song để giải quyết gọn gàng hàng đợi.

Chi tiết cấu hình SQS và các thông số Trigger (như Batch Size) được thể hiện tại đây:

![Cấu hình Lambda Trigger từ SQS](/images/5-Workshop/5.5-Nguoi3-Async/13-lambda-sqs-trigger.jpg)
![Chi tiết cấu hình SQS](/images/5-Workshop/5.5-Nguoi3-Async/03-sqs-queue-detail.jpg)

Code xử lý của Lambda Worker (viết bằng C#) sẽ lặp qua từng bản ghi trong SQS, giải mã JSON, và truy xuất cơ sở dữ liệu DynamoDB để lấy địa chỉ Email thực tế của User:

```csharp
public async Task FunctionHandler(SQSEvent evnt, ILambdaContext context)
{
    foreach (var message in evnt.Records)
    {
        // 1. Parse dữ liệu JSON từ SQS Message Body
        var eventData = JsonSerializer.Deserialize<WorkoutPlanEvent>(message.Body);
        
        // 2. Lấy thông tin chi tiết (Email, Tên) từ DynamoDB
        var user = await _dynamoContext.LoadAsync<UserProfile>(eventData.UserId);
        
        // 3. Chuẩn bị gọi SES để gửi thư...
    }
}
```

### 3. Tích hợp Amazon SES để tự động gửi Email

Khâu cuối cùng trong chuỗi hành động này là giao tiếp với **Amazon SES (Simple Email Service)**. Đây là giải pháp gửi email quy mô lớn của AWS với khả năng chống spam mạnh mẽ và tỷ lệ gửi thành công (deliverability) cực cao.

Trước khi hệ thống có thể gửi email thay mặt PeriodIQ, tôi phải thực hiện quá trình xác thực danh tính (Domain/Email Verification) trên SES để tuân thủ các chính sách chống giả mạo email (DKIM, SPF):

![Khởi tạo SES Identity](/images/5-Workshop/5.5-Nguoi3-Async/04-ses-create-identity.jpg)
![Xác thực và cấu hình Email](/images/5-Workshop/5.5-Nguoi3-Async/05-ses-setup-email.jpg)

Sau khi thiết lập, tất cả các danh tính hợp lệ sẽ được AWS quản lý và cấp quyền hoạt động:

![Danh sách SES Identities đã xác thực](/images/5-Workshop/5.5-Nguoi3-Async/06-ses-list-identities.jpg)

Trở lại với Lambda Worker, sau khi đã có địa chỉ Email của User, Worker sẽ nạp một HTML Template chuyên nghiệp (chứa giao diện đẹp mắt của giáo án 4 tuần) và gửi yêu cầu thông qua SES SDK:

```csharp
var sendRequest = new SendEmailRequest
{
    Source = "noreply@periodiq.com", // Email đã được verified trên SES
    Destination = new Destination { ToAddresses = new List<string> { user.Email } },
    Message = new Message
    {
        Subject = new Content(" Giáo án 4 tuần của bạn đã sẵn sàng! - PeriodIQ"),
        Body = new Body { Html = new Content(htmlTemplate) }
    }
};
await _sesClient.SendEmailAsync(sendRequest);
```

### 4. Đảm bảo chất lượng với Unit Test

Để đảm bảo hệ thống SQS Handler hoạt động hoàn hảo và không xảy ra lỗi (regression) trong tương lai, tôi đã viết các Unit Test sử dụng xUnit và Moq. Quá trình test bao gồm việc giả lập (mock) SQS Event, DynamoDB Context và SES Client để kiểm tra toàn bộ luồng logic giải mã JSON và gọi API gửi email.

Kết quả toàn bộ các test cases đều **Passed** xuất sắc trong vòng 1.2s:

![Kết quả Unit Test SQS Handler](/images/5-Workshop/5.5-Nguoi3-Async/07-sqs-unit-tests-passed.jpg)

### Tổng kết
Với kiến trúc **API -> SQS -> Lambda -> SES**, PeriodIQ đạt được 3 lợi ích to lớn:
1. **Hiệu suất (Performance):** Trải nghiệm người dùng mượt mà, không bao giờ bị nghẽn (block) bởi các tác vụ mạng.
2. **Khả năng phục hồi (Fault Tolerance):** Nếu dịch vụ Email (SES) tạm thời gặp sự cố, Lambda sẽ báo lỗi và tin nhắn vẫn nằm an toàn trong SQS (hoặc đẩy vào Dead-Letter Queue). Không có bất kỳ dữ liệu nào bị mất.
3. **Chi phí (Cost-effective):** Kiến trúc hoàn toàn 100% Serverless, chỉ trả tiền cho mỗi tin nhắn SQS và mỗi mili-giây Lambda thực thi. Khi không có ai sử dụng, chi phí vận hành bằng $0.
