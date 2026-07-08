---
title : "Lê Hoài Huân - Auth & User Profile"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

This is my role in the **PeriodIQ** project. My main responsibility is to ensure the Authentication, Authorization, and User Profile management systems operate smoothly from Frontend to Backend, while protecting the system through AWS security barriers.

Below are the 3 core AWS services I have implemented:

| AWS Service | Role |
|---|---|
| **Amazon Cognito** | Manage User Pool, Email OTP authentication, JWT issuance, and Role-based authorization via Cognito Groups (`Admin` / `Users`). |
| **AWS WAF** | Deploy Web ACL (Global scope `CLOUDFRONT` in `us-east-1`) attached directly to CloudFront to block Bots, prevent SQL Injection, XSS, and apply Rate Limiting at the Edge. |
| **Amazon CloudFront** | Setup CDN with OAC to securely host the React Frontend on S3, and route `/api/*` requests to API Gateway. |

In addition to the infrastructure services above, I am also responsible for the full Development lifecycle of this data flow:
- **Backend (.NET 10):** Setup JWT Middleware, map `cognito:groups` to `[Authorize(Roles="Admin")]`, develop `UserProfilesController` (CRUD user info), and write **15 Unit Tests** with a 100% Passed rate.
- **Frontend (React + Vite):** Build Login, Register, and Forgot Password pages. Write `AuthContext.jsx` to handle Sessions, automatically attach Bearer Tokens via Axios Interceptor, and optimize React state to prevent Race Conditions when reading Admin roles.

#### Detailed Contents

1. [Amazon Cognito & JWT Auth in ASP.NET](5.3.1-cognito/)
2. [AWS WAF (Web Application Firewall)](5.3.2-waf/)
3. [Amazon CloudFront & S3 Hosting](5.3.3-cloudfront/)
4. [Backend User Profile & Unit Tests](5.3.4-userprofile/)
