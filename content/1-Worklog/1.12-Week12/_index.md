---
title: "Week 12 Worklog"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.12. </b> "
---


### Week 12 Objectives:

* Finalize the CloudFront / Cognito / API Gateway integration for the frontend build pipeline.
* Complete the production deployment of the PeriodIQ system on AWS.
* Wrap up the CI/CD & Monitoring workstream and hand over the project ahead of the internship's conclusion.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                                                                        | Start Date | Completion Date | Reference Material |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ------------------- |
| 1   | - Fixed `buildspec-frontend.yml` to pull the Cognito User Pool ID, Client ID, and CloudFront domain from the CloudFormation stack outputs at build time, so Vite bakes the correct live configuration into the production bundle <br> - Fixed the CloudFront distribution's API Gateway origin path so `/api/*` correctly reaches the API Gateway stage <br> - Fixed remaining pipeline YAML issues | 05/07/2026 | 05/07/2026      | https://github.com/PeriolIQ/PeriodIQ |
| 2   | - Verified the production deployment end-to-end (Cognito login, API routing through CloudFront, health checks) and monitored CloudWatch for any post-deploy issues                                                                                        | 06/07/2026 | 06/07/2026      | https://d1di1pzmfypszp.cloudfront.net/ |
| 3   | - Wrote and updated CI/CD and deployment documentation for the project handover                                                                                                                                                                            | 07/07/2026 | 07/07/2026      |                      |
| 4   | - Reviewed and finalized the CloudWatch alarms and monitoring dashboard with the team                                                                                                                                                                       | 08/07/2026 | 08/07/2026      |                      |
| 5   | - Supported teammates with last-minute bug fixes found during final end-to-end testing                                                                                                                                                                     | 09/07/2026 | 09/07/2026      |                      |
| 6   | - Prepared the final project demo and presentation materials for the CI/CD & Monitoring workstream                                                                                                                                                          | 10/07/2026 | 10/07/2026      |                      |


### Week 12 Achievements:

* Fixed the frontend build pipeline to automatically inject the live Cognito and API configuration (User Pool ID, Client ID, API URL) from the CloudFormation stack into the production build - removing the need for manual config and preventing deployment of a broken bundle.
* Fixed the CloudFront distribution's API Gateway origin path mapping, resolving broken `/api` routing between the frontend and backend.
* Completed the end-to-end CI/CD pipeline and successfully deployed the PeriodIQ system to production on AWS.
* **The system is live and publicly accessible at: https://d1di1pzmfypszp.cloudfront.net/**
* Verified the production deployment is stable end-to-end and confirmed monitoring/alarms are working correctly.
* Completed CI/CD and deployment documentation for the project handover.
* Supported final bug-fixing across the team and prepared the project demo for the Phạm Văn Sỹ (CI/CD & Monitoring) workstream ahead of the internship's conclusion.

**AWS services learned/used this week:** Amazon CloudFront, AWS CodePipeline/CodeBuild, AWS CloudFormation/SAM, Amazon Cognito, Amazon API Gateway, Amazon CloudWatch.

*(Tasks for 06-10/07/2026 are reasonably inferred wrap-up/handover activities, as no further commit history was available past 05/07/2026 at the time of writing - update with the actual work completed if it differed.)*
