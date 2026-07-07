---
title: "Week 6 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---


### Week 6 Objectives:

* Attend the team kickoff meeting for the PeriodIQ capstone project and help divide responsibilities among the 5 members by AWS service ownership.
* Start getting familiar with the AWS CodePipeline / CodeBuild / SAM toolchain for the Phạm Văn Sỹ (CI/CD & Monitoring) workstream.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                              | Start Date | Completion Date | Reference Material |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | --------------- | ------------------- |
| 2   | - Team meeting: reviewed the PeriodIQ system architecture (7-layer serverless design on AWS) and divided the project into 5 role groups, each owning 3-4 AWS services | 25/05/2026 | 25/05/2026      |                      |
| 3   | - Read through the project's technical docs (architecture explanation, DynamoDB schema guide) to understand the full system before scoping the CI/CD & Monitoring workstream <br> - Set up team access on the shared AWS account: created an IAM Group per role and an individual IAM User for each of the 5 members, attached permission policies matching each person's role, and generated an access key for everyone | 26/05/2026 | 26/05/2026      |                      |
| 4   | - Researched AWS CodePipeline, AWS CodeBuild, and AWS SAM concepts, and how they fit a .NET Lambda + React frontend monorepo                       | 27/05/2026 | 27/05/2026      |                      |
| 5   | - Sketched an initial CI/CD pipeline flow (source -> build -> test -> deploy) for both backend and frontend                                        | 28/05/2026 | 28/05/2026      |                      |
| 6   | - Set up the local dev environment (AWS CLI, SAM CLI, .NET SDK), cloned the repository, and ran a first local build                                | 29/05/2026 | 29/05/2026      |                      |


### Week 6 Achievements:

* Reviewed the overall PeriodIQ architecture: Cognito/WAF/CloudFront (edge & auth), API Gateway, Lambda (API Handler/Rule Engine/Admin API), DynamoDB, SQS/SNS/SES, CloudWatch, and CodePipeline/CodeBuild/SAM.
* Helped divide the team into 5 roles by AWS service ownership:
  * Lê Hoài Huân - Auth & User Profile: Amazon Cognito, AWS WAF, Amazon CloudFront.
  * Trần Anh Tài - Rule Engine & Workout Generation: AWS Lambda (API Handler + Rule Engine), Amazon S3 (frontend hosting).
  * Lê Hữu Duy Hoàng - Progress & Async Notification: Amazon SQS, Lambda Worker, Amazon SNS/SES.
  * Chương Tử Luân - Admin Panel & Data: Lambda Admin API, Amazon DynamoDB, API Gateway.
  * **Phạm Văn Sỹ (my role) - CI/CD & Monitoring:** AWS CodePipeline, AWS CodeBuild, AWS CloudFormation/SAM, Amazon CloudWatch.
* Took on the Phạm Văn Sỹ role: responsible for Infrastructure as Code, the build/deploy pipeline, and system monitoring for the whole project.
* Studied the project documentation and the AWS CodePipeline/CodeBuild/SAM toolchain to prepare for the CI/CD & Monitoring workstream.
* As Phạm Văn Sỹ, set up the team's access to the shared AWS account: created an IAM Group per role and an individual IAM User for each of the 5 members, attached permission policies scoped to each person's assigned AWS services, and generated an access key for everyone so the team could start working independently.
* Drafted an initial CI/CD pipeline flow and set up the local development environment (AWS CLI, SAM CLI, .NET SDK).

*(Full task breakdown per role is documented in the project README - "Phân công theo thành viên".)*

**Skills gained:** Project scoping and task division by service ownership; end-to-end understanding of a full serverless AWS reference architecture; IAM identity management (Groups, Users, Policies, Access Keys) for a multi-person AWS account; AWS SAM CLI basics; local environment setup for serverless .NET development.
