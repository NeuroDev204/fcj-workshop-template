---
title: "Blog 1"
date: 2026-07-12
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# SUBDOMAIN TAKEOVER: KHI MỘT BẢN GHI DNS BỊ LÃNG QUÊN TRỞ THÀNH CỬA NGÕ TẤN CÔNG

Trong quá trình tìm hiểu về Cloud Security, mình có đọc được một bài viết khá thú vị từ AWS Security Blog về một kỹ thuật tấn công có tên là **Subdomain Takeover** (chiếm đoạt tên miền phụ).

Điều mình thấy đáng chú ý là kỹ thuật này không khai thác lỗ hổng của AWS hay lỗi trong ứng dụng, mà xuất phát từ một vấn đề rất phổ biến trong quá trình vận hành hạ tầng: quản lý DNS và tài nguyên Cloud không đồng bộ.

## Bài toán

Khi triển khai ứng dụng hoặc các chiến dịch marketing trên AWS, chúng ta thường tạo các tài nguyên như Amazon S3, CloudFront hoặc Elastic Beanstalk, sau đó cấu hình DNS để trỏ một subdomain của doanh nghiệp tới các tài nguyên này.

Ví dụ:

```
api.congty.com → my-bucket.s3.amazonaws.com
```

Sau khi dự án kết thúc, tài nguyên AWS được xóa để tiết kiệm chi phí. Tuy nhiên trong nhiều trường hợp, bản ghi DNS tương ứng lại không được dọn dẹp.

Lúc này, subdomain vẫn tồn tại nhưng đang trỏ tới một tài nguyên không còn thuộc quyền kiểm soát của tổ chức. AWS gọi đây là **Dangling DNS Record**.

## Kịch bản tấn công diễn ra như thế nào?

Theo AWS, các đối tượng tấn công thường sử dụng các công cụ quét tự động để tìm kiếm những bản ghi DNS dạng này trên Internet.

Khi phát hiện một bản ghi DNS đang trỏ tới tài nguyên đã bị xóa, kẻ tấn công có thể đăng ký lại tài nguyên đó nếu dịch vụ cho phép tái sử dụng tên. Ví dụ với Amazon S3, nếu bucket đã bị xóa và tên bucket chưa được ai sử dụng, kẻ tấn công có thể tạo lại bucket cùng tên.

Kết quả là toàn bộ truy cập tới subdomain của doanh nghiệp sẽ được chuyển tới tài nguyên do kẻ tấn công kiểm soát.

## Những rủi ro có thể phát sinh

Bài viết của AWS đề cập đến một số tác động đáng chú ý:

* Phát tán nội dung giả mạo trên chính subdomain của doanh nghiệp.
* Thực hiện các chiến dịch lừa đảo (Phishing) thông qua tên miền có độ tin cậy cao.
* Ảnh hưởng đến uy tín thương hiệu và trải nghiệm khách hàng.
* Gia tăng nguy cơ phát sinh các sự cố bảo mật khác nếu không được phát hiện kịp thời.

## Giải pháp AWS đề xuất

Điểm mình thấy khá hay là AWS không chỉ mô tả rủi ro mà còn đưa ra cách tiếp cận để phát hiện và xử lý các bản ghi DNS lơ lửng.

Giải pháp được đề xuất sử dụng AWS Config kết hợp với AWS Lambda để đối chiếu giữa các bản ghi DNS trên Route 53 và danh sách tài nguyên thực tế trong tài khoản AWS. Khi phát hiện một bản ghi DNS đang trỏ tới tài nguyên không còn tồn tại, hệ thống có thể:

* Đánh dấu cấu hình không tuân thủ (Non-Compliant).
* Gửi cảnh báo tới AWS Security Hub.
* Thông báo cho đội ngũ vận hành thông qua Amazon SNS.

Qua đó giúp phát hiện sớm các rủi ro trước khi bị khai thác.

![Kiến trúc phát hiện Dangling DNS Record](/images/3-Blog/blog1.png)

### Những dịch vụ AWS xuất hiện trong kiến trúc

* AWS Config
* AWS Lambda
* Amazon Route 53
* AWS Security Hub
* Amazon SNS
* Amazon CloudWatch

## Bài học mình rút ra

Điều mình thấy thú vị nhất từ bài viết này là Subdomain Takeover không phải là lỗ hổng của AWS. Vấn đề thực sự nằm ở quy trình vận hành và quản lý vòng đời tài nguyên. Một tài nguyên có thể đã được xóa, nhưng nếu DNS vẫn còn tồn tại thì bề mặt tấn công vẫn chưa thực sự biến mất.

Case study này cũng cho thấy rằng:

* Bảo mật không chỉ nằm ở code hay cấu hình hệ thống.
* DNS cũng là một phần quan trọng của attack surface.
* Việc tự động hóa kiểm tra cấu hình giúp giảm đáng kể các sai sót thủ công.
* Quy trình vận hành tốt đôi khi quan trọng không kém các giải pháp bảo mật phức tạp.

**Nguồn tham khảo:** [AWS Security Blog - Threat tactic spotlight: subdomain takeover](https://aws.amazon.com/vi/blogs/security/threat-tactic-spotlight-subdomain-takeover/)

**Facebook:** [AWS Study Group FCJ](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2187399542025006/)