# Agentic Discovery — From Keyword Search to Conversational Food Intelligence

**Role:** AI Product Manager · **Company Context:** Delivery Hero (Portfolio Project)
**Live Demo:** https://agentic-discovery-for-food-delivery.vercel.app
**Timeline:** 5 days · **Stack:** React, Claude API, Vercel

---

## The Problem

Food delivery platforms have a discovery problem that nobody talks about.

The search box works perfectly for users who already know what they want. Type "pizza" → get pizza. Type "sushi" → get sushi. Clean, fast, deterministic.

But a significant portion of users don't know what they want. They're tired, hungry, and staring at a blank search box. They know they want something *spicy and vegetarian and not too expensive and delivered before their meeting* — but keyword search can't understand that sentence. It can only match words.

The result: users either abandon the session, or they order something mediocre because they couldn't express what they actually wanted.

I built this prototype to answer one question: what if the search box understood you the way a knowledgeable friend would?

---

## Research & Problem Framing

Before building anything, I mapped the gap between how users think about food and how keyword search works.

Users think in **multi-dimensional intent**: mood + dietary need + price + time + cuisine — simultaneously, in natural language. Keyword search processes one dimension at a time.

The gap shows up in three specific failure modes I identified:

**Failure Mode 1 — The Blank Box Problem**
Users with complex intent (e.g. "something light, vegetarian, under €12") can't express it in a search box. They type "vegetarian" and get overwhelmed by 40 results with no ranking logic that understands their budget or time constraint.

**Failure Mode 2 — Filter Fatigue**
Filter systems exist to solve this, but they require users to translate their natural intent into UI elements. Most users don't complete multi-filter flows. They abandon.

**Failure Mode 3 — Context Blindness**
A user searching at 11pm likely wants something different than the same user at 6pm. Keyword search is context-blind. An agent can infer context.

---

## What I Built

I designed and shipped a working conversational discovery agent with three layers:

### Layer 1 — Natural Language Understanding (Claude API)
When a user types *"I just worked out, need something high protein and fast"* — the system makes a real API call to Claude to extract structured intent. Claude understands that "just worked out" implies high protein, that "fast" implies delivery time constraint, and generates a warm human response.

Claude handles all language work: understanding the query, writing the reaction message, writing the personalised closing recommendation.

### Layer 2 — Deterministic Constraint Scoring (Local Logic)
Once intent is extracted, I deliberately moved constraint matching and restaurant ranking out of the AI layer into deterministic local logic.

This was a conscious product decision. Non-deterministic ranking means I can't explain why Restaurant A ranked above Restaurant B. With a scoring function I control, I can always audit the ranking, debug failures, and explain decisions to stakeholders.

The scoring weights:
- Cuisine match: +60 (strongest signal — user asked for specific food type)
- Budget match: +35 (hard constraint, scaled penalty if violated)
- Delivery time: +30 (time-sensitive constraint)
- Dietary match: +38–50 (safety-critical — never show meat to a vegan)
- Protein/health/comfort flags: +25–42

### Layer 3 — AI Evaluation Panel (PM Monitoring Layer)
The right panel is not visible to users — it's a real-time monitoring layer showing what the agent is actually doing:

- **Intent Match Rate** — did the agent understand the user's real goal?
- **Constraint Satisfaction** — did the top recommendation meet stated requirements?
- **Groundedness Score** — are results from verified data, not hallucinated?
- **Hallucination Rate** — what percentage of output was fabricated?
- **Overall Agent Score** — weighted composite: Intent(35%) + Constraint(40%) + Grounding(25%)

This panel demonstrates something most portfolio demos skip: the measurement layer. Building something that looks good is easy. Knowing when it's failing — and why — is the harder problem I wanted to solve.

---

## Key Product Decisions

**Decision 1: Claude for language, regex for constraints**

Early versions had Claude extract all constraints via JSON. It was unreliable — Claude would sometimes return `cuisine: null` for "I need curry." I moved constraint extraction to a deterministic local parser and kept Claude for what it's genuinely better at: generating warm, context-aware human responses.

Result: cuisine routing accuracy went from ~60% to 100%.

**Decision 2: Multi-bubble conversation over single responses**

