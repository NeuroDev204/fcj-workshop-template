---
title : "Lê Hữu Duy Hoàng - Progression & Async Notification"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

This is my role in the **PeriodIQ** project. My core responsibility is handling all **Asynchronous Data Flows** and **User Progression Tracking** logic, from the moment users finish their workout to the moment the system sends notifications.

Below are the 3 core AWS services I set up and integrated:

| AWS Service | Role |
|---|---|
| **Amazon SQS** | Acts as a Message Queue to offload heavy tasks (e.g., generating plans and sending emails) to free up the API for instant responses to the User. |
| **AWS Lambda (Worker)** | A background function automatically triggered whenever a new message falls into SQS. Responsible for post-processing logic. |
| **Amazon SES** | Automated Email Service. The Lambda Worker calls SES to send 4-week workout plans or schedule reminders to users. |

According to the team assignment, I heavily focused on developing the **Backend (.NET 10)** for these workflows:
- **SQS Queue & Lambda Worker:** Developed `SqsMessageQueueService` for the API to publish messages, and `SqsHandler` for Lambda to consume messages and invoke AWS SES.
- **Workout Logging:** Built `WorkoutSessionLogsController` to record the actual gym session results of the User.
- **XP & Streak Calculation (Gamification):** Engineered the logic to award Experience Points (XP) and calculate consecutive workout days (Streak) immediately after a successful workout log.
- **DynamoDB Progress:** Configured and interacted with the DynamoDB table (storing Progress) to continuously update user achievements.

#### Backend Implementation Details:

1. [Asynchronous Architecture with SQS & Lambda Worker](5.5.1-sqs-worker/)
2. [Workout Log API & Gamification System (XP/Streak)](5.5.2-gamification/)
