# Implementation Tasks

**Project**: Notion-AWS Integration for AI-Driven Development Lifecycle (AI DLC)
**Last Updated**: 2026-02-05
**Status**: Sprint 0 - Feasibility & Foundation

## Task Status Legend

| Icon | Status | Meaning |
|------|--------|---------|
| ⬜ | TODO | Not started, available for work |
| 🔄 | IN PROGRESS | Currently being worked on |
| ✅ | DONE | Completed |
| 🚫 | BLOCKED | Waiting on dependency |
| ⏸️ | ON HOLD | Paused |

## Key Principles

- Work in vertical slices (end-to-end features)
- Verify unknowns in `.sandbox/` before production implementation
- Create proposals before changing spec files
- Keep tasks small and independently testable
- Prioritize what Notion MCP cannot provide (see spec/proposals/20260204_notion_mcp_competitive_analysis.md)
- Prioritize product discovery acceleration (see spec/proposals/20260204_product_discovery_acceleration.md)
- Build on the Claude Code ecosystem — Claude Agent SDK + Skills + MCP (see spec/proposals/20260204_claude_code_ecosystem_design.md)
- Use event source adapter pattern: platform-specific adapters produce canonical invocation format (see spec/proposals/20260205_event_source_abstraction.md)

## Sprint 0: Feasibility & Specification

**Goal**: Validate technical feasibility of core integrations, finalize specifications, and prepare for stakeholder review
**Deliverable**: Verified integration patterns, complete spec documents, stakeholder presentation materials

### Tasks

- ✅ Define requirements specification (requirements.md)
- ✅ Define high-level architecture (design.md)
- ✅ Create proposal document (spec/proposals/20260203_notion_aws_integration.md)
- ✅ Competitive analysis against Notion MCP (spec/proposals/20260204_notion_mcp_competitive_analysis.md)
- ✅ Product discovery acceleration proposal (spec/proposals/20260204_product_discovery_acceleration.md)
- ✅ Claude Code ecosystem design proposal (spec/proposals/20260204_claude_code_ecosystem_design.md)
- ⬜ Verify Claude Agent SDK on AgentCore: deploy base container with `claude-agent-sdk` + `bedrock-agentcore`, invoke via `query()`, confirm Bedrock model access
- ⬜ Verify Notion MCP via `.mcp.json`: configure `@notionhq/notion-mcp-server` in `.mcp.json`, confirm agent can read/write Notion pages
- ⬜ Verify GitHub MCP via `.mcp.json`: configure `@modelcontextprotocol/server-github`, confirm agent can create branches and PRs
- ⬜ Verify Agent Skills loading: create a test SKILL.md in `.claude/skills/`, confirm agent auto-discovers and invokes it
- ⬜ Verify Notion API webhook capabilities in sandbox
- ⬜ Verify S3 static site deployment with CloudFront and auto-expiry (for prototype hosting)
- ⬜ Measure latency and token cost: Claude Agent SDK on AgentCore end-to-end invocation
- ⬜ Document sandbox findings in implementation_qa.md
- ⬜ Prepare stakeholder review materials (customer journey diagram, cost estimates, Notion MCP differentiation, discovery pipeline vision, Claude Code ecosystem advantages)
- ⬜ Review and refine requirements with product team feedback

### Deferred from Sprint 0 (reason: Knowledge Base moved to Sprint 4+)

- ⏸️ Verify Bedrock Knowledge Base ingestion from Notion content format in sandbox

## Sprint 1: Dual Agent MVP — Code + Prototype (US-001 + US-009)

**Goal**: Build the shared agent pipeline and demonstrate two agent types: Code Agent (delivery) and Mock Agent (discovery)
**Deliverable**: PM writes a user story in Notion → triggers Code Agent → gets GitHub PR, OR triggers Mock Agent → gets clickable prototype at a shareable URL
**User Stories**: US-001 (Trigger Code Generation), US-009 (Generate Clickable Prototype)

### Shared Pipeline Tasks

- ⬜ Set up AWS CDK project structure with dev environment
- ⬜ Implement Trigger Handler Lambda with adapter pattern (route to platform-specific adapter, produce canonical SQS message)
- ⬜ Implement Notion adapter (validate Notion webhook signature, parse event, resolve agent type, snapshot page content)
- ⬜ Set up SQS queue for canonical invocation requests
- ⬜ Implement Orchestrator Lambda (dequeue → assemble prompt with trigger context → invoke AgentCore Runtime)
- ⬜ Set up DynamoDB table for invocation tracking (include `source_type` and `source_page_id` fields)
- ⬜ Implement Completion Handler Lambda with result writer pattern (parse agent output → update DynamoDB → delegate to platform-specific result writer)
- ⬜ Implement Notion result writer (update Notion dashboard, update originating page, notify users)
- ⬜ Set up CloudWatch logging and basic alarms

