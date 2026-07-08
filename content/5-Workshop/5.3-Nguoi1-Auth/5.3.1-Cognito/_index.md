---
title : "Amazon Cognito & Auth"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

In the PeriodIQ project, **Amazon Cognito** acts as the central "gatekeeper", handling the entire identity and authentication process.

#### User Pool Setup on AWS
I created a Cognito User Pool with the following key configurations:
- Require Email authentication via OTP (One-Time Password).
- The App Client is configured for a Single Page Application (React) without an App Client Secret.
- Created 2 default User Pool Groups:
  - **`Admin`**: Reserved for system administrators.
  - **`Users`**: (Default) General users.

*(Cognito User Pool Setup Interface)*
![Cognito User Pool Setup](/images/5-Workshop/5.3-Nguoi1-Auth/01-cognito-user-pool.png)

*(Admin Group Creation Interface)*
![Cognito Groups](/images/5-Workshop/5.3-Nguoi1-Auth/02-cognito-groups.png)

---

#### Backend Integration (.NET 10)

For the Backend to understand and accept Tokens issued by Cognito, I set up **JwtBearer Authentication** in the `Program.cs` file.

Specifically, to ensure ASP.NET Core's `Role-Based Access Control` works flawlessly with Cognito Groups, I configured `RoleClaimType` to directly map to the AWS `"cognito:groups"` claim:

```csharp
builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = cognitoAuthority;
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            ValidateIssuer           = true,
            ValidIssuer              = cognitoAuthority,
            ValidAudience            = cognitoAudience,
            ValidateAudience         = false, // Disabled to allow both IdToken & AccessToken
            ValidateLifetime         = true,
            RoleClaimType            = "cognito:groups" // <-- Crucial Mapping
        }; 
    });
```

Thanks to this configuration, any Controller requiring admin privileges can simply use the `[Authorize(Roles = "Admin")]` attribute. If a user lacks a valid token or doesn't belong to the `Admin` group, ASP.NET Core immediately blocks the request and returns a `403 Forbidden` status code before it reaches the lower layers.

---

#### Frontend Implementation (React + Vite)

On the Frontend, I use the `amazon-cognito-identity-js` library to communicate directly with AWS. I wrote a wrapper file, `authService.js`, to handle all tasks: Sign Up, Login, Forgot Password, and fetching Refresh Tokens.

**Fixing the Race Condition Error (Session Synchronization):**
When React reloads the page in Strict Mode, reading Session info and extracting role groups sometimes causes a race condition, leading to the Admin UI disappearing. I resolved this by reading the `cognito:groups` claim directly from the Token Payload as soon as the Session is retrieved:

```javascript
// Excerpt from AuthContext.jsx
getCurrentSession()
  .then(async (result) => {
    if (result) {
      const attrs = await getUserAttributes();
      // Read directly from the ID Token payload for 100% synchronization
      const payload = result.session.getIdToken().payload;
      const userGroups = payload['cognito:groups'] || [];
      
      setUser(attrs);
      setGroups(userGroups);
      setIsAuth(true);
    }
  });

// Lightweight permission check function, reusable across components
const isAdmin = groups.includes('Admin');
```

Finally, all Backend API calls are configured via an **Axios Interceptor** that automatically attaches `Authorization: Bearer <Token>` to the Header, providing a seamless experience for end-users.
