---
title : "Lê Hoài Huân - Auth & User Profile"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

Đây là vai trò của chính tôi trong dự án **PeriodIQ**. Trách nhiệm chính của tôi là đảm bảo hệ thống xác thực (Authentication), phân quyền (Authorization) và quản lý hồ sơ người dùng (User Profile) hoạt động trơn tru từ Frontend đến Backend, đồng thời bảo vệ hệ thống thông qua các rào chắn bảo mật trên AWS.

Dưới đây là 3 dịch vụ AWS cốt lõi mà tôi đã thiết lập:

| Dịch vụ AWS | Vai trò |
|---|---|
| **Amazon Cognito** | Quản lý User Pool, xác thực OTP qua Email, phát hành JWT và phân quyền bằng Cognito Groups (`Admin` / `Users`). |
| **AWS WAF** | Triển khai Web ACL (chuẩn Regional) gắn trực tiếp vào Cognito để chặn Bot, chống SQL Injection, XSS và Rate Limit. |
| **Amazon CloudFront** | Thiết lập CDN với OAC để host Frontend React trên S3 an toàn, định tuyến (route) requests `/api/*` tới API Gateway. |

Ngoài các dịch vụ hạ tầng trên, tôi cũng chịu trách nhiệm toàn bộ quá trình phát triển (Development) cho luồng dữ liệu này:
- **Backend (.NET 10):** Thiết lập Middleware xử lý JWT, ánh xạ `cognito:groups` sang `[Authorize(Roles="Admin")]`, phát triển `UserProfilesController` (CRUD thông tin user), và viết **15 Unit Tests** đạt tỷ lệ 100% Passed.
- **Frontend (React + Vite):** Xây dựng các trang Đăng nhập, Đăng ký, Quên mật khẩu. Viết `AuthContext.jsx` xử lý Session, tự động đính kèm Bearer Token qua Axios Interceptor và tối ưu hóa React state để tránh lỗi Race Condition khi đọc quyền Admin.

#### Nội dung chi tiết các phần

1. [Amazon Cognito & JWT Auth trong ASP.NET](5.3.1-cognito/)
2. [AWS WAF (Web Application Firewall)](5.3.2-waf/)
3. [Amazon CloudFront & S3 Hosting](5.3.3-cloudfront/)
4. [Backend User Profile & Unit Test](5.3.4-userprofile/)
