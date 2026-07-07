---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Tại đây liệt kê, giới thiệu các blog mình đã đăng trên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj).

###  [Blog 1 - Subdomain Takeover: Khi một bản ghi DNS bị lãng quên trở thành cửa ngõ tấn công](3.1-Blog1/)
Blog nói về kỹ thuật tấn công Subdomain Takeover, khi một "bản ghi DNS lơ lửng" (dangling DNS record) vẫn trỏ tới tài nguyên AWS đã bị xoá (S3, CloudFront, Elastic Beanstalk...) có thể bị kẻ tấn công chiếm lại. Bài viết mô tả cách tấn công diễn ra và giải pháp AWS đề xuất để phát hiện sớm bằng AWS Config + Lambda đối chiếu bản ghi Route 53 với tài nguyên thực tế, cảnh báo qua Security Hub và SNS.

###  [Blog 2 - Xây dựng kiến trúc Hybrid Multi-Tenant cho dịch vụ stateful trên AWS](3.2-Blog2/)
Blog tóm tắt một case study từ AWS Architecture Blog về hệ thống ad-serving của Amazon Ads, chuyển từ mô hình tốn kém "mỗi tenant một tài khoản AWS" sang kiến trúc lai 3 tầng Tier-Cell-Infra Group (Route 53 weighted routing, ALB rule riêng theo tenant, ECS cluster riêng, PrivateLink dùng chung) — giúp giảm thời gian onboarding tenant từ 52 ngày xuống còn 7 ngày.

###  [Blog 3 - Session Policies trong Amazon EKS Pod Identity](3.3-Blog3/)
Blog giới thiệu tính năng session policies vừa được bổ sung cho Amazon EKS Pod Identity, cho phép thu hẹp quyền IAM một cách linh hoạt và chính xác cho từng pod mà không cần tạo thêm nhiều IAM role riêng biệt — một bước tiến quan trọng giúp áp dụng nguyên tắc least privilege hiệu quả hơn trong môi trường Kubernetes quy mô lớn.