### Agent Runtime Tasks (Claude Agent SDK)

- ⬜ Build agent container: Dockerfile with Node.js + claude-code + Python + claude-agent-sdk
- ⬜ Implement `main.py` entrypoint with `BedrockAgentCoreApp` + `query()` pattern
- ⬜ Configure `.mcp.json` with Notion MCP and GitHub MCP servers
- ⬜ Write `CLAUDE.md` base system prompt with project context and security restrictions
- ⬜ Deploy AgentCore Runtime via CDK (`agentcore.Runtime` with ECR image)
- ⬜ Configure environment variables (NOTION_TOKEN, GITHUB_TOKEN from Secrets Manager)

### Code Generation Skill Tasks (US-001)

- ⬜ Write `code-generation` SKILL.md (coding conventions, PR format, test generation)
- ⬜ End-to-end test: agent invoked with user story → skill generates code → creates GitHub PR via MCP

### Prototype Generation Skill Tasks (US-009)

- ⬜ Write `prototype-generation` SKILL.md (HTML/CSS/JS prototype, realistic sample data)
- ⬜ Set up S3 bucket + CloudFront distribution for prototype hosting
- ⬜ Configure S3 upload via Bash tool (AWS CLI) in agent container
- ⬜ Implement auto-expiry (S3 lifecycle rules, default 7 days)

### Integration Tests

- ⬜ End-to-end test: Notion story → Code Agent → GitHub PR → Notion summary
- ⬜ End-to-end test: Notion story → Mock Agent → deployed prototype URL → Notion link

## Sprint 2: Feedback Loops — Code Iteration + Discovery (US-002 + US-010 + US-011)

**Goal**: Close both feedback loops — code iteration for delivery, and Demo Deck + Insight synthesis for discovery
**Deliverable**: PM can iterate on code output via Notion feedback. PM can generate Demo Decks for customer validation and synthesize feedback with existing customer data — all in Notion.
**User Stories**: US-002 (Review & Feedback), US-010 (Demo Deck), US-011 (Insight Synthesis)

### Code Feedback Loop Tasks (US-002)

- ⬜ Extend webhook handler to detect "Needs Changes" property + feedback comments
- ⬜ Implement feedback re-trigger (re-invoke Code Agent with original context + feedback)
- ⬜ Add parent_invocation_id to DynamoDB for iteration history tracking
- ⬜ Display iteration history on Notion page
- ⬜ End-to-end test: story → PR → feedback in Notion → re-generation → updated PR

### Demo Deck Skill Tasks (US-010)

- ⬜ Write `demo-deck` SKILL.md (generate explanation pages, embed prototype link, create feedback form)
- ⬜ Test Notion multi-page creation via `mcp__notion__*` tools (problem statement → solution → demo scenario → feedback form)
- ⬜ Test structured feedback database creation with pre-populated questions, rating scales, and open-ended fields
- ⬜ Integrate with Prototype Generation skill output (embed prototype URL in guided scenario section)
- ⬜ End-to-end test: story → "Generate Demo Deck" → Notion page sequence + feedback DB → customer submits feedback via Notion link

### Insight Synthesis Skill Tasks (US-011)

- ⬜ Write `insight-synthesis` SKILL.md (multi-source analysis, cross-referencing, confidence scoring, recommendations)
- ⬜ Test Notion MCP multi-database reads (demo feedback, customer tickets, feature requests, sales notes)
- ⬜ Test cross-reference logic via skill instructions (connect new demo feedback with existing customer signals)
- ⬜ Post synthesis results to user story page (patterns, quotes, confidence, proceed/pivot recommendation)
- ⬜ End-to-end test: demo feedback DB + customer ticket DB → "Synthesize" → cross-referenced insight summary

## Sprint 3: Team Governance + Gateway Migration (US-004 + US-003 + US-014)

**Goal**: Add team-wide visibility, project configuration, skill management, and migrate MCP to Gateway
**Deliverable**: Team leads can configure projects, manage skills, and monitor all agent executions from Notion
**User Stories**: US-004 (Monitor Execution), US-003 (Project Configuration), US-014 (Skill Management)

### Governance Tasks

- ⬜ Build Notion execution dashboard database template
- ⬜ Implement real-time status updates from agents to Notion dashboard
- ⬜ Add cost tracking per invocation (token usage → estimated USD)
- ⬜ Implement cost aggregation per project and per time period
- ⬜ Add error handling with user-friendly messages posted to Notion
- ⬜ Build Notion project configuration page template
- ⬜ Implement configuration reader (Notion config page → DynamoDB)
- ⬜ Wire project config into Orchestrator (coding standards, repo, branch strategy)
- ⬜ Configuration validation with clear error messages

### Gateway Migration Tasks

