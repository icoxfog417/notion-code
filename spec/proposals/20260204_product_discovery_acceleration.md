# Proposal: Product Discovery Acceleration — Mock Agent and Discovery Pipeline

**Date**: 2026-02-04
**Author**: Claude Agent
**Status**: Proposed

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

### Agent 2: User Interview Agent (Interview Preparation)

**Trigger**: PM writes user story + selects "Prepare Interview"
**Output**: Structured interview kit posted to a Notion sub-page

**What it generates**:
- Interview script with open-ended questions tailored to the user story
- Scenario walkthrough guide ("Ask the user to complete this task using the prototype")
- Observation checklist ("Watch for: confusion at step 3, time to complete, verbal reactions")
- Hypothesis statement ("We believe users will prefer approach X because Y. We'll know we're wrong if Z.")
- Feedback capture template (Notion database for structured notes per interview)

**Why this matters**: Most PMs don't have formal UX research training. They know they should interview users but aren't sure what to ask. The Interview Agent provides structure that turns a casual conversation into actionable research.

**PM value**: "I have a professional interview script 5 minutes after writing the story. I know exactly what to observe."

### Agent 3: Insight Agent (Feedback Synthesizer)

**Trigger**: PM has conducted 3-5 interviews and captured notes in the feedback database + selects "Synthesize Feedback"
**Output**: Insight summary posted to the user story's Notion page

**What it generates**:
- Pattern analysis across interviews ("4 of 5 users struggled with the navigation between steps")
- Verbatim quotes organized by theme
- Confidence assessment ("High confidence: users want X. Low confidence: unclear if Y matters")
- Suggested story refinements based on feedback patterns
- Decision recommendation ("Proceed to build" / "Pivot: users want Z instead" / "Need more interviews")

**Why this matters**: Synthesizing qualitative research is time-consuming and prone to confirmation bias. The PM tends to remember the most recent or most dramatic feedback, not the most representative. The Insight Agent provides systematic analysis.

**PM value**: "I have an objective synthesis of all user interviews in 5 minutes. I can confidently decide whether to proceed, pivot, or dig deeper."

### Agent 4: Variant Agent (Alternative Approach Generator)

**Trigger**: PM writes user story + selects "Generate Variants"
**Output**: 3 alternative approaches posted to Notion, each with a prototype URL

**What it generates**:
- 3 distinct approaches to solving the same user problem
- Each approach has a brief rationale ("This approach prioritizes simplicity" / "This approach prioritizes power users" / "This approach matches competitor X's pattern")
- Each approach has a deployable prototype
- Comparison matrix: trade-offs between approaches

**Why this matters**: PMs tend to converge on their first idea. The Variant Agent forces divergent thinking by generating alternatives the PM didn't consider. Combined with user interviews, this enables A/B prototype testing.

**PM value**: "I can test 3 different approaches with users in a single afternoon instead of committing to one approach and discovering it's wrong after development."

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

These 5 agents form a complete product discovery pipeline:

```
Phase 1: Diverge
    PM writes user story
        → Variant Agent generates 3 approaches (3 prototypes)
        → Landing Page Agent tests demand for each variant

Phase 2: Test
    PM picks top 2 variants based on landing page data
        → Interview Agent prepares interview scripts
        → PM conducts user interviews with prototypes

Phase 3: Converge
    PM captures interview notes in Notion
        → Insight Agent synthesizes feedback
        → PM decides: proceed / pivot / dig deeper

Phase 4: Refine
    PM updates user story based on validated insights
        → Mock Agent generates refined prototype
        → One more round of interviews to confirm

Phase 5: Build (existing US-001 + US-002)
    PM triggers Code Agent with validated, user-tested story
        → Production code generated with high confidence
        → Developer reviews code that solves a proven problem
```

**Total time from idea to validated concept: 3-5 days instead of 1-2 months.**

The developer enters the picture only at Phase 5, reviewing code for a feature that's already been validated with real users. This is the most efficient use of developer time possible — they never build something users don't want.

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

**US-010: Prepare User Interview from User Story**

**As a** Product Owner, **I want to** generate an interview script and observation checklist from a user story, **so that** I can conduct structured user interviews without UX research expertise.

Acceptance Criteria:
- [ ] A "Prepare Interview" action is available on user story pages in Notion
- [ ] The agent generates an interview script with open-ended questions tailored to the story
- [ ] A scenario walkthrough guide is included for use with the prototype
- [ ] An observation checklist is generated to capture structured notes
- [ ] A feedback capture database is created in Notion for interview notes
- [ ] All materials are posted to a sub-page of the user story

**US-011: Synthesize User Interview Feedback**

**As a** Product Owner, **I want to** synthesize feedback from multiple user interviews into patterns and recommendations, **so that** I can make evidence-based decisions about whether to proceed, pivot, or dig deeper.

Acceptance Criteria:
- [ ] A "Synthesize Feedback" action is available when 2+ interview notes exist in the feedback database
- [ ] The agent identifies patterns across interviews (common themes, recurring pain points)
- [ ] Key verbatim quotes are organized by theme
- [ ] A confidence assessment is provided for each finding
- [ ] A decision recommendation is provided (proceed / pivot / need more data)
- [ ] The synthesis is posted to the user story page

### Optional (Phase 2 Discovery Agents)

- **US-012: Generate Prototype Variants** — Multiple approaches to the same user problem
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

**Sprint 2: US-002 (Code feedback) + US-010 (Interview prep) + US-011 (Insight synthesis)**

All three stories are about feedback loops. US-002 closes the code iteration loop. US-010 + US-011 close the discovery iteration loop. Building them together creates the complete "Notion to Validated Product" workflow.

US-010 and US-011 are architecturally simpler than US-002 — their output is Notion pages (text), not code. The agents are text-generation agents, not coding agents. Lower implementation cost.

**Sprint 3 (was Sprint 2): US-004 + US-003**

Team governance moves one sprint later. Acceptable trade-off: governance is important for production adoption, but the Mock Agent demo is more important for initial stakeholder buy-in.

**Sprint 4 (was Sprint 3): US-006**

Workshop readiness. Now the workshop demonstrates both discovery and delivery flows — significantly more compelling.

### Implementation Cost of New Stories

| Story | Agent Type | Output Target | New Infrastructure | Cost |
|-------|-----------|--------------|-------------------|------|
| US-009 | Mock Agent (frontend-only code gen) | S3 static hosting + CloudFront URL | S3 bucket, CloudFront distribution, TTL-based cleanup | Medium |
| US-010 | Interview Agent (text generation) | Notion sub-page | None (uses existing Notion Writer) | Low |
| US-011 | Insight Agent (text analysis) | Notion page update | None (uses existing Notion Writer + reads feedback DB) | Low |

US-010 and US-011 are **low cost** because they are text-in, text-out agents that write back to Notion. No new infrastructure — they use the same pipeline as US-001/US-002, just with different agent prompts and Notion as the output target instead of GitHub.

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
