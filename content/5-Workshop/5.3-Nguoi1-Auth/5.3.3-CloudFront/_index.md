---
title : "Amazon CloudFront & S3 Hosting"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

To distribute the React website to users with high speed and the lowest latency, I set up **Amazon CloudFront** to act as a global Content Delivery Network (CDN).

In our architecture, CloudFront is responsible for routing data from two entirely different Origins.

#### 1. S3 Origin (Frontend Static Files)
The React website (SPA) is built and stored in an Amazon S3 Bucket.
- All default requests (`Default Cache Behavior: /*`) are routed by CloudFront to this S3 bucket.
- For enhanced security, I applied **Origin Access Control (OAC)**. As a result, the S3 Bucket completely blocks public access (Block all public access). The only way to read files on S3 is to pass through CloudFront.
- Due to the nature of Single Page Applications (SPA), I also configured a default error page on CloudFront: When a `404 Not Found` error occurs (because a user manually enters a React Router sub-path), CloudFront returns a `200 OK` status with the contents of the `/index.html` file, allowing React to handle the routing internally.

#### 2. API Gateway Origin (Backend APIs)
All requests with the `/api/*` prefix bypass S3 and are routed directly to **Amazon API Gateway** (which sits in front of the .NET Lambda Backend).
- For this path, I disabled caching (Cache Disabled) to ensure the latest real-time data is always fetched.
- I configured it to forward all crucial HTTP Headers (like `Authorization: Bearer ...`) from the user to the Backend to ensure accurate identity verification.

*(CloudFront Origins Configuration and S3 Bucket Policy with OAC Configuration )*
![CloudFront Origins Setup and S3 OAC Policy](/images/5-Workshop/5.3-Nguoi1-Auth/05-cloudfront-origins.png)