- ⬜ Deploy AgentCore Gateway via CDK with MCP protocol configuration
- ⬜ Add Notion MCP as Gateway target with OAuth credential provider
- ⬜ Add GitHub MCP as Gateway target with OAuth credential provider
- ⬜ Migrate agent `.mcp.json` from direct connections to Gateway endpoint
- ⬜ Configure AgentCore Identity for credential rotation
- ⬜ Verify all existing skills work through Gateway (no behavior change)

### Skill Management Tasks (US-014)

- ⬜ Document skill creation guide for developers (SKILL.md format, testing, deployment)
- ⬜ Test installing community skills from the Claude Code ecosystem
- ⬜ Establish skill review process (PR-based, team lead approval)

## Sprint 4: Workshop Readiness & Production Scale (US-006 + US-007)

**Goal**: Prepare workshop environment and build Knowledge Base for production-scale context
**Deliverable**: Workshop kit for facilitators + Knowledge Base for large workspaces
**User Stories**: US-006 (Workshop Demo), US-007 (Knowledge Base Sync)

### Workshop Tasks (US-006)

- ⬜ Create workshop Notion workspace template (pre-configured databases, sample stories)
- ⬜ Build one-click workshop environment setup (CDK deploy with workshop config)
- ⬜ Write workshop facilitator guide (include both discovery and delivery demo flows)
- ⬜ Write team adoption guide (how to set up for your own project)
- ⬜ Create sample user stories that demonstrate Code Agent, Mock Agent, Interview Agent, and Insight Agent
- ⬜ Performance tuning: optimize end-to-end latency for demo scenarios
- ⬜ Add spending limits and cost alerts for workshop accounts
- ⬜ Conduct dry-run workshop with internal team

### Knowledge Base for Production Tasks (US-007)

- ⬜ Implement Content Sync Lambda (Notion → Knowledge Base ingestion)
- ⬜ Configure Bedrock Knowledge Base with Notion content schema
- ⬜ Implement incremental sync (only changed pages)
- ⬜ Update Orchestrator to prefer Knowledge Base retrieval when available (fall back to on-demand Notion API)
- ⬜ Sync status visible in Notion configuration page

## Backlog

Items not yet scheduled for a sprint:

- ⬜ A/B Test skill: generate 2-3 variant approaches, each with prototype + Demo Deck (US-012)
- ⬜ Landing page generation skill for demand testing (US-013)
- ⬜ Skill chaining: Spec → Code → Review workflows (US-008)
- ⬜ Enriched PR descriptions with full Notion context (US-005, descoped — Notion MCP covers developer context needs)
- ⬜ Support for multiple GitHub repositories per project
- ⬜ Notion template gallery for different project types
- ⬜ Skill marketplace/registry: catalog of available skills per project with Notion-based management
- ⬜ Install Notion official skills for Claude (partner skills from Notion)
- ⬜ Integration with additional code hosting platforms (GitLab, CodeCommit) via MCP servers
- ⬜ Additional event source adapters for workspace tools beyond Notion (implement adapter interface, platform-specific MCP servers, result writers)
- ⬜ Notion sidebar app for inline agent interaction
- ⬜ Automated acceptance testing of generated code
- ⬜ Multi-language support for generated code (beyond initial target language)
- ⬜ Analytics dashboard for workshop outcomes (stories attempted, success rate, time-to-PR)
- ⬜ Support for Notion API v2 features as they become available
- ⬜ Multi-runtime deployment: separate AgentCore Runtimes per skill category for independent scaling

## Reference

### Sprint Assignment History

#### 2026-02-05: Event Source Abstraction (proposal: 20260205_event_source_abstraction.md)

| Change | Before | After | Reason |
|--------|--------|-------|--------|
| Webhook Handler | Notion-specific Lambda | Trigger Handler with adapter pattern | Separates platform-specific event handling from orchestration |
| Data Model | `notion_page_id` field | `source_type` + `source_page_id` fields | Platform-agnostic invocation tracking |
| Completion Handler | Direct Notion writes | Result writer pattern (delegates by `source_type`) | Separates platform-specific result delivery from shared logic |
| SQS Message | Notion-specific format | Canonical invocation format with `source_type` | Platform-agnostic contract between adapters and orchestrator |
| Requirements | No extensibility section | Section 5.7 Extensibility (REQ-NF-060 through REQ-NF-064) | Formalize platform separation as a requirement |

#### 2026-02-04: Claude Code Ecosystem Design (proposal: 20260204_claude_code_ecosystem_design.md)

