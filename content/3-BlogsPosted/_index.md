---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section lists and introduces the blogs I have posted to [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj).

###  [Blog 1 - Subdomain Takeover: When a Forgotten DNS Record Becomes an Attack Vector](3.1-Blog1/)
This blog covers the Subdomain Takeover attack technique, where a "dangling DNS record" left pointing at a deleted AWS resource (S3, CloudFront, Elastic Beanstalk...) can be re-claimed by an attacker. It walks through how the attack happens and AWS's proposed detection approach using AWS Config + Lambda to cross-check Route 53 records against actual account resources, alerting via Security Hub and SNS.

###  [Blog 2 - Building a Hybrid Multi-Tenant Architecture for Stateful Services on AWS](3.2-Blog2/)
This blog summarizes an AWS Architecture Blog case study on Amazon Ads' ad-serving system, which moved from a costly "one AWS account per tenant" model to a hybrid 3-tier Tier-Cell-Infra Group architecture (Route 53 weighted routing, per-tenant ALB rules, dedicated ECS clusters, shared PrivateLink) - cutting tenant onboarding time from 52 days to 7.

###  [Blog 3 - Session Policies in Amazon EKS Pod Identity](3.3-Blog3/)
This blog introduces the newly added session policies feature in Amazon EKS Pod Identity, which allows you to narrow IAM permissions flexibly and precisely for each pod without needing to create multiple separate IAM roles - an important step toward applying least privilege more effectively in large-scale Kubernetes environments.