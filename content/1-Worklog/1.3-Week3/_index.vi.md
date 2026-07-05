---
title: "Worklog Tuần 3"
date: 2026-07-05
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---


### Mục tiêu tuần 3:

* Tìm hiểu tích hợp hybrid DNS giữa mạng nội bộ và Amazon VPC bằng Amazon Route 53.
* Tìm hiểu chiến lược di chuyển server và database (VM Import/Export, DMS, SCT).
* Học cách thiết kế chiến lược backup và lưu trữ file hybrid với AWS Backup và Storage Gateway.
* Làm quen với Amazon CloudFront và tham gia một sự kiện AWS.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                       | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | -------------- |
| 2   | - Thiết lập hệ thống hybrid DNS tích hợp giữa mạng nội bộ và Amazon VPC bằng Amazon Route 53                                                                                     | 04/05/2026   | 04/05/2026      | https://000010.awsstudygroup.com/ |
| 3   | - Di chuyển virtual server bằng AWS VM Import/Export <br> - Thực hiện di chuyển database bằng AWS Database Migration Service (DMS) và Schema Conversion Tool (SCT)              | 05/05/2026   | 05/05/2026      | https://cloudjourney.awsstudygroup.com/2-migrate/ |
| 4   | - Triển khai kế hoạch backup hệ thống với AWS Backup                                                                                                                             | 06/05/2026   | 06/05/2026      | https://000013.awsstudygroup.com/ |
| 5   | - Sử dụng AWS Storage Gateway (File Gateway) <br> - Tìm hiểu Amazon CloudFront                                                                                                   | 07/05/2026   | 07/05/2026      |                 |
| 7   | - Tham gia sự kiện AWS                                                                                                                                                            | 09/05/2026   | 09/05/2026      |                 |


### Kết quả đạt được tuần 3:

* Thiết lập tích hợp hybrid DNS giữa mạng nội bộ và Amazon VPC bằng Amazon Route 53.
* Di chuyển virtual server bằng AWS VM Import/Export và thực hiện di chuyển database liên môi trường bằng AWS DMS và Schema Conversion Tool (SCT).
* Triển khai kế hoạch backup hệ thống với AWS Backup.
* Học cách sử dụng AWS Storage Gateway (File Gateway) cho lưu trữ file hybrid, và làm quen với Amazon CloudFront cho việc phân phối nội dung.
* Tham gia một sự kiện AWS chính thức để mở rộng networking và kiến thức thực tế.

**Khó khăn & Giải pháp:**

* **Khó khăn:** Quên xóa các EBS volume không dùng sau khi thực hành Storage Gateway, dẫn đến phát sinh chi phí ngoài dự kiến (~2.1$).
* **Giải pháp:** Xóa toàn bộ volume không dùng và tạo thói quen dọn dẹp tài nguyên thử nghiệm ngay sau khi sử dụng.

**Dịch vụ AWS đã học trong tuần:** Amazon Route 53, AWS VM Import/Export, AWS Database Migration Service (DMS), AWS Schema Conversion Tool (SCT), AWS Backup, AWS Storage Gateway, Amazon CloudFront.


