---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section will list and introduce the blogs you have posted to [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj). For example:

###  [Blog 1 - SESSION POLICIES IN AMAZON EKS POD IDENTITY](3.1-Blog1/)
This blog introduces the newly added session policies feature in Amazon EKS Pod Identity, which allows you to narrow IAM permissions flexibly and precisely for each pod without needing to create multiple separate IAM roles. This is an important step forward that helps apply the principle of least privilege more effectively in large-scale Kubernetes environments.

###  [Blog 2 - A Hybrid Multi-Tenant Architecture for Stateful Services on AWS](3.2-Blog2/)
This blog covers a case study from the AWS Architecture Blog on Amazon Ads' ad-serving system, which handles millions of requests per second. It breaks down why a pure "dedicated account per tenant" or a "share everything" model both fall short for stateful services, and walks through the hybrid "cellular" architecture that balances tenant isolation with operational efficiency at scale.

###  [Blog 3 - Scaling Serverless to 1 Million Lambda Functions](3.3-Blog3/)
This blog covers a case study from the AWS Architecture Blog on scaling a SaaS platform to 1 million Lambda functions across thousands of AWS accounts. It walks through the operational nightmares that emerge only at massive scale — self-inflicted DDoS from synchronized cron jobs, an observability bill that doubled cloud costs, CloudFormation StackSets throttling, and hidden Scale-to-Zero waste — and the architectural fixes (jitter, tiered observability, true scale-to-zero, mono-repo) that solved them.