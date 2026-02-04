# Proposal: Design Specification Review — Alignment with Requirements v0.2.0

**Date**: 2026-02-04
**Author**: Claude Agent (Tech Lead Review)
**Status**: Proposed

## Background

The requirements specification was updated to v0.2.0 following two major proposals:

- **Notion MCP Competitive Analysis** (`20260204_notion_mcp_competitive_analysis.md`): Shifted the MVP context strategy from Knowledge Base to on-demand Notion API/MCP reads, deferring Knowledge Base to Sprint 4.
- **Product Discovery Acceleration** (`20260204_product_discovery_acceleration.md`): Added three new discovery agents (Mock Agent, Demo Deck Agent, Insight Agent) and reprioritized sprints to lead with discovery.

The design specification (`design.md`) remains at v0.1.0 and has not been updated to reflect these changes. This review identifies all gaps between the current design and the current requirements.

## Findings

### Critical — Design contradicts or omits active requirements

#### C1: Agent Definitions Missing Discovery Agents

**Requirements**: REQ-AO-002 defines 7 agent types. Sprint 1 requires Mock Agent; Sprint 2 requires Demo Deck Agent and Insight Agent.

**Design (Section 4.4)**: Only defines Spec Agent, Code Agent, and Review Agent — all three are in the backlog per tasks.md. The three Sprint 1-2 discovery agents are entirely absent.

**Impact**: No system prompt, tool configuration, or I/O specification exists for any of the agents that will actually be built first.

**Recommendation**: Rewrite Section 4.4 to define all active agents in sprint order:

| Agent | Sprint | Input | Tools | Output |
|-------|--------|-------|-------|--------|
| Code Agent | 1 | User story + context via Notion MCP + project config | Notion MCP, code sandbox | GitHub PR + PR description |
| Mock Agent | 1 | User story via Notion MCP | Notion MCP, code sandbox | HTML/CSS/JS files deployed to S3/CloudFront |
| Demo Deck Agent | 2 | User story + prototype URL via Notion MCP | Notion MCP, Notion API (write) | Notion page sequence + feedback database |
| Insight Agent | 2 | Feedback DB IDs + designated data source IDs via Notion MCP | Notion MCP (multi-DB read) | Notion page update (synthesis) |
| Spec Agent | Backlog | User story + context | TBD | Technical spec document |
| Review Agent | Backlog | Generated code + criteria | TBD | Review comments |

#### C2: `agent_type` Enum is Stale

**Requirements**: 7 agent types defined.

**Design (Section 2.1)**: `agent_type` enum is `spec, code, review`.

**Recommendation**: Update to `code, mock, demo_deck, insight, a_b_test, spec, review`.

#### C3: Notion MCP Not Referenced Anywhere in Design

**Requirements**: Section 1 states agents "leverage Notion MCP to extract abundant content." Dependencies list "Notion MCP: For agents to extract abundant content from Notion workspaces (on-demand reads for MVP)."

**Design**: Zero mentions of Notion MCP. Agents list only "Knowledge Base retrieval" as a context tool. The Orchestrator (4.3 step 3) assembles prompts with "Knowledge Base references."

**Impact**: The fundamental context mechanism for the MVP is undesigned. AgentCore agents need Notion MCP configured as a tool, not Knowledge Base retrieval.

**Recommendation**:
- Add Notion MCP to the technology stack table
- Add Notion MCP as a tool for all agents in Section 4.4
- Update Orchestrator (4.3) to configure Notion MCP for agent sessions instead of Knowledge Base
- Document Notion MCP authentication flow (how does the AgentCore agent session get Notion API credentials for MCP?)

#### C4: Delivery Handler Only Covers GitHub PRs

**Requirements**:
- REQ-OD-001: GitHub PR delivery (Code Agent) ✅ covered
- REQ-OD-004: Prototype deployment to shareable URLs with auto-expiry (Mock Agent) ❌ missing
- REQ-OD-005: Notion page/database generation as output (Demo Deck Agent, Insight Agent) ❌ missing

**Design (Section 4.5)**: Only describes GitHub branch → PR → Notion write-back flow.

**Recommendation**: Redesign Section 4.5 as a delivery dispatcher with three output handlers:

