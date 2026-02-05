# Proposal: Business Strategy, Revenue Model, and Competitive Moat

**Date**: 2026-02-05
**Author**: Co-Founder Discussion (Claude Agent)
**Status**: Proposed

## Background

As cofounders, we need to resolve three strategic questions before Sprint 1:

1. **Platform risk**: If Notion or Anthropic ships native "trigger agents from Notion" functionality, does our product survive?
2. **Revenue model**: Open source today, but what generates revenue? Subscription, invocation-based, or hybrid?
3. **Competitive moat**: What makes us defensible — not just differentiated today, but hard to replicate over 18-24 months?

This proposal provides an honest assessment, including areas where our current positioning is weaker than we might assume.

## 1. Platform Risk Assessment

### 1.1 Threat Matrix

| Threat Source | What They Could Build | Likelihood (18mo) | Impact | Our Response |
|--------------|----------------------|-------------------|--------|-------------|
| **Notion AI** | Native "Generate Prototype" button, built-in feedback synthesis | **High** | **Critical** | Notion already has Notion AI and is investing heavily. They could add agent triggers natively. |
| **Notion MCP expansion** | Event-driven triggers (webhooks → MCP), PM-facing MCP client embedded in Notion | **Medium** | **High** | Currently pull-based only, but Notion could add push. |
| **Anthropic (Claude)** | Claude-native Notion integration, direct "Notion to Prototype" workflow | **Low-Medium** | **Medium** | Anthropic builds horizontal platforms, not vertical products. But they could partner with Notion. |
| **AWS (Bedrock)** | Bedrock AgentCore templates for "Notion-to-X" workflows | **Low** | **Low** | AWS moves slowly on vertical solutions. More likely to enable us than compete with us. |
| **Cursor / Windsurf** | Expand from developer IDE into PM-facing tools | **Low** | **Low** | IDE-centric companies have no PM workflow expertise. |
| **Zapier / Make** | "Notion trigger → AI agent → output" as a Zap template | **Medium-High** | **High** | Zapier already has Notion triggers + AI actions. Shallow but fast. |

### 1.2 Honest Assessment

**The hardest threat is Notion itself.** Notion has:
- The user relationship with PMs (our target persona)
- The data (all the Notion content our agents need)
- The distribution (every PM already uses Notion)
- The AI team (Notion AI is already a product)

If Notion adds a "Generate Prototype" button that calls their own AI, our value proposition weakens significantly. This is not a distant risk — Notion AI already generates text, tables, and pages. Extending to prototype generation is architecturally straightforward for them.

**The second hardest threat is Zapier.** Zapier already supports "Notion database trigger → webhook → AI action." A Zapier user could wire Notion → Claude API → GitHub/S3 today. The experience would be worse than ours, but it would be available immediately with no new product needed.

### 1.3 What They Cannot Easily Replicate

| Our Capability | Why It's Hard to Copy |
|---------------|----------------------|
| **Skill ecosystem with community content** | Requires building an open standard, community contributions, and curation. Network effects take time. |
| **Deep discovery loop (prototype → demo → feedback → synthesis)** | Not a single feature but a 4-step pipeline. Notion would need to build the full loop, not just one agent type. |
| **Cross-platform agent orchestration** | We're not Notion-only. We can add Linear, Jira, Asana triggers. Notion only serves Notion users. |
| **Enterprise governance (cost controls, audit, team config)** | Enterprise features are unglamorous but create switching costs. Notion AI has none of this today. |
| **Model flexibility** | Notion AI is locked to whatever model Notion chooses. Enterprises want choice. |

## 2. Business Drivers

### 2.1 Primary Driver: The Discovery Loop

The core business driver is **time compression in product discovery**. The existing proposals correctly identify this:

- Current discovery cycle: 1-2 months per concept
- With our platform: 3-5 days per concept
- This is a **10-20x improvement** in the PM's most valuable activity

**Revenue implication**: If a PM team runs 10 discovery cycles per quarter, and each cycle previously consumed ~$15K in developer time (2-3 days of a developer building throwaway prototypes), the platform saves $120K-150K per quarter per team. This justifies significant pricing.

### 2.2 Secondary Driver: The Skill Marketplace

The skill-based architecture (SKILL.md) creates a marketplace opportunity:

