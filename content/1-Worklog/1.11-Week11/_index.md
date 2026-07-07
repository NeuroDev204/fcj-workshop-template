---
title: "Week 11 Worklog"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---


### Week 11 Objectives:

* Build and stabilize the CI/CD pipeline (AWS CodePipeline + CodeBuild) for PeriodIQ.
* Upgrade the backend to .NET 10 and fix the resulting Lambda custom-runtime issues.
* Build out the CI/CD monitoring dashboard on the frontend.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                                     | Start Date | Completion Date | Reference Material |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | --------------- | ------------------- |
| 1   | - Created the CI/CD pipeline definition `cicd-pipeline.yml` (AWS CodePipeline orchestrating the backend/frontend CodeBuild stages)                                                                                      | 28/06/2026 | 28/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 2   | - Wrote and updated the project README documentation                                                                                                                                                                    | 29/06/2026 | 29/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 3   | - Updated `template.yml` with infrastructure fixes                                                                                                                                                                       | 30/06/2026 | 30/06/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 4   | - Connected the GitHub repository via AWS CodeStar Connections and ran the pipeline <br> - Upgraded the backend to .NET 10 (SDK install path, `global.json` pinning, custom runtime bootstrap assembly name fixes) <br> - Converted `buildspec-frontend.yml` to use pnpm <br> - Updated the admin dashboard | 01/07/2026 | 01/07/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 5   | - Fixed the admin "deploy history" API call <br> - Redesigned the monitoring dashboard <br> - Displayed the pipeline's 4 build stages with live logs while building <br> - Added charts to the monitoring page and fixed a Lambda crash | 02/07/2026 | 02/07/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 7   | - Added monitoring charts and updated the README for Lê Hoài Huân <br> - Fixed failing unit tests <br> - Corrected the AWS WAF WebACL region configuration                                                                  | 04/07/2026 | 04/07/2026      | https://github.com/PeriolIQ/PeriodIQ |


### Week 11 Achievements:

* Built the CodePipeline/CodeBuild CI/CD pipeline (`cicd-pipeline.yml`), orchestrating the backend and frontend build/deploy stages end to end.
* Upgraded the entire backend to .NET 10, resolving Lambda custom runtime (`provided.al2023`), .NET 10 SDK installation, and bootstrap assembly name issues.
* Connected the GitHub repository to CodePipeline via AWS CodeStar Connections and validated full pipeline runs.
* Built the CI/CD monitoring dashboard on the frontend: live build-stage progress, real-time build logs, and monitoring charts.
* Fixed failing unit tests and corrected the AWS WAF WebACL region configuration.

**AWS services learned/used this week:** AWS CodePipeline, AWS CodeBuild, AWS CodeStar Connections, AWS CloudFormation/SAM, AWS Lambda (custom runtime), AWS WAF, Amazon CloudWatch.
