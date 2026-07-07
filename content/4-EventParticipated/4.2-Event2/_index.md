---
title: "Event 2"
date: 2026-05-23
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---


# Summary Report: "FCAJ Community Day"

**Date & Time:** 09:00, May 23, 2026

### Event Objectives

- Pull together AWS community builders, working engineers, and students to trade hands-on experience across AI, cloud architecture, and how modern applications actually get shipped
- Get into the practical, sometimes uncomfortable side of building with LLMs - from their reliability quirks to the discipline of context engineering
- Put real production and hackathon case studies from community members on stage, not just polished marketing examples
- Take a close look at Amazon CloudFront as a foundation for cost predictability, security, and edge performance

### Speakers

- **Duc Dao** – Solution Architect, Cloud Kinetics — *Non-Determinism of "Deterministic" LLM Settings*
- **Vy Lam** – Senior Business Systems Analyst, VPBank — *Enterprise-Grade Multi-Agent System: The Case of Startup Credit Scoring*
- **Pham Ng Hai Anh** – AWS Community Builder, G-AsiaPacific Vietnam — *Friendly AI Assistant w/ Amazon Quick*
- **Nguyen Tuan Thinh** – DevOps Engineer, First Cloud AI Journey — *From Edge to Origin: CloudFront as Your Foundation*
- **Tinh Truong** – Platform Engineer, GoTymeX — *Context Is Everything*
- **Team VIB** – LotusHacks 2026 finalists — *UTMorpho: Building UTMorpho from Idea to Reality*

### Key Highlights

#### Non-Determinism of "Deterministic" LLM Settings

Duc Dao opened with a finding that quietly undermines something most of us take for granted: setting `temperature=0` is supposed to guarantee the same output every time, but research from Atil et al. (2024) found that **not a single model tested actually produced consistent output** across every task. Accuracy swung by as much as 15%, and the gap between the best and worst runs reached 70% - numbers that are hard to square with the word "deterministic."

The root cause turned out to be more mundane than a model quirk: **floating-point arithmetic on GPUs isn't associative**, and inference requests get processed in **batches**, so the exact result for your request quietly depends on whatever else happens to be sitting in the same batch at that moment. Duc's suggested mitigations were pragmatic rather than exotic - running the same request multiple times and taking a majority vote, constraining outputs to a structured format, self-hosting a model when you need full control over the inference stack, and, maybe most importantly, designing all downstream logic to treat LLM output as **probabilistic rather than deterministic** by default. One small but useful tip he shared: `temp=0.1` is often a safer choice than a hard `temp=0`, since it avoids the model getting stuck cycling through repetitive loops.

#### Enterprise-Grade Multi-Agent System — Startup Credit Scoring

Vy Lam's talk started from a mismatch that's easy to miss until you're staring right at it: traditional credit-scoring models are built around three-plus years of financial history and standardized reporting, which is exactly what a startup doesn't have - instead you get 6 to 18 months of hypergrowth and messy, unstructured data.

A single AI agent, she argued, buckles under that kind of problem - it's asked to hold multi-domain expertise, juggle conflicting objectives, and stay auditable all at once, and while it technically "works," the output isn't reliable enough for a decision with real financial stakes. Her proposed alternative was a **virtual credit committee**: a set of specialized agents (Financial, Market, Team, Risk, Compliance) that collaborate under a Manager agent to arrive at a consensus score, a risk rating, a confidence level, and a full audit trail behind every decision. She framed enterprise-grade thinking as resting on six pillars - **security, data governance, network, operations, human factors, and compliance** - and backed it with numbers: 95% faster processing and a measured 200-300% three-year ROI.

#### Friendly AI Assistant with Amazon Quick

Pham Ng Hai Anh's session zeroed in on a familiar frustration for business users: a lot of their day is eaten by repetitive tasks that require pulling and cross-referencing information scattered across multiple systems.

His pitch for **Amazon Quick Suite** was that it collapses this into one governed, secure layer - combining company data, general world knowledge, more than 40 data connectors, and agentic capabilities (automation flows, dashboards, chat, research) behind a single set of guardrails and access controls, rather than stitching together a pile of disconnected tools. The demo that landed best was a PM assistant that automatically drafted meeting minutes, emailed the relevant stakeholders, and scheduled the follow-up meeting on its own.

#### From Edge to Origin: CloudFront as Your Foundation

Nguyen Tuan Thinh's talk started with a problem anyone who's run a public-facing service has felt: usage-based CDN billing is notoriously hard to forecast, and a monthly bill can swing anywhere from three cents to a hundred-thousand-dollar spike the moment traffic goes viral or you get hit with an attack.

