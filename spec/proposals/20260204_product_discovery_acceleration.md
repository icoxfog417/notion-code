# Proposal: Product Discovery Acceleration — Mock Agent and Discovery Pipeline

**Date**: 2026-02-04
**Author**: Claude Agent
**Status**: Proposed (Revised 2026-02-04 — Demo Deck Agent, A/B Test Agent, refined positioning)

## Background

### The Insight

Our competitive analysis (20260204_notion_mcp_competitive_analysis.md) established that our core differentiator is the **PO-owned iteration loop** — the PM triggers agents from Notion without developer involvement. The parallel invocation discussion revealed that when execution becomes cheap and parallel, the bottleneck shifts from "doing" to "judging."

The Mock Agent proposal pushes this further: **what if the PM doesn't need a developer at all?**

The current user stories (US-001 through US-008) focus on the **delivery** phase — generating production-quality code that developers review and ship. But the PM's most time-consuming bottleneck isn't delivery. It's **discovery**: the cycle between "I have an idea" and "I've validated it with real users."

### The Discovery Bottleneck Today

```
PM has idea
    → Writes user story                                    (30 min)
    → Asks developer to build prototype                    (wait 1-3 days)
    → Developer builds throwaway prototype                 (1-2 days)
    → PM schedules user interview                          (wait 2-5 days)
    → PM conducts interview with prototype                 (1 hour)
    → PM synthesizes feedback manually                     (2-4 hours)
    → PM refines story and repeats                         (back to step 1)

Total cycle time: 1-2 weeks per iteration
Typical iterations before confidence: 3-5
Time to validated concept: 1-2 months
```

The developer step (build throwaway prototype) is the critical-path bottleneck — not because it's hard, but because the developer is busy with production work. The PM is competing for developer attention to build something that will be thrown away.

### Why Mock Agent Is the Right Pivot

The Mock Agent eliminates the developer from the discovery loop entirely:

```
PM has idea
    → Writes user story in Notion                          (30 min)
    → Triggers Mock Agent                                  (2 clicks)
    → Gets clickable prototype                             (5-10 min)
    → Shows to users                                       (1 hour)
    → Refines and re-triggers                              (immediate)

Total cycle time: 2 hours per iteration
5 iterations in 2 days instead of 2 months
```

Key property of the Mock Agent: **the quality bar is "good enough to show users," not "production-ready."** This means:

- No developer review needed (throwaway by design)
- Simpler agent output (HTML/CSS prototype, not full-stack application)
- Faster execution (less code to generate)
- Lower cost per invocation (fewer tokens)
- Higher iteration speed (no review bottleneck)

## Proposal: Product Discovery Agent Pipeline

The Mock Agent alone is valuable, but the real killer is a **complete discovery pipeline** — a set of agents purpose-built for the discovery phase, each eliminating a different bottleneck in the PM's workflow.

### Agent 1: Mock Agent (Prototype Generator)

**Trigger**: PM writes user story + selects "Generate Prototype"
**Output**: Deployable static prototype (HTML/CSS/JS) hosted at a shareable URL

**What it generates**:
- Clickable UI that represents the user story's flow
- Realistic sample data (not "Lorem ipsum")
- Key interactions working (navigation, form submission, state changes)
- Mobile-responsive layout
- Hosted on a shareable URL (S3 static hosting or similar)

**What it does NOT generate**:
- Backend logic
- Database schema
- Authentication
- Production error handling
- Tests

**Why this is different from US-001 (Code Agent)**:

| Dimension | Code Agent (US-001) | Mock Agent |
|-----------|-------------------|------------|
| Purpose | Production code delivery | User interview prototype |
| Output | GitHub PR | Hosted URL |
| Quality bar | Production-ready | Visually convincing |
| Developer review | Required | Not needed |
| Lifecycle | Permanent (merged to main) | Disposable (auto-expire after 7 days) |
| Complexity | Full-stack | Frontend-only |
| Execution time | 5-15 min | 2-5 min |

