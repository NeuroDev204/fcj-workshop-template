---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# SUBDOMAIN TAKEOVER: WHEN A FORGOTTEN DNS RECORD BECOMES AN ATTACK VECTOR

While digging into Cloud Security, I came across a pretty interesting article from the AWS Security Blog about an attack technique called **Subdomain Takeover**.

What stood out to me is that this technique doesn't exploit any vulnerability in AWS or in an application — it comes from a very common operational issue: DNS records and cloud resources falling out of sync.

## The problem

When deploying an application or a marketing campaign on AWS, we typically create resources such as Amazon S3, CloudFront, or Elastic Beanstalk, then configure DNS so a company subdomain points to those resources.

Example:

```
api.company.com → my-bucket.s3.amazonaws.com
```

Once the project ends, the AWS resource is deleted to save cost. In many cases, however, the corresponding DNS record is never cleaned up.

The subdomain still exists, but it now points to a resource that is no longer under the organization's control. AWS calls this a **Dangling DNS Record**.

## How the attack scenario plays out

According to AWS, attackers typically use automated scanning tools to search the internet for these kinds of dangling DNS records.

Once they find a DNS record pointing to a deleted resource, they can re-register that same resource if the service allows name reuse. For example, with Amazon S3, if a bucket has been deleted and its name is not yet taken, an attacker can create a new bucket with the exact same name.

As a result, all traffic to the company's subdomain gets redirected to a resource controlled by the attacker.

## Risks that can arise

The AWS article highlights several notable impacts:

* Serving fake/malicious content directly on the company's own subdomain.
* Running phishing campaigns through a domain name with high trust.
* Damaging brand reputation and customer experience.
* Increasing the risk of further security incidents if not detected in time.

## AWS's proposed solution

What I find quite good is that AWS doesn't just describe the risk — it also proposes an approach to detect and remediate dangling DNS records.

The proposed solution uses AWS Config together with AWS Lambda to cross-reference DNS records in Route 53 against the actual list of resources in the AWS account. When a DNS record is found pointing to a resource that no longer exists, the system can:

* Flag the configuration as Non-Compliant.
* Send an alert to AWS Security Hub.
* Notify the operations team via Amazon SNS.

This allows risks to be detected early, before they can be exploited.

![Dangling DNS Record detection architecture](/images/3-Blog/blog1.png)

### AWS services involved in the architecture

* AWS Config
* AWS Lambda
* Amazon Route 53
* AWS Security Hub
* Amazon SNS
* Amazon CloudWatch

## What I took away from this

The most interesting takeaway for me is that Subdomain Takeover is not an AWS vulnerability. The real problem lies in operational processes and resource lifecycle management. A resource may have been deleted, but if its DNS record still exists, the attack surface hasn't actually gone away.

This case study also shows that:

* Security isn't just about code or system configuration.
* DNS is also an important part of the attack surface.
* Automating configuration checks significantly reduces manual mistakes.
* Good operational process can matter just as much as sophisticated security solutions.

**Reference:** [AWS Security Blog - Threat tactic spotlight: subdomain takeover](https://aws.amazon.com/blogs/security/threat-tactic-spotlight-subdomain-takeover/)

**Facebook:** [AWS Study Group FCJ](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2187399542025006/)