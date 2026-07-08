---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Scaling Serverless lên 1 Triệu Lambda Functions: Bài Học Kiến Trúc từ AWS

## Giấc Mơ vs. Hiện Thực ở Quy Mô Siêu Lớn

Chào anh em AWS Study Group! Hôm nay mình muốn chia sẻ một case study cực kỳ đắt giá từ AWS Architecture Blog, nói về bài toán vận hành và mở rộng hệ thống Serverless ở quy mô siêu lớn — cụ thể là một nền tảng SaaS đã scale lên tới **1 triệu Lambda functions chạy trên hàng ngàn tài khoản AWS**.

Điều mình thấy thú vị là ở quy mô nhỏ, Serverless luôn là một "giấc mơ màu hồng". Nhưng khi chạm mốc 1 triệu hàm, những lý thuyết tối ưu hay "Best Practices" thông thường mà chúng ta hay đọc đột nhiên bị bẻ gãy hoàn toàn, buộc đội ngũ kỹ sư phải đập đi xây lại tư duy kiến trúc.

## Bài Toán Ban Đầu: Kiến Trúc "Mỗi Khách Hàng Một Tài Khoản"

Để đạt được ranh giới bảo mật tuyệt đối, quản lý chi phí minh bạch và tránh triệt để tình trạng "hàng xóm ồn ào" (Noisy Neighbors), đội ngũ kỹ sư lựa chọn giải pháp tách mỗi Tenant (khách hàng) vào một tài khoản AWS riêng biệt.

Mô hình này cô lập cực tốt, nhưng khi hệ thống tăng trưởng lên hàng ngàn tài khoản với tổng số lượng hàm chạm mốc 1 triệu, họ lập tức va phải hàng loạt bài toán vận hành khốc liệt:

*(Quy trình "vending" tài khoản dùng để khởi tạo mỗi tài khoản tenant mới: tạo account, chuyển vào đúng OU, gắn tag, và điều chỉnh quota dịch vụ)*
![Account vending Step Functions workflow](/images/3-BlogsPosted/3.3-Blog3/01-account-vending-stepfunctions.png)

### Vấn đề 1: Self-DDoS (Tự DDoS Mạng Của Chính Mình)

Do sử dụng các tác vụ định kỳ (Cron Job) đồng bộ dạng fixed interval (5 phút) trên hàng ngàn tài khoản, tất cả các hàm Lambda sẽ đồng loạt kích hoạt vào chính xác giây đầu tiên của block 5 phút, tạo ra một đợt Spike Traffic khổng lồ tự quật ngã các API nội bộ.

### Vấn đề 2: Thuế Quan Sát (Observability Tax)

Chi phí Compute của Lambda không phải là gánh nặng lớn nhất. Cơn ác mộng thực sự xuất hiện khi hệ thống phải stream, tập trung dữ liệu Logs và Metrics từ hàng ngàn tài khoản về một Dashboard trung tâm, khiến **hóa đơn Observability tăng vọt gấp đôi tổng chi phí Cloud**.

### Vấn đề 3: Giới Hạn Công Cụ (Performance Ceiling)

AWS CloudFormation StackSets vốn là "vũ khí hạng nặng" để quản trị multi-account. Nhưng khi số lượng tài khoản và số hàm quá lớn, StackSets bắt đầu gặp hiện tượng throttling và sinh ra hàng loạt lỗi tích tụ ở quy mô lớn.

### Vấn đề 4: Lãng Phí Khi Scale-to-Zero

Dù Serverless hứa hẹn "không dùng không trả tiền", nhưng thực tế các chi phí ẩn (như CloudWatch Alarms cho mỗi hàm Lambda) vẫn âm thầm tích tiểu thành đại trên hàng ngàn tài khoản đang ở trạng thái rảnh rỗi.

## Giải Pháp: Tối Ưu Hóa Phân Tầng

Để giải quyết bài toán quy mô này, đội ngũ kỹ sư không thể tối ưu theo cách thông thường mà phải **tái thiết kế lại cách hệ thống giao tiếp và phân phối**.

### 1. Request Scattering và Jitter (Randomized Offsets)

Để xử lý triệt để trận chiến "Self-DDoS" do các Cron Job đồng loạt, hệ thống bắt buộc phải thêm khoảng thời gian trễ ngẫu nhiên (Jitter) vào các lệnh gọi. Thay vì dồn cục vào giây đầu tiên, lượng tải từ 1 triệu Lambda giờ đây được trải đều ra toàn bộ khung thời gian, làm mịn biểu đồ traffic và giải cứu hệ thống downstream.

