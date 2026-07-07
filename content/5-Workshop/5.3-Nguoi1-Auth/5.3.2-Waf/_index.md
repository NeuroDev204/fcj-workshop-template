---
title : "AWS WAF (Web ACL)"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

To protect authentication endpoints from automated attacks or data breaches, I deployed **AWS WAF (Web Application Firewall)**.

#### Architectural Change: REGIONAL Scope

In the original design, WAF was intended to be attached to CloudFront (requiring creation in the `us-east-1` region). However, to optimize resource management centrally in `ap-southeast-1` and to protect directly at the Authentication layer rather than the Edge layer, I changed the WAF scope to **REGIONAL**.

This Regional WebACL is directly attached to the **Amazon Cognito User Pool** via `AWS::WAFv2::WebACLAssociation`. This change provides higher reliability, blocking attacks aimed directly at the authentication API (e.g., Brute force, Credential stuffing) that might bypass CloudFront.

#### Security Rule Sets

I configured 3 primary security rule groups:

1. **Bot Control (`AWSManagedRulesBotControlRuleSet`)**
   - Blocks malicious bots, scrapers, and automated scripts probing for user information.
   
2. **Core Rule Set (`AWSManagedRulesCommonRuleSet`)**
   - Includes rule sets to protect against common web vulnerabilities (OWASP Top 10) such as SQL Injection (SQLi) and Cross-Site Scripting (XSS).

3. **Rate Limiting (Custom)**
   - Aimed at preventing localized DDoS attacks or Password Brute-forcing.
   - I set a limit of **2000 requests per 5 minutes** from the same IP address. If exceeded, subsequent requests from that IP are entirely blocked until the cycle ends.

*(Illustration: WebACL Rules Configuration on AWS WAF)*
![WAF WebACL Rules](/images/5-Workshop/5.3-Nguoi1-Auth/03-waf-rules.png)

*(Illustration: Attaching WAF to Cognito User Pool)*
![WAF attached to Cognito](/images/5-Workshop/5.3-Nguoi1-Auth/04-waf-cognito-assoc.png)