1. **GitHub Delivery Handler** (Code Agent): Existing design, minor updates
2. **Prototype Delivery Handler** (Mock Agent): Agent output → S3 upload → CloudFront URL → Notion write-back with URL + screenshot
3. **Notion Content Delivery Handler** (Demo Deck Agent, Insight Agent): Agent output → Notion API page/database creation → link posted to originating story

#### C5: S3 + CloudFront Missing from Architecture

**Requirements**: REQ-OD-004, US-009, Dependencies section all reference S3 + CloudFront for prototype hosting.

**Design**: Not present in the architecture diagram, technology stack, or component design.

**Recommendation**: Add to architecture diagram under a "Prototype Hosting" subgraph. Add to technology stack table. Design the deployment flow including:
- S3 bucket structure (per-prototype prefix with invocation_id)
- CloudFront distribution configuration
- S3 lifecycle rules for auto-expiry (default 7 days, configurable per REQ-OD-004)
- CORS configuration for prototype assets

#### C6: Webhook Trigger is Single-Agent

**Requirements**: Multiple trigger actions required per agent type:
- US-001: "Generate Code" action
- US-009: "Generate Prototype" action
- US-010: "Generate Demo Deck" action
- US-011: "Synthesize Feedback" action

**Design (Section 3.1)**: Webhook trigger condition is hardcoded to `Status` property changed to "Generate Code."

**Recommendation**: Design a trigger dispatch table:

| Trigger Value | Agent Type | Notes |
|--------------|------------|-------|
| Generate Code | code | Creates GitHub PR |
| Generate Prototype | mock | Deploys to S3/CloudFront |
| Generate Demo Deck | demo_deck | Requires prototype URL from prior Mock Agent run |
| Synthesize Feedback | insight | Requires feedback database to exist |

The webhook handler should read the trigger value and resolve it to an agent type before queuing to SQS. The SQS message schema should include the resolved `agent_type`.

---

### Major — Design is incomplete for requirements that are scheduled

#### M1: API Gateway Missing from Architecture Diagram

**Requirements**: tasks.md lists API Gateway as Sprint 1 AWS service.

**Design**: The architecture diagram shows `WH[Lambda: Webhook Handler]` receiving directly from Notion. In reality, API Gateway sits in front of Lambda to provide the HTTPS endpoint.

**Recommendation**: Add API Gateway to the architecture diagram between Notion and the Webhook Handler Lambda. Include in technology stack table.

#### M2: DynamoDB Missing from Architecture Diagram

**Requirements**: Invocation tracking and project configuration require persistent storage.

**Design**: Sections 2.1 and 2.2 define DynamoDB schemas, but the architecture diagram doesn't show DynamoDB. The Orchestrator component (4.3) references DynamoDB in prose but it's not in the visual.

**Recommendation**: Add DynamoDB to the architecture diagram, connected to the Orchestrator (read/write invocation records) and the Webhook Handler (read project config).

#### M3: No Feedback Database Schema

**Requirements**: US-010 specifies "structured feedback database with pre-populated questions, rating scales, and open-ended fields." US-011 reads from this database.

**Design**: No data model for feedback data. Section 2 only covers invocation records, project config, and knowledge base documents.

**Recommendation**: Add a Feedback Entry data model:

| Field | Type | Description |
|-------|------|-------------|
| feedback_id | string | Unique identifier |
| notion_page_id | string | Notion page where feedback is collected |
| user_story_id | string | Originating user story |
| demo_deck_id | string | Associated demo deck invocation |
| respondent | string | Customer/respondent identifier |
| ratings | object | `{question_key: numeric_rating}` |
| open_responses | object | `{question_key: text_response}` |
| submitted_at | datetime | When feedback was submitted |

Note: This data lives in Notion (as a Notion database), not DynamoDB. The schema should document the Notion database structure the Demo Deck Agent creates, so the Insight Agent knows what to read.

#### M4: Cost Enforcement Not Designed

**Requirements**: REQ-NF-052 requires configurable spending limits per project/workspace. REQ-AO-007 requires execution limits (timeout, token budget, cost cap) per invocation.

**Design**: Project config (2.2) has `cost_limit_per_invocation` and `cost_limit_per_month` fields, but no component checks them.