**PM value**: "I can show users a working prototype 10 minutes after writing the story. No developer needed."

### Agent 2: Demo Deck Agent (Customer Demo & Feedback Collection)

**Trigger**: PM writes user story + selects "Generate Demo Deck"
**Output**: A complete demo experience in Notion — presentation, prototype embed, and feedback capture — all in one place

**What it generates**:
- **Explain**: Notion page sequence that walks the customer through the problem statement, proposed solution, and value proposition — structured as a presentation deck the PM can share-screen or send as a link
- **Show**: Embedded prototype link (from Mock Agent) with guided scenario instructions ("Try adding an item to your cart, then check out")
- **Collect**: Structured feedback form as a Notion database — pre-populated with key questions, rating scales, and open-ended fields the customer fills in directly

**The key insight**: The entire customer interview happens **in Notion**. The PM shares a Notion page link with the customer. The customer reads the explanation, clicks through the prototype, and submits structured feedback — all without a meeting. Synchronous interviews still work (PM shares screen and walks through the deck), but asynchronous validation is now possible too.

**Why this is different from a simple interview script**:

| Dimension | Interview Script (old) | Demo Deck (new) |
|-----------|----------------------|-----------------|
| Format | Text document PM reads from | Interactive Notion experience customer navigates |
| Customer interaction | Requires live meeting | Works async (customer self-serves) or sync (PM presents) |
| Feedback capture | PM takes notes manually | Customer fills structured Notion database directly |
| Prototype integration | Separate link | Embedded in the flow |
| Scalability | 1 interview at a time | Send to 10 customers simultaneously |

**PM value**: "I send a Notion link to 10 customers. They experience the demo and submit feedback at their own pace. I collect structured data without scheduling 10 meetings."

### Agent 3: Insight Agent (Feedback Synthesizer)

**Trigger**: PM selects "Synthesize Feedback" when feedback data exists (demo deck responses, customer tickets, or any Notion database)
**Output**: Insight summary posted to the user story's Notion page

**What it generates**:
- Pattern analysis across all data sources — demo deck feedback, customer tickets, support conversations, survey responses
- Cross-reference with existing Notion databases ("3 of 5 demo respondents raised the same issue that appears in 12 open support tickets")
- Verbatim quotes organized by theme
- Confidence assessment ("High confidence: users want X. Low confidence: unclear if Y matters")
- Suggested story refinements based on feedback patterns
- Decision recommendation ("Proceed to build" / "Pivot: users want Z instead" / "Need more data")

**Data sources** (the agent reads from any Notion database the PM designates):
- Demo Deck feedback database (structured responses from Agent 2)
- Customer support ticket databases
- Feature request databases
- NPS/survey result databases
- Sales call notes
- Any table-form data in Notion

**Why this matters**: PMs already have abundant customer data scattered across Notion databases — support tickets, feature requests, sales notes. The Insight Agent doesn't just analyze new interview feedback; it **connects new discovery data with existing customer signals**. This turns fragmented data into a coherent picture.

The agent leverages **Notion MCP** to extract content from any designated Notion database, giving it access to the full breadth of customer data without manual export or copy-paste.

**PM value**: "The agent connects my demo feedback with 6 months of customer tickets I never had time to cross-reference. I can see whether this idea solves a real problem at scale, not just for 5 interviewees."

### Agent 4: A/B Test Agent (Alternative Approach Generator)

**Trigger**: PM writes user story + selects "Generate A/B Test"
**Output**: 2-3 alternative approaches posted to Notion, each with its own Demo Deck and prototype URL

**What it generates**:
- 2-3 distinct approaches to solving the same user problem
- Each approach has a brief rationale ("This approach prioritizes simplicity" / "This approach prioritizes power users" / "This approach matches competitor X's pattern")
- Each approach has a deployable prototype (via Mock Agent)
- Each approach has a Demo Deck (via Demo Deck Agent) — ready to send to customers
- Comparison matrix: trade-offs between approaches
- Unified feedback database that tracks which variant each respondent experienced

