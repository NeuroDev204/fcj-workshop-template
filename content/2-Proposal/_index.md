---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# PeriodIQ - Serverless Periodization Engine
## A Science-Based, Automated Workout Plan Generator on AWS Serverless

### 1. Executive Summary
PeriodIQ is a serverless web platform that acts as an "automatic personal trainer": a gym-goer enters their personal metrics (body weight, fitness level, training goal, available days per week), and a rule-based engine running on AWS Lambda generates a scientifically periodized 4-week workout plan with concrete target weights (kg) for every exercise. Instead of following a generic, one-size-fits-all template found online, users get a plan that respects their real training capacity, tracks their Personal Records (PR), and adapts week over week - including automatic deload weeks to reduce the risk of injury from central nervous system (CNS) fatigue. The system is built entirely on AWS serverless services (16 services across 8 architectural layers) and was developed and deployed by a 5-person team, each owning a distinct service group.

### 2. Problem Statement

**What's the Problem?**
Most gym-goers rely on static, generic workout templates (from the internet, a book, or a paid coach) that do not account for their individual fitness level, actual strength (1RM/PR), recovery capacity, or number of available training days. Progressive overload is rarely tracked systematically - people either under-train (no progression) or over-train (stacking high-CNS-stress exercises back to back, risking injury and burnout). Manually building a periodized plan and adjusting it week by week requires expertise most beginners and intermediate lifters don't have, while dedicated coaching apps/services are often costly and complex.

**The Solution**
PeriodIQ automates evidence-based periodization through a 3-step Rule Engine:
1. **Volume Filter** - caps total weekly training volume (sets) per muscle group so no muscle group is overtrained.
2. **Conflict Resolution** - prevents scheduling two high-CNS-stress exercises on the same day, redistributing neural fatigue across the week.
3. **Progression Builder** - increases load/reps week over week based on the user's Personal Records and fitness level, and automatically inserts a deload week (reduced volume) at the end of the 4-week cycle.

Users also log daily CNS status (sleep, stress, soreness) and actual workout results (sets/reps/kg/RPE), which feed back into the Rule Engine to keep future plans realistic and safe. Admins manage the underlying exercise library, workout templates, and rule parameters through a dedicated admin panel.

**Benefits and Value**
- Personalized, science-based programming without needing a human coach.
- Automatic progressive overload and deload scheduling, reducing injury risk from CNS overload.
- Async email/reminder notifications keep users engaged without slowing down the main request flow.
- Entirely serverless (pay-per-use): no idle infrastructure cost, scales automatically with usage, and is cheap to run for a small user base.
- Doubles as a practical, full-stack team learning project: 5 members each build and operate a real slice of a 16-service AWS architecture, including a real CI/CD pipeline and production deployment.

### 3. Solution Architecture

The system is deployed in AWS region `ap-southeast-1` and organized into 8 layers: Edge (WAF + CloudFront + S3), Auth (Cognito), API (API Gateway), Core Engine (3 Lambda functions), Data (DynamoDB), Async/Messaging (SQS + Lambda Worker + SNS/SES), Monitoring (CloudWatch), and CI/CD (CodePipeline + CodeBuild + CloudFormation/SAM).

![PeriodIQ AWS Architecture](/images/2-Proposal/aws_architecture.png)

**Main request flow (Generate Workout Plan):**
1. The User/Admin accesses the app (web + API) through **CloudFront**, protected at the edge by **AWS WAF** (blocks bots/SQLi/XSS and rate-limits per IP).
2. CloudFront serves the static React SPA from **S3**, and routes `/api/*` to **API Gateway (HTTP API)**.
3. API Gateway verifies the request's JWT via a **Cognito** authorizer.
4. API Gateway invokes the appropriate Lambda: **API Handler** (user requests), **Rule Engine** (plan generation), or **Admin API** (admin operations).
5. The Rule Engine reads the user's profile, Personal Records, recent CNS check-ins, and active rules, then runs Volume Filter -> Conflict Resolution -> Progression Builder, and saves the generated 4-week plan to **DynamoDB** (on-demand capacity + TTL cache).
6. The API Handler enqueues an email job onto **Amazon SQS** so the user gets an instant response without waiting for email delivery.
7. A **Lambda Worker**, triggered by SQS, publishes the workout-plan email / workout reminder via **Amazon SNS/SES**.
8. **CloudWatch** collects Logs, Metrics, and Alarms across the Core Engine and Async layers for observability.
9. Developers push code to GitHub, which triggers the **CI/CD layer**: **CodePipeline** orchestrates **CodeBuild** (build & test) and deploys infrastructure via **CloudFormation/SAM**.

