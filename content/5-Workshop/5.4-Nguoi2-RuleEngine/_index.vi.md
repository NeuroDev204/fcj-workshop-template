---
title : "Trần Anh Tài - Rule Engine & Sinh giáo án"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

Đây là phần việc của tôi trong dự án PeriodIQ. Tôi phụ trách luồng sinh giáo án tập luyện, từ dữ liệu người dùng nhập vào đến giáo án 4 tuần được tạo ra.

Phạm vi của tôi sử dụng 3 dịch vụ AWS chính:

| Dịch vụ AWS | Vai trò |
|---|---|
| **Lambda API Handler** | Nhận request đã xác thực từ user và cung cấp endpoint sinh giáo án |
| **Lambda Rule Engine** | Chạy các luật Volume, Conflict và Progression để sinh giáo án periodization 4 tuần |
| **Amazon S3** | Hosting SPA React + Vite có giao diện Workout Plan cho người dùng |

Ngoài ra, tôi triển khai **luồng sản phẩm Workout Plan** để user nhập thông số tập luyện, tạo giáo án, xem progression theo tuần và xem mục tiêu từng bài như set, rep, RPE, thời gian nghỉ và mức tạ.

#### Nội dung

1. [Kiến trúc & dịch vụ AWS](5.4.1-architecture-and-aws-services/)
2. [Luồng sản phẩm Workout Plan](5.4.2-workout-plan-product-flow/)
