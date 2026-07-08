---
title : "AWS WAF (Web ACL)"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

Để bảo vệ các endpoints xác thực khỏi những đợt tấn công tự động hoặc rò rỉ dữ liệu, tôi đã triển khai **AWS WAF (Web Application Firewall)**.

#### Bảo vệ biên mạng với Global Scope (CLOUDFRONT)

Tuân thủ chặt chẽ theo thiết kế kiến trúc chuẩn của dự án PeriodIQ, tôi đã triển khai WAF WebACL với **Scope CLOUDFRONT** (được khởi tạo bắt buộc tại Region `us-east-1`). 

WebACL này được gắn trực tiếp vào **Amazon CloudFront Distribution** thông qua thuộc tính `WebACLId`. Chiến lược này tạo ra một "tấm khiên" khổng lồ bảo vệ toàn diện hệ thống ở tầng Edge, giúp chặn đứng các luồng truy cập độc hại (DDoS, Bot, SQLi) ngay từ vòng ngoài trước khi chúng có cơ hội chạm tới giao diện React (S3) hay hệ thống API Gateway Backend.

#### Các lớp quy tắc bảo vệ (Rules)

Tôi đã thiết lập 3 nhóm quy tắc bảo mật chính yếu:

1. **Bot Control (`AWSManagedRulesBotControlRuleSet`)**
   - Chặn đứng các bot độc hại, scraper, và các script tự động dò quét thông tin người dùng.
   
2. **Core Rule Set (`AWSManagedRulesCommonRuleSet`)**
   - Bao gồm các tập luật bảo vệ khỏi lỗ hổng bảo mật web phổ biến (OWASP Top 10) như SQL Injection (SQLi) và Cross-Site Scripting (XSS).

3. **Rate Limiting (Tự định nghĩa)**
   - Nhằm ngăn chặn các đợt tấn công DDoS diện hẹp hoặc Brute-force mật khẩu.
   - Tôi đã thiết lập giới hạn: **2000 requests / 5 phút** từ cùng một địa chỉ IP. Nếu vượt quá, các truy cập tiếp theo từ IP đó sẽ bị chặn hoàn toàn cho đến khi hết chu kỳ.

*(Cấu hình WebACL trên AWS WAF)*
![WAF WebACL Rules](/images/5-Workshop/5.3-Nguoi1-Auth/03-waf-rules.png)

*(Gắn WAF vào CloudFront Distribution)*
![WAF attached to CloudFront](/images/5-Workshop/5.3-Nguoi1-Auth/04-waf-cloudfront-assoc.png)