### AWS Services Used (16 services)
| # | Service | Layer | Role |
|---|---------|-------|------|
| 1 | AWS WAF | Edge | Web application firewall (WebACL, scope CLOUDFRONT) attached to the CloudFront distribution |
| 2 | Amazon CloudFront | Edge | CDN and routing, protected by WAF |
| 3 | Amazon S3 | Edge | Hosting the React SPA (static frontend) |
| 4 | Amazon Cognito | Auth & Security | User/Admin authentication, issues JWTs |
| 5 | Amazon API Gateway (HTTP) | API | Receives requests, verifies JWT |
| 6 | AWS Lambda - API Handler | Core Engine | Handles user-facing requests |
| 7 | AWS Lambda - Rule Engine | Core Engine | Generates the 4-week workout plan |
| 8 | AWS Lambda - Admin API | Core Engine | Handles admin operations |
| 9 | Amazon DynamoDB | Data | Storage + TTL cache (8 tables, on-demand billing) |
| 10 | Amazon SQS | Async | Asynchronous task queue |
| 11 | AWS Lambda - Worker | Async | Processes messages from SQS |
| 12 | Amazon SNS / SES | Async | Sends emails and notifications |
| 13 | Amazon CloudWatch | Monitoring | Logs, Metrics, Alarms |
| 14 | AWS CodePipeline | CI/CD | Orchestrates the deployment pipeline |
| 15 | AWS CodeBuild | CI/CD | Builds and tests the source code |
| 16 | AWS CloudFormation / SAM | CI/CD | Infrastructure as Code |

### Component Design
- **Edge & Frontend**: A React 19 + Vite SPA hosted on S3, served through CloudFront (with Origin Access Control) and shielded by AWS WAF managed rule sets (Common Rule Set + Bot Control) plus a per-IP rate limit.
- **Auth**: Amazon Cognito User Pool with `Users` and `Admins` groups; the .NET backend validates the Cognito JWT and maps the `sub` claim to `UserProfile.Id`.
- **Core Engine**: Built as a "Monolithic Lambda" - a single ASP.NET Core Web API project (`PeriodIQ.Api`) hosted via `Amazon.Lambda.AspNetCoreServer.Hosting`, following Clean Architecture so the Rule Engine and Worker logic live in shared `PeriodIQ.Core` services and could be split into separate Lambda functions later without rewriting business logic.
- **Data**: 8 DynamoDB tables (master data, user profiles/tracking, generated workout plans) using `PAY_PER_REQUEST` billing, with GSIs for the tables that need complex queries.
- **Async/Messaging**: SQS decouples slow notification work (plan emails, workout reminders, PR congratulations) from the main request/response cycle; a Lambda Worker consumes the queue and sends via SNS/SES.
- **Monitoring**: CloudWatch Logs, custom metric filters (e.g., Rule Engine execution time, plan generation count, error count), and Alarms for Lambda error rate/duration, DynamoDB throttling, and concurrency limits.
- **CI/CD**: CodePipeline (triggered by a GitHub webhook via CodeStar Connections) runs `buildspec-backend.yml` (restore/build/test/package via SAM) and `buildspec-frontend.yml` (install/security scan/build/deploy to S3 + CloudFront invalidation).

### 4. Technical Implementation

**Team structure** - the project was split among 5 members, each owning 3-4 AWS services end-to-end (backend + frontend):
- **Lê Hoài Huân - Auth & User Profile**: Amazon Cognito, AWS WAF, Amazon CloudFront.
- **Trần Anh Tài - Rule Engine & Workout Generation**: AWS Lambda (API Handler + Rule Engine), Amazon S3 (frontend hosting).
- **Lê Hữu Duy Hoàng - Progress & Async Notification**: Amazon SQS, Lambda Worker, Amazon SNS/SES.
- **Chương Tử Luân - Admin Panel & Data**: Lambda Admin API, Amazon DynamoDB, API Gateway.
- **Phạm Văn Sỹ (my role) - CI/CD & Monitoring**: AWS CodePipeline, AWS CodeBuild, AWS CloudFormation/SAM, Amazon CloudWatch.

