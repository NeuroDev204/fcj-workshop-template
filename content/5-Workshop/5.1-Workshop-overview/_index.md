---
title : "Introduction"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### What is PeriodIQ?

PeriodIQ is a serverless application that generates a personalized 4-week gym workout plan for a user based on their fitness level, body weight, training goal, and Personal Records, using a rule-based engine (Volume Filter -> Conflict Resolution -> Progression Builder). The full architecture (16 AWS services across 8 layers) is described in the [Proposal](../../2-proposal/) section.

#### How the team split the work

| Section | Role | AWS Services |
|---|---|---|
| Lê Hoài Huân | Auth & User Profile | Amazon Cognito, AWS WAF, Amazon CloudFront |
| Trần Anh Tài | Rule Engine & Workout Generation | Lambda (API Handler + Rule Engine), Amazon S3 |
| Lê Hữu Duy Hoàng | Progress & Async Notification | Amazon SQS, Lambda Worker, Amazon SNS |
| Chương Tử Luân | Admin Panel & Data | Lambda Admin API, Amazon DynamoDB, API Gateway |
| **Phạm Văn Sỹ (me)** | **CI/CD & Monitoring** | **AWS CodePipeline, AWS CodeBuild, CloudFormation/SAM, Amazon CloudWatch** |

#### Scope of this workshop

This workshop documents **my own role, Phạm Văn Sỹ**, in detail, using the **actual production pipeline and infrastructure** - not a simplified rebuild. `periodiq-pipeline-dev` (defined in the real `cicd-pipeline.yml`), the CodeBuild projects it runs, the SAM-deployed `periodiq-dev` stack, and the CloudWatch monitoring around them are all real, already-deployed resources in the AWS account this project runs in. Every command in [section 5.7](../5.7-nguoi5-cicd/) is a genuine `aws` CLI call against that real deployment - reading real pipeline executions, real build logs, real stack outputs, and real alarm states - with a real terminal screenshot as evidence for each one.

> **CLI Note:** because this workshop documents the live production system rather than a disposable sandbox copy, everything here is **read-only** - `describe-*`, `get-*`, `list-*` - against the real pipeline/stack. Nothing was deployed, changed, or deleted on the real infrastructure while writing this section (see [Clean up](../5.8-cleanup/) for what that means in practice).
