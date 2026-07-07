---
title : "Workout Log API & Gamification System"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

For a Fitness application to retain users over the long term, merely providing workout plans is not enough. Users need a sense of achievement and recognition after every drop of sweat shed in the gym. That is why I designed and directly integrated a **Gamification** system deep into the Backend's workout logging workflow.

This system revolves around two core metrics:
- **XP (Experience Points):** Reward points accumulated after every completed workout session.
- **Streak:** The number of consecutive workout days without skipping.

![Workout Logging UI](/images/5-Workshop/5.5-Nguoi3-Async/09-workout-logging.jpg)
![Workout Detail (Part 1)](/images/5-Workshop/5.5-Nguoi3-Async/11-workout-detail-1.jpg)
![Workout Detail (Part 2)](/images/5-Workshop/5.5-Nguoi3-Async/12-workout-detail-2.jpg)

### 1. Workout Log API Workflow

When a user completes their final exercise and clicks "Complete Workout" on the React frontend, the Frontend fires an HTTP POST Request containing a comprehensive payload (including the list of exercises, Sets, actual Reps, lifted weights, and RPE level) to the Backend.

At the Core layer of our Clean Architecture, this process is received and handled by the `WorkoutSessionLogService`. This service not only records the raw data into DynamoDB (so the Rule Engine can measure Central Nervous System - CNS fatigue) but also instantly triggers the Gamification calculation algorithm.

![Dashboard XP and blazing Streak flame](/images/5-Workshop/5.5-Nguoi3-Async/10-gamification-dashboard.jpg)

### 2. Gamification Algorithm at the Backend

Instead of relying on delayed cronjobs for calculations, I decided to update the XP and Streak in real-time. This data is stored in the `Progress.cs` Entity within DynamoDB, containing key fields such as `XP`, `CurrentStreak`, `LongestStreak`, and `LastWorkoutDate`.

![Progress Table in DynamoDB](/images/5-Workshop/5.5-Nguoi3-Async/14-dynamodb-progress-table.jpg)

The Gamification calculation algorithm operates on the following rules:
- **Increase Streak:** If the user works out consecutively (the last workout was yesterday) or this is their very first workout, the system adds 1 day to the current Streak.
- **Punish by Resetting Streak:** If the user skips more than 1 day, the current Streak is immediately reset to 1 to enforce discipline.
- **Personal Record:** The system always compares and records the longest consecutive workout sequence (Longest Streak) the user has ever achieved.
- **Award XP:** Regardless of the Streak, every completed workout session guarantees a flat reward of 50 XP to boost motivation.

The code snippet in `WorkoutSessionLogService.cs` cleanly executes this logic:

```csharp
var today = DateTime.UtcNow.Date;
var lastDate = progress.LastWorkoutDate?.Date;

if (lastDate == null || lastDate == today.AddDays(-1))
{
    progress.CurrentStreak += 1;
}
else if (lastDate < today.AddDays(-1))
{
    progress.CurrentStreak = 1;
}

if (progress.CurrentStreak > progress.LongestStreak)
{
    progress.LongestStreak = progress.CurrentStreak;
}

progress.XP += 50; 
progress.LastWorkoutDate = DateTime.UtcNow;

await _dynamoContext.SaveAsync(progress);
```

### 3. Impact on User Experience

Thanks to the seamless processing architecture at the Backend, the moment the API returns an HTTP `200 OK` Status, the latest `Progress` data is already prepared.

When the Frontend automatically navigates the user back to the Dashboard page, the screen instantly increments the new XP number, accompanied by a blazing flame icon representing their current Streak. This Immediate Feedback creates a massive Dopamine hit in the brain, sparking a sense of satisfaction and inadvertently forming a "healthy addiction," providing immense motivation for them to grab their gym bag and return the next day!