**Why this matters**: PMs tend to converge on their first idea. The A/B Test Agent forces divergent thinking by generating alternatives the PM didn't consider, and makes it effortless to test them. Send variant A's Demo Deck to 5 customers and variant B's to 5 others. The Insight Agent then compares feedback across variants.

**PM value**: "I A/B test 3 different approaches with real customers in a single day, all from Notion. I pick the winner based on data, not opinion."

### Agent 5: Landing Page Agent (Demand Validation)

**Trigger**: PM writes feature description + selects "Generate Landing Page"
**Output**: Deployed landing page with analytics tracking

**What it generates**:
- Marketing-quality landing page describing the proposed feature
- Clear value proposition headline
- Feature benefit breakdown
- Call-to-action button ("Sign up for early access" / "Join the waitlist")
- Simple analytics: page views, CTA clicks, scroll depth
- Results posted back to Notion

**Why this matters**: This is the classic lean startup "fake door test." Before building anything, test whether users even want the feature. A 5% click-through rate means interest; 0.1% means the PM should reconsider.

**PM value**: "I can test market demand for a feature before anyone writes a single line of production code."

## The Complete Discovery Loop

These agents form a complete product discovery pipeline — all in Notion:

```
Phase 1: Diverge
    PM writes user story
        → A/B Test Agent generates 2-3 approaches
        → Each approach gets a Mock prototype + Demo Deck
        → (Optional) Landing Page Agent tests demand

Phase 2: Test
    PM sends Demo Deck links to customers
        → Customers experience the demo in Notion (async or sync)
        → Customers submit structured feedback directly in Notion
        → PM can send to 10+ customers in parallel

Phase 3: Converge
    PM triggers Insight Agent
        → Synthesizes demo feedback + existing customer tickets + support data
        → Cross-references new findings with existing Notion databases
        → Recommends: proceed / pivot / need more data

Phase 4: Refine
    PM updates user story based on validated insights
        → Mock Agent generates refined prototype
        → Demo Deck Agent generates updated demo
        → One more round with customers to confirm

Phase 5: Build (existing US-001 + US-002)
    PM triggers Code Agent with validated, user-tested story
        → Production code generated with high confidence
        → Developer reviews code that solves a proven problem
```

**Total time from idea to validated concept: 3-5 days instead of 1-2 months.**

The developer enters the picture only at Phase 5, reviewing code for a feature that's already been validated with real users. This is the most efficient use of developer time possible — they never build something users don't want.

### The Notion-Native Advantage

The entire discovery loop happens in Notion:
- PM **creates** the story in Notion
- Agents **generate** demos and decks in Notion
- Customers **experience** the demo in Notion
- Customers **submit feedback** into Notion databases
- Insight Agent **reads** feedback + existing data from Notion (via Notion MCP)
- Insight Agent **posts** synthesis back to Notion
- PM **decides** and triggers Code Agent — from Notion

No external survey tools. No separate interview recording platforms. No spreadsheets for data synthesis. Notion is the single environment for the entire discovery process.

## Why This Is Our Killer Differentiator

### vs. Notion MCP

Notion MCP gives developers access to Notion content. It does nothing for PMs in the discovery phase. A PM cannot use Cursor to generate prototypes, run interviews, or synthesize feedback. Our discovery pipeline serves a user (PM) that Notion MCP completely ignores.

### vs. Design Tools (Figma, etc.)

Figma prototypes require design skills. The Mock Agent generates from user stories written in natural language. The PM doesn't need to learn a design tool.

### vs. No-Code Tools (Bubble, Retool, etc.)

No-code tools still require the PM to learn the tool and build manually. The Mock Agent generates from text. Zero learning curve.

### vs. AI Coding Assistants (Cursor, Copilot, etc.)

AI coding assistants require an IDE. The PM stays in Notion. The prototype is deployed to a URL, not saved as local files.

### The Positioning Shift

```
Before:  "Notion to Code" (delivery acceleration)
After:   "Notion to Validated Product" (discovery + delivery acceleration)
```

