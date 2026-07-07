---
title: "Event 1"
date: 2026-05-09
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---


# Summary Report: "FCAJ Community Day"

**Date & Time:** 09:00, May 9, 2026

### Event Objectives

- Look at how the arrival of AI is quietly rewriting the everyday habits, tools, and workflows developers rely on to learn, write prompts, onboard onto a codebase, and ship software
- Dig into concrete, repeatable techniques for pushing LLM output quality up, instead of the usual trial-and-error prompt tweaking
- Unpack what "AI-ready" actually means for a fresh graduate stepping into today's hiring market
- Walk through a structured, agent-based way of doing AI-assisted software development, as an alternative to one long freeform chat

### Speakers

- **Huynh Hoang Long** – Admin of FCAJ - *Addicted to learning like you're addicted to Social Media*
- **Nguyen Tuan Thinh** – DevOps/Cloud Engineer - *Automated Prompt Engineering: Enhancing LLM Output Quality*
- **Khang** – Solution Architect - *AI-Ready Freshers*
- **Thao** - Software development - *BMAD Method*

### Key Highlights

#### Addicted to Learning Like You're Addicted to Social Media

Huynh Hoang Long opened by pointing out something most of us already sense but rarely name: social media apps aren't accidentally addictive - they're engineered around a tight loop of **trigger → action → variable reward → investment**, and there's no reason that same loop has to be reserved for scrolling. It can just as easily be pointed at learning.

His core argument was that willpower is a bad long-term strategy, because it's inconsistent by nature - some days you have it, some days you don't, and a habit that only survives on high-willpower days won't survive at all. What actually holds up is the environment and the feedback loop around the behavior, not the discipline of the person doing it.

He then walked through the mechanics anyone can borrow: a daily trigger (a reminder, a streak counter), an action kept deliberately small so it's repeatable without friction, a reward that varies enough to stay interesting (a new insight, a small win, recognition from the community), and an "investment" step - taking notes, writing something public - that quietly locks you further into the loop next time. The takeaway he left us with: treat consistent learning as a **habit-design problem** you engineer, not a discipline problem you white-knuckle through.

#### Automated Prompt Engineering: Enhancing LLM Output Quality

Nguyen Tuan Thinh's session tackled a pain point most of us have felt without naming it properly: hand-tuning prompts by trial and error simply doesn't scale, and it's unsettling how much a small wording change can swing both output quality and consistency.

His proposed fix was to stop treating prompting as an art and start treating it as an engineering process: define a clear success metric up front, build a small evaluation dataset, generate several candidate prompts, score every candidate against that dataset (including using an LLM itself as a judge), and keep iterating on whichever candidate wins. Framed that way, prompt engineering stops being guesswork and becomes a **repeatable, measurable process** - one where you can objectively compare prompt version A against version B before either one ever reaches production. It also connects directly to a theme that kept resurfacing across the day: you can't fully eliminate LLM variance, but you absolutely can drive it down systematically.

#### AI-Ready Freshers

Khang's talk was aimed squarely at anyone about to graduate into today's job market, and the framing was blunt: AI tools already absorb a large share of the repetitive, boilerplate work junior engineers used to be hired to do, so "I know how to write a prompt" no longer counts as a differentiator on its own.

What he argued actually separates an AI-ready fresher from everyone else is a combination of things that have nothing to do with prompting skill - **solid fundamentals** (data structures, systems thinking, the ability to debug when something breaks), the habit of critically checking AI-generated output rather than accepting it at face value, and genuine comfort operating inside AI-assisted workflows (Copilot, Amazon Q Developer, and similar tools). His practical advice for anyone building a portfolio: pair AI tooling with real engineering discipline - tests, version control, code review - instead of leaning entirely on "vibe coding" and hoping it holds together.

#### BMAD Method

Thao closed out the talks with a look at **BMAD (Breakthrough Method for Agile AI-Driven Development)**, a way of structuring AI-assisted software work around specialized agent personas - Analyst, PM, Architect, Scrum Master, Developer, QA - each one responsible for a distinct, well-defined slice of the workflow.

Rather than throwing everything at a single general-purpose AI assistant and hoping it juggles planning and execution at once, BMAD deliberately separates the two: each agent produces a structured artifact (a PRD, an architecture doc, a story file) that hands off cleanly to whichever agent comes next. The payoff, as Thao described it, is that AI-driven development stays aligned with agile practice instead of drifting into chaos - better traceability, more consistency, and a team that stays in sync across a project instead of everyone relying on one sprawling, unstructured chat log.

### Key Takeaways

#### Personal Growth Needs System Design, Not Just Willpower

- Consistent learning habits can be engineered the same way a product team engineers an engagement loop.
- Small, repeatable actions with visible feedback consistently beat big, ambitious goals that quietly become unsustainable.

#### Prompting and AI Adoption Need Rigor, Not Just Intuition

- Prompt quality is something you measure and iterate on systematically - not something you tune by feel.
- Being "AI-ready" is mostly a matter of judgment and fundamentals, not a checklist of prompting tricks.

#### Structure Beats Ad-Hoc AI Usage

- BMAD is a concrete demonstration that giving AI agents clearly defined roles and handoffs produces far more consistent results than one unstructured assistant trying to do everything.
- It's really one instance of a bigger pattern: AI performs best inside a well-designed process - it's a poor substitute for having a process at all.

### Applying to Work

- Redesign one of my own personal learning habits around the trigger–action–reward–investment loop - a daily streak, or writing up something small in public.
- Put together a lightweight prompt-evaluation script with a small test set, and run prompts through it before locking them in for real projects.
- Lean on fundamentals-first criteria - not prompting fluency - when mentoring or evaluating junior engineers who work with AI tools.
- Pilot the BMAD Method on a small project, splitting an AI-assisted workflow into distinct Analyst/PM/Architect/Dev/QA roles instead of one long freeform chat.

### Event Experience

Sitting in on **"FCAJ Community Day"** turned out to be less about the AI tools themselves and more about the human habits and processes wrapped around them - which honestly made it more useful than a typical tools-and-features talk. A few things stuck with me afterward:

**Learning reframed as something you design, not something you discipline yourself into.** Huynh Hoang Long's talk stuck with me the most, mainly because of how uncomfortable the comparison was - the exact loop that makes social media hard to put down is the same loop you can point at your own learning, just aimed in a better direction.

**Prompt engineering treated like an actual engineering discipline.** Nguyen Tuan Thinh made a convincing case that prompt quality doesn't have to live in the realm of guesswork - it can be measured, tested, and iterated on the same way any other engineering artifact is.

**A reality check on what "AI-ready" really means for new grads.** Khang's session was a useful gut-check: prompting skill by itself won't make a fresher stand out; fundamentals and the judgment to question AI output will.

**A structured alternative to the endless AI chat.** Thao's walkthrough of the **BMAD Method** landed well because it directly addressed something a lot of us have quietly struggled with - one long, unstructured AI conversation just doesn't hold up once a project gets past a certain size.

**Lessons learned overall:**
- Both learning and prompting improve the moment you start treating them as systems to design, rather than things to just try harder at.
- AI tools amplify whatever process they're plugged into - so investing in the process pays off more than chasing a better tool.
- "AI-ready" turned out to be a mindset and a skill set, not just familiarity with a chat window.

#### Some event photos

![FCAJ Community Day - Event 1 group photo](/images/4-Event/4.1-Event1/picture1.jpg)

> Overall, the event tied together personal learning habits, rigor in prompt engineering, career readiness, and structured AI-driven development under one central idea: getting real value out of AI has less to do with the tool itself, and much more to do with the systems and discipline you build around it.