**Tech Stack**
- Backend: .NET 10 (upgraded from .NET 9 mid-project), ASP.NET Core Web API, Clean Architecture (5 projects), Amazon DynamoDB, Amazon SQS, Serilog, Swagger/Scalar, AWS SAM/CloudFormation.
- Frontend: React 19, Vite 8, Tailwind CSS 4 + Mantine 9, TanStack React Query 5, Axios, React Router 7.

**Implementation Phases**
1. Architecture design and team role division (planning weeks).
2. AWS architecture diagram design, review, and revision based on team feedback.
3. Backend and infrastructure implementation (Rule Engine, controllers, DynamoDB schema, IAM setup).
4. CI/CD pipeline construction (SAM template, buildspecs, CodePipeline), monitoring dashboard, and production deployment.

### 5. Timeline & Milestones
- **Weeks 1-5**: AWS Study Group curriculum (individual, foundational AWS labs).
- **Week 6**: Team kickoff - reviewed the architecture, divided roles by service ownership, and set up team-wide AWS access (IAM Groups/Users/access keys).
- **Weeks 7-9**: Designed and iterated on the AWS architecture diagram based on team feedback; began backend implementation.
- **Week 10**: Built the CI/CD foundation - SAM template, `samconfig.toml`, health-check controller, and backend/frontend buildspecs.
- **Week 11**: Built and stabilized the CodePipeline/CodeBuild pipeline; upgraded the backend to .NET 10; built the CI/CD monitoring dashboard.
- **Week 12**: Finalized the CloudFront/Cognito/API Gateway integration and completed the production deployment.

### 6. Budget Estimation
PeriodIQ runs on a fully serverless, pay-per-use architecture with no idle compute cost. For a small-scale capstone deployment (a handful of active users, low request volume), the dominant cost drivers are:
- **AWS Lambda**: billed per invocation + duration; negligible at low traffic (largely covered by the AWS Free Tier's 1M free requests/month).
- **Amazon DynamoDB**: `PAY_PER_REQUEST` billing across 8 tables - cost scales with actual reads/writes, not provisioned capacity.
- **Amazon API Gateway (HTTP API)**: billed per request; the cheapest API Gateway type available.
- **Amazon CloudFront + S3**: minimal cost for a small SPA bundle and low request volume.
- **Amazon SQS / SNS / SES**: negligible at low message volume (all have generous free tiers).
- **AWS CodePipeline / CodeBuild**: a small fixed cost per active pipeline plus CodeBuild compute-minutes per build.

*(No fixed AWS Pricing Calculator estimate is included here, since actual cost depends on final user traffic; the architecture was deliberately chosen to keep marginal cost near zero at low scale.)*

### 7. Risk Assessment
- **CNS/Volume miscalculation leading to unsafe plans** (medium impact, low probability) - mitigated by the Volume Filter and Conflict Resolution rules, plus daily CNS check-ins that can trigger an automatic deload.
- **Lambda cold starts / custom runtime issues** (medium impact, medium probability, especially after the .NET 10 upgrade) - mitigated by pinning the SDK version, fixing the bootstrap assembly name for the custom Lambda runtime, and validating with the CI/CD pipeline's automated tests.
- **CI/CD pipeline drift** (e.g., a stack losing sync with its actual deployed resources) - mitigated by treating CloudFormation/SAM as the single source of truth and re-deploying (delete + redeploy) when drift is detected.
- **Broken frontend/backend integration after deploy** (e.g., wrong CloudFront origin path, stale Cognito config baked into the frontend bundle) - mitigated by pulling live Cognito/API configuration from the CloudFormation stack outputs at build time, rather than hardcoding it.

### 8. Expected Outcomes
- A fully working, publicly deployed serverless application generating personalized, science-based workout plans - live at **https://d1di1pzmfypszp.cloudfront.net/**.
- A reusable reference architecture and CI/CD pipeline for future .NET-on-Lambda serverless projects.
- Hands-on, end-to-end experience across the full AWS service stack (auth, API, compute, data, messaging, monitoring, CI/CD) for all 5 team members.