### 2. Phân Tầng Dữ Liệu Logs/Metrics (Tiered Observability)

Họ thiết lập một bộ lọc nghiêm ngặt ngay tại nguồn của từng tài khoản:
- **High-priority**: Chỉ những log lỗi nghiêm trọng (Critical) hoặc metric cốt lõi mới được stream theo thời gian thực về tài khoản quản trị trung tâm
- **Low-priority**: Các log debug hoặc log vận hành thông thường được giữ lại tại local hoặc đẩy về S3, cắt giảm ngay lập tức gánh nặng chi phí

### 3. Đạt Trạng Thái "True Scale-to-Zero" Bằng Loại Bỏ Polling

Đội ngũ đã loại bỏ hoàn toàn cơ chế Polling của Amazon SQS nằm giữa EventBridge và Lambda đối với các tài khoản không hoạt động. Bằng tối ưu triệt để, chi phí duy trì cho mỗi tài khoản ở trạng thái rảnh rỗi đã kéo xuống **chưa tới $1/tháng**.

*(Pattern 1 - thiết kế ban đầu theo từng account: mỗi tài khoản tự giữ một chuỗi EventBridge → SQS → Lambda → DLQ riêng, nên account rảnh rỗi vẫn phải trả phí cho một hàng đợi luôn sống)*
![Pattern 1 - Single Account](/images/3-Blog/3.3-Blog3/02-pattern1-single-account.png)

*(Pattern 2 - luồng cross-account sau khi tái thiết kế: EventBridge và Lambda vẫn ở account nguồn, còn hàng đợi SQS được chuyển về một account trung tâm dùng chung, loại bỏ nhu cầu account rảnh rỗi phải tự duy trì hạ tầng polling)*
![Pattern 2 - Cross Account](/images/3-Blog/3.3-Blog3/03-pattern2-cross-account.png)

### 4. Quản Trị Tập Trung Bằng Cấu Trúc Mono-Repo

Để quản lý 1 triệu hàm hoạt động nhất quán mà không bị phân mảnh, toàn bộ 20 microservices được gom về một kho mã nguồn duy nhất. Việc này giúp:
- Áp dụng chung một bộ công cụ quét bảo mật
- Đồng bộ hóa phiên bản nâng cấp thư viện/runtime
- Chạy chuỗi CI/CD chuẩn chỉnh

## Kết Quả Đo Được

- **Chi Phí Tài Khoản Idle**: Giảm xuống <$1/tháng/account—đạt trạng thái True Scale-to-Zero
- **Hóa Đơn Giám Sát**: Tiết kiệm khổng lồ sau khi áp dụng phân tầng Logs/Metrics
- **Độ Ổn Định**: Triệt tiêu hoàn toàn Spike Traffic nhờ cơ chế Jitter
- **Vận Hành**: Quản lý 1 triệu hàm trở nên khả thi nhờ kiến trúc Mono-repo

## Bài Học Rút Ra

Case study này thách thức một giả định quan trọng: **Những gì chạy tốt ở quy mô nhỏ chưa chắc đã đúng ở quy mô lớn.**

1. **Tốc Độ Tối Ưu Phải Vượt Tốc Độ Tăng Trưởng**: Ở quy mô siêu lớn, tối ưu chi phí phải chạy nhanh hơn tăng trưởng dữ liệu.

2. **Serverless Không Miễn Phí Hoàn Toàn**: Chi phí ẩn từ các tài nguyên cấu hình (CloudWatch Alarms, Polling) tích tiểu thành đại. True Scale-to-Zero cần kỷ luật.

3. **Đơn Giản Hóa Quản Lý Mã Nguồn**: Chia nhỏ repo ở quy mô lớn là thảm họa; Mono-repo tốt là đòn bẩy hiệu quả nhất.

4. **Hệ Thống Phân Tán Cần Chiến Lược Đồng Bộ**: Jitter không chỉ là nice-to-have—ở 1 triệu hàm, nó là hạ tầng quan trọng.

Insight cốt lõi: **Serverless ở quy mô lớn đòi hỏi tư duy kỹ sư khác biệt**. Những kỹ thuật tối ưu truyền thống sẽ chạm trần—bạn phải tái thiết kế toàn bộ hệ thống để tiến lên.

**Tham Khảo**: [AWS Architecture Blog: Lessons learned from scaling to 1 million Lambda functions](https://aws.amazon.com/vi/blogs/architecture/lessons-learned-from-scaling-to-1-million-lambda-functions/)
