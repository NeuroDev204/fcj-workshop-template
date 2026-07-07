---
title : "Chương Tử Luân - Admin Panel & Data"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

This is my role in the PeriodIQ project. I am responsible for three core AWS services:

| AWS Service | Role |
|---|---|
| **Amazon DynamoDB** | Full schema design for 8 tables, table creation, GSIs, seed data |
| **Lambda Admin API** | 4 backend controllers (Exercise, Template, Rule, AdminStats) |
| **API Gateway (HTTP)** | Routes configuration, Cognito JWT Authorizer, CORS |

Additionally, I built the entire **Admin Panel frontend** — 4 admin pages (Dashboard, Exercises, Rules, Templates) — and wrote **unit tests** for all controllers in this role.

#### Content

1. [Amazon DynamoDB](5.6.1-dynamodb/)
2. [Lambda Admin API](5.6.2-lambda-admin-api/)
3. [Admin Panel Frontend](5.6.3-admin-panel-frontend/)
