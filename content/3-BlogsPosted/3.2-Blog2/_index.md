---
title: "Blog 2"
date: 2026-07-12
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# A HYBRID MULTI-TENANT ARCHITECTURE FOR STATEFUL SERVICES ON AWS

Today I want to share a pretty interesting case study from the AWS Architecture Blog about multi-tenant architecture for stateful services (services that keep state in memory) — specifically Amazon Ads' ad-serving system, which handles millions of requests per second.

What I find interesting is that this article doesn't go the "pure multi-tenant with a dedicated account per tenant" route, nor the "share everything" route — instead it proposes a hybrid model that balances isolation with operational efficiency.

## The starting problem: the "cellular" architecture

Previously, the Amazon Ads team used a model where each tenant (customer) got its own AWS account, along with a dedicated ALB and ECS cluster. This gave great isolation, but created a long list of operational problems:

* **Scaling problem:** just 18 clients across 4 Regions already required 181 separate targets — each client meant its own set of accounts, VPCs, load balancers, IAM roles, and downstream service connections.
* **Efficiency problem:** servers sat idle more than 98% of the time, with average CPU utilization of only 3% and RAM at 19%. In other words, paying for massive infrastructure that went unused most of the time.
* **Onboarding problem:** adding a new client took about 52 days — just creating the AWS account, setting up VPC/network, configuring IAM, and integrating downstream services ate up nearly two months.
* **Scale-out problem:** every time traffic grew or a new client came on, the only option was to stand up an entirely new "cell" and migrate — making it impossible to run multiple large (tier-1) events at the same time.
* **"Noisy neighbor" problem:** even with isolation efforts, tenants sharing infrastructure still affected each other's performance.

## Why does each tenant need its own compute?

This is the key point: the ad-serving system is stateful — it loads and keeps each tenant's data in memory instead of querying a database on every request, in order to optimize speed. But because of this, if two tenants share the same cluster, their in-memory data competes for the same heap space. A tenant with a large dataset can trigger an out-of-memory condition and affect the tenant next to it as well.

So the problem becomes: how do you keep isolation at the cluster level while cutting the operational overhead of the old cellular model?

## The solution: a 3-tier layered architecture (Tier – Cell – Infra Group)

The bright idea in this design is separating "shared infrastructure" from "tenant-dedicated infrastructure":

* **Tier:** the layer that classifies tenants by traffic profile (High TPS, Standard TPS, Low TPS).
* **Cell:** the boundary of an AWS account — the unit of horizontal scale-out at the account level.
* **Infra group:** an independent infrastructure unit inside each cell — consisting of 1 VPC, 1 ALB, multiple ECS clusters (one dedicated cluster per tenant), IAM roles, and monitoring.

This 3-layer design addresses exactly two different types of AWS quotas: the target group limit on a single ALB, and the ENI/VPC endpoint limit on a single account. Whichever limit you hit, you add the corresponding unit (an infra group or a cell) instead of tearing everything down and rebuilding it.

![Hybrid multi-tenant architecture for stateful services](/images/3-Blog/blog2.png)

### Key techniques used

* **Route 53 Weighted Routing:** a single fixed DNS endpoint for the whole tier, distributing traffic across multiple ALBs in different accounts. Scaling further just means adding a new weighted record — tenants never need to change their DNS.
* **Per-tenant ALB listener rules:** each ALB routes requests to the correct tenant's ECS cluster based on path or header, with a limit of around 50 tenants per infra group (due to the 100 target-group quota and 5 target-groups-per-listener-rule limit).
* **Dedicated ECS cluster per tenant:** ensures one tenant's in-memory state never collides with another's, and the 5,000 tasks/service quota applies only to that specific tenant.
* **Shared AWS PrivateLink:** instead of each tenant setting up its own connection to downstream services, VPC endpoints are pre-provisioned at the tier level — new tenants automatically inherit the connection with no extra configuration, cutting network setup steps by 80%.

## Measured results

* Tenant onboarding time: from 52 days down to 7 days (an 86% reduction)
* Infrastructure setup steps per tenant: reduced by 80%
* Engineering effort per onboarding: reduced by 80%
* Feature release time: from 2-3 days down to 1 day
* A single account can now serve up to 100 tenants while still maintaining cluster-level isolation

## What I took away from this

The thing I found most compelling in this case study is the core design decision: decoupling dependency setup from the tenant onboarding process itself. Pre-wiring PrivateLink, IAM roles, and cache endpoints at tier-creation time — rather than at tenant-addition time — is what turns onboarding from "a multi-week infrastructure project" into "a config change."

This case study also shows that:

* Multi-tenancy doesn't have to be a choice between "full isolation" (one account per tenant) and "share everything" — a hybrid design can strike the right balance.
* For stateful systems (holding data in RAM), isolation at the compute/cluster level matters far more than isolation at the account level.
* A good layered architecture gives you multiple independent scaling levers, instead of just one way to grow.
* Investing in pre-provisioning and automation up front saves enormous effort later, even though it takes more design work initially.

**Reference:** [AWS Architecture Blog - Building hybrid multi-tenant architecture for stateful services on AWS](https://aws.amazon.com/blogs/architecture/building-hybrid-multi-tenant-architecture-for-stateful-services-on-aws/)

**Facebook:** [AWS Study Group FCJ](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2201873483910945/)