The first version sent one long agent message. It read like a system output. I redesigned it to send three short messages with natural pauses — a reaction, a result set, a personal closing thought. This matched how a human friend would respond to the same question.

**Decision 3: Separate user experience from monitoring layer**

The chat UI is deliberately free of evaluation language. Users see warm, natural conversation. The eval panel exists alongside it but is clearly labelled "PM monitoring layer — hidden from users." This reflects how I think real AI products should be built: clean UX on top, rigorous measurement underneath.

---

## Failure Analysis

Three things broke during the build and taught me more than the things that worked:

**Failure 1 — Vague queries scored 42% agent quality**
Queries like "I have no idea what I want" produced 0% constraint satisfaction because there were no constraints to satisfy. My original scoring formula penalised the agent for something the user didn't ask for. I fixed this: zero stated constraints = 100% constraint satisfaction. You can only fail to meet requirements that were stated.

**Failure 2 — Sushi Zen always won**
With no cuisine or constraint filter, all rankings defaulted to rating-based scoring. Sushi Zen (4.9/5) won every query regardless of what the user asked for. I added cuisine detection and a scaled price penalty to fix this. A restaurant 12 euros over budget should be penalised more than one that's 1 euro over.

**Failure 3 — Direct API calls from browser got blocked**
My first architecture called Anthropic directly from the browser. CORS blocked it immediately. I built a server-side proxy (`/api/claude`) to handle all API calls securely, keeping the API key in environment variables on Vercel. This is the correct production pattern.

---

## Metrics & Business Impact

**Agent Performance (live demo, tested across 50 queries):**

| Metric | Score |
|---|---|
| Intent Match Rate | 88–96% |
| Constraint Satisfaction | 91–100% on specific queries |
| Groundedness Score | 96% |
| Hallucination Rate | 4% |
| Overall Agent Score | 88–96% on specific queries |

**Business Impact Framing:**

Delivery Hero processes millions of orders daily across 65 countries. If conversational discovery handles 20% of sessions and improves order completion rate by 8% through better constraint matching:

```
Sessions using agent:         1,000,000 daily
Completion rate improvement:  +8%
Additional orders:            80,000 daily
Average order value:          ~€18
Additional daily GMV:         €1,440,000
Annual GMV impact:            ~€525M
```

This is a conservative estimate. The bigger opportunity is retention: users who feel understood by a platform come back. Users who abandon a search box and order mediocre food don't.

---

## What I'd Build Next

**1. Clarification questions for ambiguous queries**
When a user types "something light" — the agent should ask one clarifying question rather than guessing. "Light on calories or light on price?" Conversational recovery from ambiguity is the hardest part of agentic UX.

**2. Context persistence across sessions**
If I ordered spicy Indian food three times this month, the agent should weight my next recommendation accordingly. Today's prototype has no memory. A production system would.

**3. A/B testing against keyword search baseline**
The real proof of value is measuring Time-to-Order and abandonment rate against the existing search experience. I've designed the evaluation framework to support this — the agent score metric exists precisely to know when we're ready to run that experiment.

**4. Human review loop for low-confidence queries**
Any query scoring below 60% on the agent score should be flagged for human review to build the golden dataset. This is how you improve the system over time — not by guessing, but by learning from real failure cases.

---

## Key Learnings

**Evaluation is not a post-launch concern — it's a design input.**
I defined my success metrics before writing the first line of code. Intent Match Rate, Constraint Satisfaction, Groundedness — these shaped every architectural decision. Most AI products measure too late.

**The hardest part of agentic UX is not the AI — it's the conversation design.**
Getting Claude to extract constraints correctly took less time than getting the reaction message to feel genuinely warm. Language quality is a product problem, not just a technical one.

**Non-deterministic AI + deterministic business logic = the right hybrid.**
AI is excellent at understanding ambiguous human language. It's unreliable as a ranking engine for business-critical decisions. Knowing which problems belong to the AI layer and which belong to deterministic logic — that distinction drove every architectural decision I made here.

---

*Priyanshu Gupta — AI Product Manager*
*Portfolio: github.com/Priyanshug09/PM-Portfolio*
*Live Demo: https://agentic-discovery-for-food-delivery.vercel.app*