| Change | Before | After | Reason |
|--------|--------|-------|--------|
| Agent Runtime | Strands Agents | Claude Agent SDK | Built-in tools, skills ecosystem, MCP integration out of the box |
| Agent Behavior | System prompt .md files | SKILL.md files | Open standard, auto-discovery, ecosystem compatibility |
| Tool Access | AgentCore Gateway (Sprint 1) | Direct MCP via .mcp.json (Sprint 1), Gateway (Sprint 3+) | Faster MVP, simpler infrastructure |
| Container Model | Separate container per agent type | Single container with all skills | Simplified deployment, skill-driven behavior |
| New User Stories | — | US-014 (Skill Management), US-015 (Built-in Tools) | Skill ecosystem and Claude Code capabilities |
| New Sprint 3 Tasks | — | Gateway migration, skill management | Phased approach to production governance |

#### 2026-02-04: Product Discovery Acceleration (proposal: 20260204_product_discovery_acceleration.md)

| User Story | Before | After | Reason |
|-----------|--------|-------|--------|
| US-001 | Sprint 1 | Sprint 1 | Unchanged — now paired with Mock Agent |
| US-002 | Sprint 1 | **Sprint 2** | Moved — grouped with discovery feedback loops for thematic cohesion |
| US-003 | Sprint 2 | **Sprint 3** | Shifted one sprint — discovery agents take priority |
| US-004 | Sprint 2 | **Sprint 3** | Shifted one sprint — discovery agents take priority |
| US-006 | Sprint 3 | **Sprint 4** | Shifted one sprint |
| US-007 | Sprint 3 | **Sprint 4** | Shifted one sprint |
| US-009 (new) | — | **Sprint 1** | Mock Agent: generate clickable prototypes for customer validation |
| US-010 (new) | — | **Sprint 2** | Demo Deck Agent: Notion-native demo experience (explain, show, collect feedback) |
| US-011 (new) | — | **Sprint 2** | Insight Agent: synthesize demo feedback + existing customer data (via Notion MCP) |

#### 2026-02-04: Notion MCP Competitive Analysis (proposal: 20260204_notion_mcp_competitive_analysis.md)

| User Story | Before | After | Reason |
|-----------|--------|-------|--------|
| US-001 | Sprint 1 | Sprint 1 | Unchanged — core differentiator |
| US-002 | Sprint 2 | Sprint 1 | Promoted — completes PO loop |
| US-005 | Sprint 1 | Backlog | Demoted — Notion MCP covers developer context needs |
| US-007 | Sprint 1 | Sprint 3 | Deferred — MVP uses on-demand Notion API reads |
| US-008 | Sprint 2 | Backlog | Deferred — medium value, high cost |

### Skill Inventory

All skills run on a single AgentCore Runtime powered by the Claude Agent SDK. Behavior is determined by SKILL.md files.

| Skill | Type | Input | Output | Sprint |
|-------|------|-------|--------|--------|
| `code-generation` | Code generation | User story + context | GitHub PR | 1 |
| `prototype-generation` | Prototype generation | User story | Hosted URL (S3/CloudFront) | 1 |
| `demo-deck` | Notion page + DB generation | User story + prototype URL | Notion page sequence + feedback DB | 2 |
| `insight-synthesis` | Multi-source analysis (via Notion MCP) | Feedback DB + customer tickets + any Notion DB | Notion page update (cross-referenced synthesis) | 2 |
| `ab-test` | Multi-variant generation | User story | 2-3 variants × (prototype + Demo Deck) + unified feedback DB | Backlog |
| `spec-generation` | Text generation | User story | Technical spec document | Backlog |
| `code-review` | Code analysis | Generated code + criteria | Review comments | Backlog |
| Developer-created | Project-specific | Varies | Varies | 3+ |

### Key Technologies

| Technology | Role | Sprint |
|-----------|------|--------|
| Claude Agent SDK | Agent runtime — Claude Code as a library | 1 |
| Agent Skills (SKILL.md) | Agent behavior configuration | 1 |
| Notion MCP (`@notionhq/notion-mcp-server`) | Notion workspace access via MCP | 1 |
| GitHub MCP (`@modelcontextprotocol/server-github`) | GitHub operations via MCP | 1 |
| API Gateway | Event source webhook endpoint | 1 |
| Lambda | Trigger handler (with adapters), orchestrator, completion handler (with result writers) | 1 |
| SQS | Invocation queue with DLQ | 1 |
| DynamoDB | Invocation records, project config | 1-3 |
| Bedrock AgentCore Runtime | Serverless microVM execution (hosts Claude Agent SDK) | 1 |
| Bedrock (Claude) | Foundation model (via `CLAUDE_CODE_USE_BEDROCK=1`) | 1 |
| S3 + CloudFront | Prototype hosting | 1 |
| Secrets Manager | Notion/GitHub credentials | 1 |
| CloudWatch | Logging, metrics, alarms, cost tracking | 1-3 |
| KMS | Encryption key management | 1 |
| AgentCore Gateway | Centralized MCP endpoint with auth + policy | 3 |
| Bedrock Knowledge Bases | Notion content storage and retrieval (production scale) | 4 |
