---
title : "AWS WAF (Web ACL)"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

To protect authentication endpoints from automated attacks or data breaches, I deployed **AWS WAF (Web Application Firewall)**.

#### Edge Protection with Global Scope (CLOUDFRONT)

Strictly adhering to the standard architecture design of the PeriodIQ project, I deployed the WAF WebACL with the **CLOUDFRONT Scope** (which must be provisioned in the `us-east-1` region).

This WebACL is attached directly to the **Amazon CloudFront Distribution** via the `WebACLId` property. This strategy creates a massive "shield" providing comprehensive system protection at the Edge layer, effectively blocking malicious traffic (DDoS, Bots, SQLi) at the perimeter before it can reach the React Frontend (S3) or the API Gateway Backend.

#### Security Rule Sets

I configured 3 primary security rule groups:

1. **Bot Control (`AWSManagedRulesBotControlRuleSet`)**
   - Blocks malicious bots, scrapers, and automated scripts probing for user information.
   
2. **Core Rule Set (`AWSManagedRulesCommonRuleSet`)**
   - Includes rule sets to protect against common web vulnerabilities (OWASP Top 10) such as SQL Injection (SQLi) and Cross-Site Scripting (XSS).

3. **Rate Limiting (Custom)**
   - Aimed at preventing localized DDoS attacks or Password Brute-forcing.
   - I set a limit of **2000 requests per 5 minutes** from the same IP address. If exceeded, subsequent requests from that IP are entirely blocked until the cycle ends.

*(WebACL Rules Configuration on AWS WAF)*
![WAF WebACL Rules](/images/5-Workshop/5.3-Nguoi1-Auth/03-waf-rules.png)

*(Attaching WAF to CloudFront Distribution)*
![WAF attached to CloudFront](/images/5-Workshop/5.3-Nguoi1-Auth/04-waf-cloudfront-assoc.png)
