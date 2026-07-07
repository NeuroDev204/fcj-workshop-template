---
title : "User Profile & Unit Tests"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

An essential part of my role is Backend API and Frontend Development, specifically handling user information directly. This ensures that immediately after a successful authentication on AWS Cognito, users can store their personal data in DynamoDB.

#### 1. Building the UserProfilesController
I wrote the profile management Endpoints (`/api/userprofiles/me`) based on strict security principles:
- **Zero-trust input:** The API never trusts the Client to send the `UserId`. Instead, it automatically extracts the identifier from the Token (`ClaimTypes.NameIdentifier` or Cognito's `"sub"`) to ensure Users can only interact with their own profiles.
- Seamlessly handles creation scenarios (POST) right after registration, or updates (PUT) when users modify parameters like Body Weight or Fitness Goals.

#### 2. Unit Tests for Verification
My final responsibility is to ensure this Auth & Profile code runs perfectly across all scenarios. I applied `xUnit` and `Moq` libraries to write comprehensive Unit Tests for the `UserProfilesController`.

Notably, I mocked a **ClaimsPrincipal** identical to the Cognito JWT structure to inject into the test context (Controller Context):
```csharp
private void SetupUserClaims(string sub, string email, string[]? groups = null)
{
    var claims = new List<Claim> { new("sub", sub), new("email", email) };
    if (groups != null)
    {
        foreach (var g in groups)
            claims.Add(new("cognito:groups", g));
    }

    var identity  = new ClaimsIdentity(claims, "TestAuth");
    var principal = new ClaimsPrincipal(identity);
    _controller.ControllerContext = new ControllerContext {
        HttpContext = new DefaultHttpContext { User = principal }
    };
}
```

Result: All **15 Test Cases** within my Controller passed with a **100% SUCCESS RATE**, covering everything from happy paths (`200 OK`) to error edge cases (`409 Conflict`, `404 Not Found`).

*(Green Unit Test Results Board)*
![Unit Tests Results](/images/5-Workshop/5.3-Nguoi1-Auth/06-unit-tests-passed.png)
