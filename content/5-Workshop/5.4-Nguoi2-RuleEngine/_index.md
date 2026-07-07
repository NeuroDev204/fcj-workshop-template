---
title : "Trần Anh Tài - Rule Engine & Workout Generation"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

This is my role in the PeriodIQ project. I am responsible for the workout plan generation flow, from user input to the generated 4-week training program.

My scope uses three core AWS services:

| AWS Service | Role |
|---|---|
| **Lambda API Handler** | Receives authenticated user requests and exposes the workout generation endpoint |
| **Lambda Rule Engine** | Runs the Volume, Conflict, and Progression rules to generate a 4-week periodized plan |
| **Amazon S3** | Hosts the React + Vite SPA that contains the Workout Plan user interface |

Additionally, I built the **Workout Plan product flow** where a user can enter training parameters, generate a plan, view weekly progression, and inspect exercise targets such as sets, reps, RPE, rest time, and target weight.

#### Content

1. [Architecture & AWS Services](5.4.1-architecture-and-aws-services/)
2. [Workout Plan Product Flow](5.4.2-workout-plan-product-flow/)