He walked through AWS's answer to that unpredictability - new **fixed-price CDN and security plans** (launching November 2025) that bundle CloudFront, WAF, DDoS protection, Route 53, and CloudWatch logging into one predictable monthly price instead of a usage-based surprise. Beyond pricing, he made the case that CloudFront meaningfully improves four things at once: **cost** (compression, free data transfer from AWS origins), **security** (TLS 1.3/mTLS, Origin Shield/OAC/VPC origins to keep your origin hidden, signed URLs), **resiliency** (multi-layer caching, origin failover, graceful degradation under stress), and **performance** (HTTP/3, persistent backbone connections, compute pushed out to the edge).

#### Context Is Everything

Tinh Truong's talk reframed a complaint most of us have made at some point - "the AI gave me a bad answer" - by arguing that the real culprit is almost always **poor context, not a weak model**. An AI can't infer what you actually want or read your mind; it can only work with what you hand it.

He listed the usual failure modes: dumping everything into a single chat message and hoping the model figures out what matters (what he called the "internet puller" problem), restating information the model already has access to, and never stating a goal, constraints, or what success even looks like. He framed the evolution of AI usage as a progression - from one-off **prompts**, to grounded **context**, to persistent **memory** (a store → retrieve → generate → learn loop) - and used a "Second AI Brain" case study to illustrate what that looks like in practice. His practical takeaway was a simple four-part checklist to run through before asking AI anything: **goal, relevant information, constraints, success criteria**.

#### UTMorpho — Building a Product in 36 Hours (LotusHacks 2026)

Team VIB closed out the day with a candid retrospective on their LotusHacks 2026 run - starting from what they described as "head empty" at sign-up and ending with a shipped product, UTMorpho, 36 hours later.

They didn't sugarcoat the rough patches: AI overgenerating far more than they needed, hitting token limits at inconvenient moments, and burnout creeping in right before the pitch. What got them through was less about clever prompting and more about focused editing and tighter team communication. Their takeaway landed as much on team dynamics as on technology: **real frustration produces real ideas**, but generating more ideas isn't automatically better - endurance and staying in sync as a team matter just as much as the tech stack underneath.

### Key Takeaways

#### AI Reliability Is a Design Problem, Not Just a Model Problem

- Determinism settings like `temp=0` reduce randomness but don't erase it - systems need to be built to tolerate variance, not assume it away.
- For decisions that are high-stakes, span multiple domains, or need to be auditable, multi-agent architectures consistently outperform a single overloaded agent.

#### Context Engineering Is Becoming a Core Skill

- Better context - not simply more of it - is what actually drives better AI output.
- The shift from prompting, to context systems, to long-term memory is shaping up to be the next real skill gap developers need to close.

#### Infrastructure Foundations Still Matter

- CloudFront's edge capabilities (security, predictable cost, performance) remain foundational infrastructure even in a stack that's heavily AI-driven.
- Enterprise-grade thinking - security, governance, compliance - needs to be designed in from day one; bolting it on later doesn't work as well.

### Applying to Work

- Treat LLM output as probabilistic by default - add majority-voting or structured-output constraints to any pipeline currently leaning on `temp=0` for consistency.
- Prototype a multi-agent pattern for any workflow that's currently forced onto a single, overloaded agent.
- Run the four-part context framework (goal, relevant information, constraints, success criteria) before writing prompts for anything that actually matters.
- Evaluate CloudFront's fixed-price plans for any project with unpredictable traffic, to avoid a nasty bill-shock surprise later.
- Try Amazon Quick Suite for repetitive business workflows - meeting notes, stakeholder updates, scheduling - instead of doing them by hand.

### Event Experience

Sitting in on **"FCAJ Community Day"** turned out to be a good vantage point for seeing how differently various corners of the AWS community - enterprise engineers, community builders, hackathon teams - are actually putting AI and cloud fundamentals to work in practice, rather than in theory. A few things stood out to me afterward:

**Learning from practitioners instead of just official docs.** The speakers were sharing first-hand findings - like the LLM determinism research - rather than repeating marketing talking points, and the startup credit-scoring case study made it concrete how a multi-agent design solves real auditability problems in a tightly regulated industry like banking.

**Seeing infrastructure and AI concerns collide in the same conversation.** The CloudFront session was a useful reminder that edge and networking fundamentals haven't gone away just because everyone's roadmap is AI-centric now, and the context-engineering talk reframed prompting as fundamentally a product-experience problem rather than a wording trick you pick up once and reuse forever.

**Getting a look at the builder side of the community, warts and all.** Team VIB's LotusHacks retrospective was a genuinely honest look at what a 36-hour build sprint feels like from the inside - including the parts that didn't go well, not just the polished demo at the end. It drove home that team sync and endurance matter just as much as raw technical skill once you're under real time pressure.

**What I'm taking away from all of it:** reliability in AI systems has to be engineered on purpose - it's not something you get for free from a temperature setting. Good context design and good infrastructure design turn out to follow the exact same principle: give the system precisely what it needs, no more and no less. And events like this one keep surfacing practical lessons that are genuinely hard to find written down anywhere in official documentation.