This positions our solution not as a coding tool (crowded market, Notion MCP competes) but as a **product discovery platform** (empty market, no direct competitor).

### Fundamental Value and Implementation

**Fundamental value**: Accelerate product discovery to generate promising user stories, then pass validated stories to developers for implementation.

**Fundamental implementation**: "Invoke Agent from Notion" — a secure and scalable infrastructure on AWS (AgentCore + Lambda + SQS) that lets any Notion action trigger any agent type. The agents leverage Notion MCP to extract abundant content from the workspace.

This separation — value (discovery acceleration) and implementation (invoke from Notion) — means every new agent type we add multiplies value without changing the underlying infrastructure.

## Impact on Requirements

### New User Stories

**US-009: Generate Clickable Prototype from User Story**

**As a** Product Owner, **I want to** generate a clickable prototype from a user story and get a shareable URL, **so that** I can show it to users in interviews without asking a developer to build a throwaway mockup.

Acceptance Criteria:
- [ ] A "Generate Prototype" action is available on user story pages in Notion
- [ ] The agent generates a clickable HTML/CSS/JS prototype representing the story's user flow
- [ ] The prototype is deployed to a shareable URL
- [ ] The URL and screenshot preview are posted back to the Notion page
- [ ] The prototype auto-expires after a configurable period (default: 7 days)
- [ ] Prototype uses realistic sample data, not placeholder text

**US-010: Generate Demo Deck for Customer Validation**

**As a** Product Owner, **I want to** generate a complete demo experience (explanation, prototype, feedback form) as a Notion page I can share with customers, **so that** I can validate ideas with customers asynchronously without scheduling meetings or asking developers for help.

Acceptance Criteria:
- [ ] A "Generate Demo Deck" action is available on user story pages in Notion
- [ ] The agent generates a Notion page sequence: problem statement → solution explanation → embedded prototype link → guided scenario → feedback form
- [ ] A structured feedback database is created in Notion with pre-populated questions, rating scales, and open-ended fields
- [ ] The customer can navigate the entire experience (read, demo, feedback) by following a single Notion link
- [ ] The Demo Deck works both for async (customer self-serves) and sync (PM presents) use
- [ ] All materials are posted as sub-pages of the user story

**US-011: Synthesize Customer Feedback with Existing Data**

**As a** Product Owner, **I want to** synthesize feedback from demo deck responses and cross-reference it with existing customer data (support tickets, feature requests, sales notes) in Notion, **so that** I can make evidence-based decisions grounded in both new discovery data and historical customer signals.

Acceptance Criteria:
- [ ] A "Synthesize Feedback" action is available when feedback data exists in designated Notion databases
- [ ] The agent reads from multiple Notion databases: demo deck responses, customer tickets, feature requests, and any PM-designated data source
- [ ] The agent cross-references new feedback with existing data ("this issue matches 12 open support tickets")
- [ ] Pattern analysis identifies common themes across all data sources
- [ ] Key verbatim quotes are organized by theme
- [ ] A confidence assessment is provided for each finding
- [ ] A decision recommendation is provided (proceed / pivot / need more data)
- [ ] The synthesis is posted to the user story page

### Optional (Phase 2 Discovery Agents)

- **US-012: A/B Test with Multiple Approaches** — Generate 2-3 variants, each with its own Mock prototype and Demo Deck, with unified feedback tracking
- **US-013: Generate Landing Page for Demand Testing** — Lean startup fake door test

## Impact on Sprint Plan

### Proposed Revision

| Sprint | Stories | Focus |
|--------|---------|-------|
| 0 | — | Feasibility (current, unchanged) |
| **1** | **US-001 + US-009** | **Core trigger pipeline + Mock Agent** |
| **2** | **US-002 + US-010 + US-011** | **Feedback loops: code iteration + interview preparation + insight synthesis** |
| 3 | US-004 + US-003 | Team governance (monitoring + config) |
| 4 | US-006 | Workshop readiness |
| Backlog | US-007, US-008, US-005, US-012, US-013 | Knowledge Base, chaining, enriched PRs, variants, landing pages |