| Marketplace Layer | Examples | Revenue Mechanism |
|------------------|---------|-------------------|
| **Platform skills** (we build) | Code Generation, Prototype, Demo Deck, Insight Synthesis | Included in platform |
| **Industry skill packs** (we or partners build) | FinTech compliance review, HealthTech HIPAA prototype, SaaS onboarding flow generator | Premium add-ons |
| **Community skills** (developers build) | React dashboard prototype skill, Django REST API skill, Terraform infra skill | Revenue share (70/30) |
| **Enterprise custom skills** (customer's developers build) | Internal coding standards, proprietary patterns | Free (increases lock-in) |

**This is the Cursor analogy done right.** Cursor's moat isn't the model — it's the integration depth and community. Our moat isn't the AI — it's the skill ecosystem. Skills encode organizational knowledge (coding standards, architecture patterns, prototype styles) that is expensive to recreate on another platform.

### 2.3 Tertiary Driver: Cross-Platform Expansion

Notion is our beachhead, not our boundary. The "invoke agent from workspace tool" pattern applies to:

| Platform | Trigger | Target Persona |
|----------|---------|---------------|
| **Notion** (Sprint 1) | User story status change | PM |
| **Linear** (future) | Issue status change | Engineering Manager |
| **Jira** (future) | Ticket transition | PM / Engineering Manager |
| **Figma** (future) | Design review comment | Designer |
| **Slack** (future) | Slash command / message reaction | Anyone |

Each platform integration multiplies the addressable market without changing the core agent infrastructure. The same skills, same AgentCore runtime, same cost controls — different triggers.

## 3. Revenue Model Analysis

### 3.1 Option A: Pure Subscription (SaaS)

| Tier | Price | Includes |
|------|-------|----------|
| Starter | $49/mo | 50 agent invocations, 2 projects, basic skills |
| Team | $199/mo | 300 agent invocations, 10 projects, all platform skills |
| Enterprise | Custom | Unlimited invocations, custom skills, SSO, audit logs |

**Pros**: Predictable recurring revenue, higher LTV, standard SaaS metrics
**Cons**: High commitment for first-time users, doesn't scale with value delivered, overpays for low-usage months

### 3.2 Option B: Pure Invocation (Zapier Model)

| Component | Price |
|-----------|-------|
| Prototype generation | $2-5 per invocation |
| Demo deck generation | $1-3 per invocation |
| Insight synthesis | $3-8 per invocation |
| Code generation | $5-15 per invocation |

**Pros**: Zero commitment, aligns cost with value, easy to start, transparent
**Cons**: Unpredictable revenue, no recurring baseline, customers optimize aggressively to reduce invocations, race-to-bottom pricing pressure

### 3.3 Option C: Hybrid (Recommended)

| Component | Mechanism |
|-----------|-----------|
| **Platform fee** | $0 (open core — self-host for free) |
| **Managed service** | $99-299/mo base (includes infra, monitoring, support, N invocations/mo) |
| **Invocation credits** | Included credits per tier, overage at per-invocation rates |
| **Skill marketplace** | 30% take rate on community/partner skills |
| **Enterprise add-ons** | SSO, audit, custom SLA, dedicated support |

**Why hybrid**:
- The base subscription creates recurring revenue and switching costs
- Invocation credits align cost with usage (Zapier insight)
- The open-core model drives adoption (developers self-host, then bring to teams)
- The marketplace creates ecosystem revenue that scales with community size
- Enterprise add-ons capture willingness-to-pay from large organizations

### 3.4 The Zapier Insight — Applied Correctly

The user correctly identifies that Zapier's invocation model is differentiated in a market dominated by subscriptions. But the lesson from Zapier is more nuanced:

**What Zapier actually does**: Free tier (100 tasks/mo) → paid tiers with task limits ($19.99/mo for 750 tasks) → enterprise with unlimited tasks. This is a **hybrid model** — subscription base + usage gating. It is NOT pure pay-per-invocation.

**What we should take from Zapier**:
1. **Free tier drives adoption**: Let PMs try 5-10 agent invocations for free. They'll evangelize internally.
2. **Usage-based gating creates natural upsell**: Team hits the limit → upgrades.
3. **Enterprise wants predictable costs**: Unlimited tier with committed spend.
4. **Per-invocation transparency builds trust**: Show cost-per-run in the Notion dashboard. PMs know what they're spending.

## 4. Multi-Model Strategy

### 4.1 Current State

The architecture is deeply coupled to Claude via the Claude Agent SDK:
- `claude-agent-sdk` as the runtime engine
- SKILL.md files designed for Claude's instruction-following
- MCP integration through Claude's tooling
- Bedrock as the model provider

### 4.2 Why Multi-Model Matters (But Not Yet)

**The user is right that multi-model support is strategically important.** Here's why:

| Reason | Impact |
|--------|--------|
| **Enterprise compliance** | Some enterprises mandate specific model providers (Azure OpenAI for Microsoft shops, Bedrock for AWS shops). Single-model limits TAM. |
| **Cost optimization** | Different tasks have different cost profiles. Insight synthesis may work fine with a cheaper model. Prototype generation needs the best model. |
| **Vendor independence** | If Anthropic raises prices 3x, we need an exit path. |
| **Competitive positioning** | "Works with any model" is a strong message against Notion AI (locked to their model) and Cursor (primarily Claude/OpenAI). |

**But premature abstraction kills startups.** Building a model-agnostic skill engine before we have paying customers is over-engineering. The sequence should be:

1. **Sprint 1-4**: Ship with Claude only. Validate the product works.
2. **Post-launch**: Add model selection per skill. "This skill works best with Claude Opus, but you can use GPT-4o at lower cost."
3. **V2**: Abstract the skill engine to be fully model-agnostic. Skills define behavior; the platform selects the optimal model.

### 4.3 AgentCore as the Migration Path

The user correctly identifies that **AgentCore's model-agnostic framework** makes this feasible:
- AgentCore Runtime supports any container — not limited to Claude Agent SDK
- Bedrock provides access to Claude, Llama, Mistral, Cohere, and others
- We could run a Strands Agent (model-agnostic) alongside Claude Agent SDK skills
- The Gateway + Memory + Identity layers are model-independent

**Architecture evolution**:
```
Phase 1 (Now):     Claude Agent SDK → Bedrock Claude
Phase 2 (Post-MVP): Claude Agent SDK → Bedrock Claude (primary)
                     + Strands Agent  → Bedrock [any model] (cost-optimized skills)
Phase 3 (V2):       Unified SDK      → Bedrock [any model] (full flexibility)
```

## 5. Key Distinguishable Functions

Ranked by defensibility (how hard to replicate), not by user value:

### 5.1 Tier 1: Hard to Replicate (12+ months)

**Skill Ecosystem**
- Open standard (SKILL.md) → community contributions → network effects
- Enterprise custom skills encode organizational knowledge → high switching costs
- Marketplace creates revenue and content flywheel
- **Why defensible**: Like app stores, the first ecosystem with critical mass wins. Latecomers struggle to attract developers when a platform already has thousands of skills.

**Cross-Platform Agent Orchestration**
- Same skills work from Notion, Linear, Jira, Slack
- Skills are platform-agnostic; triggers are platform-specific
- **Why defensible**: Each platform integration is engineering work that compounds. A competitor entering today needs to replicate all integrations.

### 5.2 Tier 2: Medium to Replicate (6-12 months)

**Discovery Loop Pipeline**
- 4-step pipeline (prototype → demo → feedback → synthesis) not just a single feature
- Each step generates data that feeds the next
- **Why defensible**: Building one agent is easy. Building a pipeline with data flow between agents, feedback databases, and cross-referencing is hard.

**Enterprise Governance**
- Per-invocation cost tracking, monthly budgets, team-wide config
- Audit trails, execution dashboards, cost allocation by project
- **Why defensible**: Enterprise features require understanding enterprise buying processes, compliance needs, and IT admin workflows. Notion AI has none of this.

### 5.3 Tier 3: Easy to Replicate (3-6 months)

**Notion Trigger → Agent → Notion Results**
- The core "trigger from Notion" pattern is conceptually simple
- **Why NOT defensible on its own**: Notion could build this natively. Zapier can wire it today. The trigger mechanism is commodity; the value is in what happens after the trigger.

**Prototype Generation**
- HTML/CSS/JS generation from text descriptions
- **Why NOT defensible on its own**: Any LLM can generate prototypes. Claude, GPT-4o, Gemini all do this well. The agent is a thin wrapper.

### 5.4 Strategic Implication

**Our defensibility does NOT come from the trigger mechanism or the AI model.** It comes from:
1. The **skill ecosystem** (content flywheel)
2. The **discovery pipeline** (multi-step orchestration)
3. The **cross-platform reach** (compounding integrations)
4. The **enterprise governance** (switching costs)

This means our roadmap priority should invest heavily in the skill ecosystem (marketplace infrastructure, community programs, partner integrations) and cross-platform expansion — not in making the trigger mechanism fancier or the AI model smarter.

## 6. Recommended Strategy

### 6.1 Short-Term (Sprints 1-4): Validate and Ship

- Ship the discovery loop on Notion (current plan)
- Claude-only, Notion-only
- Open source the core, charge nothing
- Goal: **get 10 PM teams using it weekly**
- Metric: invocations per week, discovery cycles completed

### 6.2 Medium-Term (Months 4-8): Monetize and Expand

| Action | Purpose |
|--------|---------|
| Launch managed service (hybrid pricing) | Revenue from teams that don't want to self-host |
| Launch skill marketplace (beta) | Ecosystem flywheel begins |
| Add Linear integration | Second platform proves cross-platform thesis |
| Add model selection per skill | Enterprise compliance requirement |
| Add SSO + audit | Enterprise sales enablement |

### 6.3 Long-Term (Months 9-18): Ecosystem and Platform

| Action | Purpose |
|--------|---------|
| Skill marketplace GA with revenue share | Self-sustaining ecosystem |
| Jira, Figma, Slack integrations | Maximize addressable market |
| Full multi-model support | Model-agnostic positioning |
| "Discovery Analytics" dashboard | Data flywheel: show teams what discovery patterns lead to successful products |
| Partner program (consulting firms, agencies) | Channel revenue |

### 6.4 Competitive Positioning Statement

**"The product discovery platform that PMs trigger from the tools they already use. Prototype, validate, and build — without switching context or waiting for developers."**

Not: "Notion AI agent" (too narrow, too dependent on Notion)
Not: "AI code generator" (crowded, commoditizing)
Not: "Claude integration" (too dependent on one model)

## 7. Risks and Mitigations

| Risk | Probability | Mitigation |
|------|------------|------------|
| Notion builds native discovery agents | Medium-High | Move fast on skill ecosystem; cross-platform expansion makes us Notion-independent |
| Claude Agent SDK has breaking changes | Medium | Maintain abstraction layer; contribute to the SDK to influence direction |
| PMs don't adopt (prefer manual discovery) | Medium | Free tier with zero setup; workshop program for education |
| Skill marketplace doesn't attract developers | Medium | Seed with high-quality first-party skills; developer advocacy program |
| Enterprise sales cycle too long | High | PLG motion first (individual PM → team adoption → enterprise deal) |
| Zapier builds a good-enough version | Medium | Our depth (4-step pipeline, skill ecosystem) vs. their breadth (shallow integration) |

## 8. Summary: Answers to Co-Founder Questions

### What's our business driver?

**Time compression in product discovery** — turning 1-2 month validation cycles into 3-5 days. The revenue comes from a managed service with hybrid pricing (subscription base + invocation credits) and a skill marketplace with revenue share.

### What are the key distinguishable functions?

In order of defensibility:
1. **Skill ecosystem** — network effects from community content create compounding moat
2. **Cross-platform orchestration** — Notion today, Linear/Jira/Figma/Slack tomorrow
3. **Discovery pipeline depth** — 4-step loop is harder to replicate than any single agent
4. **Enterprise governance** — cost controls, audit, team config create switching costs

The trigger mechanism and AI model are NOT differentiators — they are commodity. Invest in ecosystem and pipeline, not in trigger UX.

### How do we survive Notion building this natively?

By being **platform-agnostic** (not just a Notion tool) and **ecosystem-driven** (skills marketplace that Notion has no incentive to build). Notion will optimize for their platform; we optimize for the PM workflow across all platforms.

### Is multi-model support fundamental?

**Strategically yes, tactically not yet.** Ship with Claude. Plan for multi-model at the architecture level. Implement it when an enterprise customer requires it or when pricing pressure makes it necessary (estimated: month 6-8).

### Should we follow the Zapier model?

**Take the insight, not the exact model.** Usage-based transparency (show cost per invocation) and low-commitment entry (free tier) are right. But add a subscription base for predictable revenue and enterprise needs. The hybrid model captures the best of both.

## Impact

- Requirements: No changes (this is strategy, not features)
- Design: No immediate changes. Future proposals will address skill marketplace infrastructure and cross-platform trigger abstraction.
- Tasks: Add "business strategy validation" tasks to Sprint 0 — customer interviews with PMs to validate discovery loop value proposition.

## Alternatives Considered

### Alternative A: Stay Open Source Only, Monetize Through Consulting
- **Rejected.** Consulting doesn't scale. Revenue is linearly proportional to headcount. The skill marketplace creates exponential revenue potential.

### Alternative B: Go Enterprise-First with Custom Pricing
- **Rejected for now.** Enterprise sales cycles are 6-12 months. We need PLG adoption first to prove demand and build case studies.

### Alternative C: Build Model-Agnostic from Day One
- **Rejected.** Premature abstraction delays shipping. Claude Agent SDK gives us the fastest path to a working product. Abstract later when the product-market fit is proven.

### Alternative D: Focus on Notion Only, Don't Plan for Cross-Platform
- **Rejected.** Single-platform dependency is the biggest strategic risk. The architecture should assume cross-platform from the start, even if we only implement Notion first.

## Implementation Plan

1. Validate discovery loop value with 5 PM interviews (Sprint 0)
2. Ship discovery agents on Notion (Sprints 1-2)
3. Validate willingness-to-pay with early adopters (Sprint 3)
4. Design skill marketplace infrastructure (Sprint 3 proposal)
5. Design cross-platform trigger abstraction (Sprint 4 proposal)
6. Launch managed service beta (post-Sprint 4)
