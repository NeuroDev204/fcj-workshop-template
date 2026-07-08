---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---


Tại đây sẽ là phần liệt kê, giới thiệu các blogs mà các bạn đã đăng trên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj). Ví dụ:

###  [Blog 1 - SESSION POLICIES TRONG AMAZON EKS POD IDENTITY](3.1-Blog1/)
Blog này giới thiệu Amazon EKS Pod Identity vừa bổ sung tính năng session policies, cho phép bạn thu hẹp quyền IAM một cách linh hoạt và chính xác cho từng pod mà không cần tạo thêm nhiều IAM roles riêng biệt. Đây là bước tiến quan trọng giúp áp dụng nguyên tắc least privilege hiệu quả hơn trong môi trường Kubernetes quy mô lớn.

###  [Blog 2 - Kiến Trúc Multi-Tenant Lai Cho Stateful Services Trên AWS](3.2-Blog2/)
Blog này chia sẻ một case study từ AWS Architecture Blog về hệ thống ad-serving của Amazon Ads, xử lý hàng triệu request mỗi giây. Bài viết phân tích vì sao mô hình "mỗi tenant một account riêng" hay "share everything" đều có giới hạn với các stateful service, và trình bày kiến trúc "cellular" lai giúp cân bằng giữa cô lập tenant và hiệu quả vận hành ở quy mô lớn.

###  [Blog 3 - Scaling Serverless Lên 1 Triệu Lambda Functions](3.3-Blog3/)
Blog này chia sẻ một case study từ AWS Architecture Blog về việc scale một nền tảng SaaS lên 1 triệu Lambda functions trên hàng ngàn tài khoản AWS. Bài viết đi qua các bài toán vận hành chỉ xuất hiện ở quy mô siêu lớn — tự DDoS do cron job đồng bộ, hóa đơn giám sát tăng gấp đôi chi phí cloud, CloudFormation StackSets bị throttle, và chi phí ẩn khi Scale-to-Zero — cùng các giải pháp kiến trúc (jitter, phân tầng observability, true scale-to-zero, mono-repo) đã giải quyết chúng.