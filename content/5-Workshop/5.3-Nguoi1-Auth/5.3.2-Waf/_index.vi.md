---
title : "AWS WAF (Web ACL)"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

Để bảo vệ các endpoints xác thực khỏi những đợt tấn công tự động hoặc rò rỉ dữ liệu, tôi đã triển khai **AWS WAF (Web Application Firewall)**.

#### Thay đổi kiến trúc: REGIONAL Scope

Theo bản thiết kế ban đầu, WAF dự định được gắn vào CloudFront (yêu cầu tạo ở Region `us-east-1`). Tuy nhiên, để tối ưu hóa việc quản lý tài nguyên tập trung tại `ap-southeast-1` và bảo vệ trực tiếp vào tầng Authentication thay vì tầng Edge, tôi đã thay đổi Scope của WAF sang dạng **REGIONAL**.

WebACL (chuẩn Regional) này được gắn trực tiếp vào **Amazon Cognito User Pool** thông qua `AWS::WAFv2::WebACLAssociation`. Sự thay đổi này mang lại độ tin cậy cao hơn, chặn đứng các cuộc tấn công nhắm thẳng vào API xác thực (Brute force, Credential stuffing) mà bỏ qua CloudFront.

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

*(Gắn WAF vào Cognito User Pool)*
![WAF attached to Cognito](/images/5-Workshop/5.3-Nguoi1-Auth/04-waf-cognito-assoc.png)
