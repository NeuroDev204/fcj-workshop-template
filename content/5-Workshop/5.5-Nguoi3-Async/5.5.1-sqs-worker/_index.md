---
title : "Asynchronous Architecture with SQS, Lambda & SES"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

In modern software system design, especially within Microservices and Serverless architectures, one of the most common "anti-patterns" is performing time-consuming tasks (like generating PDFs, processing images, or communicating with third-party Email services) right inside the synchronous processing thread of an API. This not only blocks the server's thread but also increases the overall request latency, leading to a poor user experience as they are forced to wait for a loading spinner.

To solve this problem for PeriodIQ, I applied an **Event-Driven Architecture**, leveraging the AWS trio: **Amazon SQS**, **AWS Lambda**, and **Amazon SES** to completely decouple the email notification process from the main API.

### 1. Initializing and Pushing Messages to the Queue (Amazon SQS)

**Amazon SQS (Simple Queue Service)** acts as a highly durable buffer. When users finalize their settings and click "Generate Plan" on the Frontend (provided they have enabled the Email Notification option), the Backend API handles data analysis and saves the plan to DynamoDB. 

![Email Notification Settings](/images/5-Workshop/5.5-Nguoi3-Async/08-notification-setting.jpg)

Instead of sending an email immediately, the API simply packages an event (e.g., `WorkoutPlanCreatedEvent` containing `UserId` and `PlanId`) and pushes it into the SQS Queue.

The Queue configuration process on the AWS Console was carefully executed to ensure high availability:

![Initialize SQS Queue](/images/5-Workshop/5.5-Nguoi3-Async/01-sqs-create-queue.jpg)

Below is the list of SQS Queues I established to receive various system messages:

![SQS Queue List](/images/5-Workshop/5.5-Nguoi3-Async/02-sqs-list-queues.jpg)

At the Application layer (.NET Backend), I implemented an `IMessageQueueService` interface to encapsulate the interaction logic with the AWS SDK. The code snippet below illustrates how the API pushes data to the Queue in just a few milliseconds:

```csharp
public async Task SendMessageAsync<T>(T message)
{
    var jsonMessage = JsonSerializer.Serialize(message);
    var request = new SendMessageRequest
    {
        QueueUrl = _queueUrl,
        MessageBody = jsonMessage
    };
    await _sqsClient.SendMessageAsync(request);
}
```
Thanks to this mechanism, the API instantly returns an HTTP `200 OK` status to the Frontend. The user can continue browsing or viewing the dashboard right away without realizing the email sending process is still happening in the background.

### 2. Triggering the Background Lambda Worker

SQS itself is just a message container. To "consume" these messages, I built an **AWS Lambda Function** acting as an independent Worker.

The true power of Serverless architecture shines here: I don't need to rent a 24/7 EC2 server just to wait for messages from SQS. Instead, I configured an **SQS Event Source Mapping** (SQS Trigger) directly attached to Lambda. Whenever a new message falls into the queue, the AWS system automatically wakes up (invokes) the Lambda Worker to process it. If thousands of messages pour in simultaneously, Lambda automatically scales out by spinning up thousands of parallel instances to neatly resolve the queue.

Details of the SQS configuration and Trigger parameters (like Batch Size) are shown here:

![Lambda SQS Trigger Configuration](/images/5-Workshop/5.5-Nguoi3-Async/13-lambda-sqs-trigger.jpg)
![SQS Configuration Details](/images/5-Workshop/5.5-Nguoi3-Async/03-sqs-queue-detail.jpg)

The Lambda Worker code (written in C#) loops through each record in SQS, deserializes the JSON, and queries the DynamoDB database to retrieve the User's actual Email address:

```csharp
public async Task FunctionHandler(SQSEvent evnt, ILambdaContext context)
{
    foreach (var message in evnt.Records)
    {
        // 1. Parse JSON data from SQS Message Body
        var eventData = JsonSerializer.Deserialize<WorkoutPlanEvent>(message.Body);
        
        // 2. Retrieve details (Email, Name) from DynamoDB
        var user = await _dynamoContext.LoadAsync<UserProfile>(eventData.UserId);
        
        // 3. Prepare to call SES to send email...
    }
}
```

### 3. Integrating Amazon SES for Automated Emailing

The final step in this chain of actions is communicating with **Amazon SES (Simple Email Service)**. This is AWS's large-scale email sending solution, featuring powerful anti-spam capabilities and extremely high deliverability rates.

Before the system could send emails on behalf of PeriodIQ, I had to perform the identity verification process (Domain/Email Verification) on SES to comply with email anti-spoofing policies (DKIM, SPF):

![Initialize SES Identity](/images/5-Workshop/5.5-Nguoi3-Async/04-ses-create-identity.jpg)
![Verify and Configure Email](/images/5-Workshop/5.5-Nguoi3-Async/05-ses-setup-email.jpg)

Once set up, all valid identities are managed and authorized by AWS:

![List of Verified SES Identities](/images/5-Workshop/5.5-Nguoi3-Async/06-ses-list-identities.jpg)

Back to the Lambda Worker, once it has the User's Email address, the Worker loads a professional HTML Template (containing the beautiful UI of the 4-week workout plan) and sends the request via the SES SDK:

```csharp
var sendRequest = new SendEmailRequest
{
    Source = "noreply@periodiq.com", // Email verified on SES
    Destination = new Destination { ToAddresses = new List<string> { user.Email } },
    Message = new Message
    {
        Subject = new Content(" Your 4-week workout plan is ready! - PeriodIQ"),
        Body = new Body { Html = new Content(htmlTemplate) }
    }
};
await _sesClient.SendEmailAsync(sendRequest);
```

### 4. Ensuring Quality with Unit Tests

To ensure the SQS Handler system works flawlessly and prevents future regressions, I wrote robust Unit Tests using xUnit and Moq. The testing process involved mocking the SQS Event, DynamoDB Context, and SES Client to thoroughly verify the JSON deserialization logic and the email sending API calls.

All test cases **Passed** brilliantly in just 1.2 seconds:

![SQS Handler Unit Test Results](/images/5-Workshop/5.5-Nguoi3-Async/07-sqs-unit-tests-passed.jpg)

### Summary
With the **API -> SQS -> Lambda -> SES** architecture, PeriodIQ gains 3 massive benefits:
1. **Performance:** A buttery-smooth user experience, never blocked by network operations.
2. **Fault Tolerance:** If the Email service (SES) temporarily fails, Lambda throws an error and the message safely remains in SQS (or gets pushed to a Dead-Letter Queue). No data is ever lost.
3. **Cost-effective:** A 100% Serverless architecture; you only pay for each SQS message and every millisecond of Lambda execution. When nobody is using it, the operational cost is $0.