### Rationale for Sprint Restructuring

**Sprint 1: US-001 (Code trigger) + US-009 (Mock Agent)**

Both stories share the same pipeline infrastructure (webhook → SQS → AgentCore → delivery → Notion write-back). The only difference is:
- US-001 delivers to GitHub (PR) → Code Agent
- US-009 delivers to S3 (hosted URL) → Mock Agent

Building both in Sprint 1 means we build the pipeline once and demonstrate two agent types — one for delivery, one for discovery. This is a much more compelling demo than code generation alone.

**Sprint 2: US-002 (Code feedback) + US-010 (Demo Deck) + US-011 (Insight synthesis)**

All three stories are about feedback loops. US-002 closes the code iteration loop. US-010 + US-011 close the discovery iteration loop. Building them together creates the complete "Notion to Validated Product" workflow.

US-010 creates the Demo Deck experience in Notion — the agent generates Notion pages and databases, which is architecturally simpler than code generation. US-011 reads from Notion databases (via Notion MCP) and writes synthesis back to Notion. Both are text-generation agents with Notion as both input and output. Lower implementation cost than code agents.

**Sprint 3 (was Sprint 2): US-004 + US-003**

Team governance moves one sprint later. Acceptable trade-off: governance is important for production adoption, but the Mock Agent demo is more important for initial stakeholder buy-in.

**Sprint 4 (was Sprint 3): US-006**

Workshop readiness. Now the workshop demonstrates both discovery and delivery flows — significantly more compelling.

### Implementation Cost of New Stories

| Story | Agent Type | Output Target | New Infrastructure | Cost |
|-------|-----------|--------------|-------------------|------|
| US-009 | Mock Agent (frontend-only code gen) | S3 static hosting + CloudFront URL | S3 bucket, CloudFront distribution, TTL-based cleanup | Medium |
| US-010 | Demo Deck Agent (Notion page + DB generation) | Notion sub-pages + feedback DB | None (uses Notion Writer + Notion API for page/DB creation) | Low-Medium |
| US-011 | Insight Agent (multi-source analysis) | Notion page update | Notion MCP integration for reading multiple databases | Medium |

US-010 is **low-medium cost** because it generates Notion pages and databases (no external hosting), but the page structure is richer than simple text — it creates a multi-page experience with an embedded feedback database.

US-011 is **medium cost** (upgraded from low) because it now reads from multiple Notion databases via Notion MCP, requiring the agent to handle diverse data schemas and cross-reference across sources. The analysis is more sophisticated than single-source synthesis.

US-009 is **medium cost** because it requires S3 static hosting with auto-expiry, but the agent pipeline (webhook → SQS → orchestrator → AgentCore) is shared with US-001.

## Alternatives Considered

### Alternative A: Build Mock Agent as a separate product
- **Rejected.** The value comes from integration with the same Notion workflow. Separate products fragment the PM's workflow — the exact problem we're solving.

### Alternative B: Add Mock Agent but keep original sprint order
- **Rejected.** The Mock Agent's discovery value is strongest when demonstrated early. It's our best story for stakeholder buy-in: "PMs can validate ideas in hours, not months."

### Alternative C: Focus only on Mock Agent, drop Code Agent (US-001)
- **Rejected.** The complete story is discovery-to-delivery. Mock Agent validates the idea; Code Agent builds it. Both are needed. But Mock Agent should be prioritized because it serves a market with zero competition.

## Implementation Plan

1. Add US-009, US-010, US-011 to requirements.md
2. Update design.md with Mock Agent architecture (S3 static hosting, agent prompt design)
3. Update tasks.md with revised sprint assignments
4. Add Sprint 0 sandbox task: verify static site deployment to S3 + CloudFront with auto-expiry
5. Proceed with Sprint 1: shared pipeline + Code Agent + Mock Agent
