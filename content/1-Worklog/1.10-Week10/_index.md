---
title: "Week 10 Worklog"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---


### Week 10 Objectives:

* Acting in the Phạm Văn Sỹ (CI/CD & Monitoring) role: build the initial CI/CD and Infrastructure-as-Code foundation for PeriodIQ - SAM template, samconfig, a health-check controller, and the backend/frontend buildspecs.

### Tasks to be carried out this week:
| Day | Task                                                                                                                   | Start Date | Completion Date | Reference Material |
| --- | ------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ------------------- |
| 4   | - Created the SAM template (`template.yml`) and `samconfig.toml` <br> - Implemented `HealthController.cs` health-check endpoint <br> - Created `buildspec-backend.yml` for the backend CodeBuild stage | 25/06/2026 | 25/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 5   | - Added `frontend/.env.example` <br> - Fixed a syntax error in `buildspec-backend.yml` <br> - Created `buildspec-frontend.yml` (tested successfully)                                                    | 26/06/2026 | 26/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 6   | - Updated `buildspec-backend.yml`: added test report & CodeBuild report, artifact versioning <br> - Updated `buildspec-frontend.yml`: added a security scan step                                        | 27/06/2026 | 27/06/2026      | https://github.com/PeriolIQ/PeriodIQ |


### Week 10 Achievements:

* Authored the SAM template (`template.yml`) and `samconfig.toml`, defining the PeriodIQ serverless infrastructure as code (Lambda, HTTP API, DynamoDB, S3, SQS, SNS).
* Implemented the `HealthController` health-check endpoint, used later for pipeline smoke tests and uptime monitoring.
* Created `buildspec-backend.yml` (restore/build/test/package) and `buildspec-frontend.yml` (install/build/deploy), and verified each build phase runs successfully.
* Added test reporting and CodeBuild reports, plus artifact versioning to the backend build; added a security scan step to the frontend build.

**AWS services learned/used this week:** AWS CloudFormation / AWS SAM, AWS CodeBuild, Amazon CloudWatch (health checks).