**Recommendation**: Add cost enforcement to the Orchestrator (4.3):
- **Pre-invocation check**: Before invoking AgentCore, query DynamoDB for month-to-date spend for the project. If at or over limit, reject with a clear error posted to Notion.
- **Per-invocation limit**: Pass timeout and token budget to AgentCore invocation parameters.
- **Post-invocation recording**: Record actual cost after completion.

#### M5: Knowledge Base Positioning Wrong for MVP

**Requirements**: Knowledge Base is deferred to Sprint 4. MVP uses Notion MCP for on-demand context.

**Design**: Content Sync (4.2) and Orchestrator (4.3) treat Knowledge Base as the primary context path. The entire "Content Ingestion" subgraph in the architecture diagram implies Knowledge Base is part of the core flow.

**Recommendation**: Restructure the architecture to show two context paths:
1. **MVP (Sprint 1-3)**: Agents use Notion MCP for on-demand reads. No Content Sync Lambda needed.
2. **Production Scale (Sprint 4+)**: Agents prefer Knowledge Base retrieval, fall back to Notion MCP. Content Sync Lambda ingests Notion content into Knowledge Base.

The architecture diagram should clearly label which components are MVP vs. Sprint 4+.

---

### Minor — Gaps that should be addressed but are lower priority

#### m1: Version Drift

Design is v0.1.0, requirements is v0.2.0. The design version should be updated when changes are applied (recommend v0.2.0 to match).

#### m2: Notification Design Missing

**Requirements**: REQ-FL-004 requires Notion notifications when agent workloads complete or require attention.

**Design**: No notification mechanism is designed.

**Recommendation**: Document that the Notion Writer Lambda (or Notion Content Delivery Handler) should use the Notion API to mention relevant users on the results page, which triggers Notion's built-in notification system. No separate notification infrastructure is needed.

#### m3: Incremental Sync Mechanism Unspecified

**Requirements**: REQ-NI-005 requires incremental content sync.

**Design (Section 4.2)**: Mentions "Handle incremental updates (only changed pages)" but doesn't specify the mechanism.

**Recommendation**: For Sprint 4 when Knowledge Base is implemented, specify: use Notion's `last_edited_time` property on pages, stored in the Knowledge Base metadata (`ingested_at` field in Section 2.3). On sync trigger, compare `last_edited_time > ingested_at` to identify changed pages. This avoids polling — sync is triggered by webhook or manual action.

#### m4: Prototype Auto-Expiry Not Designed

**Requirements**: US-009 requires auto-expiry with default 7 days.

**Design**: No mention of expiry mechanism.

**Recommendation**: Use S3 lifecycle rules on the prototype prefix. Each prototype is uploaded with a tag or prefix that includes the expiry date. An S3 lifecycle rule deletes objects older than the configured TTL. Add `prototype_expiry_days` to project config (default: 7).

## Impact

- **Requirements**: No changes needed — requirements are current and complete.
- **Design**: Major rewrite needed for Sections 2.1 (data model), 3.1 (webhook), 4.4 (agents), 4.5 (delivery). Architecture diagram needs significant updates. New sections needed for Notion MCP integration and prototype hosting.
- **Tasks**: No changes needed — tasks already reflect the correct architecture. The design needs to catch up.

## Recommended Update Order

1. Architecture diagram — add missing components, label MVP vs. Sprint 4+
2. Agent Definitions (4.4) — rewrite with discovery agents
3. Delivery Handler (4.5) — split into three output handlers
4. Webhook Handler (3.1, 4.1) — multi-agent trigger dispatch
5. Data Models (2) — fix enum, add feedback schema
6. Technology Stack (1.2) — add Notion MCP, API Gateway, S3/CloudFront
7. Orchestrator (4.3) — Notion MCP instead of KB, add cost enforcement
8. Minor items (notifications, auto-expiry, incremental sync)

## Alternatives Considered

**Option A: Patch the existing design incrementally** — Add discovery agents as addenda without restructuring. Rejected because the Knowledge Base-first architecture assumption is wrong for MVP; incremental patches would create internal contradictions.

**Option B: Full rewrite of design.md** — Recommended. The structural assumptions have changed enough (Notion MCP over Knowledge Base, discovery-first agent priority, multi-output delivery) that a coherent rewrite is cleaner than patching.

## Implementation Plan

1. Create this review proposal (this document)
2. Upon approval, rewrite `design.md` addressing all Critical and Major findings
3. Update version to v0.2.0
4. Commit with reference to this proposal
