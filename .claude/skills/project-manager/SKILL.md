---
name: project-manager
description: Project status, sprint tracking, and onboarding for the Notion-AWS integration project. Use when asking about progress, available skills, what to work on next, or getting oriented with the project.
---

# Project Manager

You help team members understand project status, find relevant context, and identify what to work on next. You provide onboarding for new contributors and track sprint progress.

## Project Overview

**Project**: Notion-AWS Integration for AI-Driven Development Lifecycle

**Vision**: Accelerate product discovery from 1-2 months to 3-5 days by letting PMs invoke AI agents directly from Notion — prototype, demo, collect feedback, analyze — without developer involvement.

**Architecture**: Event source adapter pattern with Notion as launch platform, Claude Agent SDK on Bedrock AgentCore, Agent Skills for behavior configuration.

## Current Status

### Sprint 0: Feasibility & Specification

**Status**: Specification complete, sandbox verification pending

**Completed** (6 items):
- ✅ Requirements specification (v0.4.0)
- ✅ Design specification (v0.4.0)
- ✅ Notion MCP competitive analysis
- ✅ Product discovery acceleration proposal
- ✅ Claude Code ecosystem design proposal
- ✅ Event source abstraction proposal

**Pending Verification** (7 items):
| Task | Owner | Dependency | Status |
|------|-------|------------|--------|
| Claude Agent SDK on AgentCore | Agent Implementation | None | ⬜ TODO |
| Notion MCP via `.mcp.json` | MCP Integration | SDK working | ⬜ TODO |
| GitHub MCP via `.mcp.json` | MCP Integration | SDK working | ⬜ TODO |
| Agent Skills loading | Agent Implementation | SDK working | ⬜ TODO |
| Notion webhook capabilities | MCP Integration | None | ⬜ TODO |
| S3 + CloudFront + auto-expiry | AWS Infrastructure | None | ⬜ TODO |
| Latency and cost measurement | Agent Implementation | All MCP working | ⬜ TODO |

**Pending Documentation** (2 items):
- ⬜ Document findings in `implementation_qa.md`
- ⬜ Prepare stakeholder review materials

### Parallelization Opportunities

**Wave 1** (can start now, no dependencies):
- SDK on AgentCore (Agent Implementation)
- Notion webhook (MCP Integration)
- S3 + CloudFront (AWS Infrastructure)

**Wave 2** (after SDK working):
- Notion MCP, GitHub MCP (MCP Integration)
- Skills loading (Agent Implementation)

**Wave 3** (after Wave 2):
- Latency & cost measurement (Agent Implementation)

**Wave 4** (after Wave 3):
- Documentation & stakeholder materials (Technical Writer)

## Available Skills

| Skill | Description | Use When |
|-------|-------------|----------|
| `aws-infrastructure` | CDK, S3, CloudFront, Lambda, DynamoDB, AgentCore deployment | Building infrastructure, CDK stacks, IAM policies |
| `agent-implementation` | Claude Agent SDK, SKILL.md files, container setup, performance | Writing agents, skills, measuring latency/cost |
| `mcp-integration` | Notion MCP, GitHub MCP, webhook handlers | Configuring MCP servers, testing tools, webhooks |
| `technical-writer` | Q&A documentation, stakeholder materials, diagrams | Documenting findings, preparing presentations |
| `project-manager` | Status, onboarding, sprint tracking | Understanding progress, finding what to work on |

## Key Files

| File | Purpose | Read When |
|------|---------|-----------|
| `spec/requirements.md` | User stories and requirements | Understanding what to build |
| `spec/design.md` | Architecture and component design | Understanding how to build |
| `spec/tasks.md` | Sprint breakdown and task status | Finding work items |
| `spec/implementation_qa.md` | Verified technical patterns | Looking for proven solutions |
| `README.md` | Project overview | Getting oriented |
| `CLAUDE.md` | Workflow guidelines | Understanding contribution process |

## Key Proposals

| Proposal | Summary |
|----------|---------|
| `20260203_notion_aws_integration.md` | Initial architecture proposal |
| `20260204_notion_mcp_competitive_analysis.md` | Why we beat Notion MCP |
| `20260204_product_discovery_acceleration.md` | Mock/Demo/Insight agents |
| `20260204_claude_code_ecosystem_design.md` | Claude Agent SDK decision |
| `20260205_event_source_abstraction.md` | Platform-agnostic trigger pipeline |
| `20260205_business_strategy_and_revenue_model.md` | Revenue model and competitive moat |

## Sprint Roadmap

| Sprint | Focus | Key Deliverables |
|--------|-------|------------------|
| **0** (current) | Feasibility | Sandbox verification, spec finalization |
| **1** | Dual Agent MVP | Code Agent + Mock Agent end-to-end |
| **2** | Feedback Loops | Demo Deck + Insight Agent, code iteration |
| **3** | Governance | Monitoring dashboard, Gateway migration |
| **4** | Workshop | Workshop kit, Knowledge Base |

## Onboarding Checklist

New to the project? Follow this sequence:

1. **Read README.md** — understand the vision and value proposition
2. **Read CLAUDE.md** — understand contribution workflow
3. **Read `spec/requirements.md`** — understand user stories
4. **Read `spec/design.md`** — understand architecture
5. **Check `spec/tasks.md`** — find your sprint and tasks
6. **Review relevant proposals** — understand key decisions
7. **Check `spec/implementation_qa.md`** — learn from verified patterns
8. **Pick a skill** — use the skill matching your expertise

## Answering Common Questions

### "What should I work on?"

1. Check `spec/tasks.md` for your sprint's pending tasks
2. Look at the parallelization matrix above
3. Identify tasks in your expertise area (Wave 1 can start now)
4. Check dependencies — don't start Wave 2 tasks until Wave 1 completes

### "How do I verify a pattern?"

1. Create sandbox directory: `.sandbox/{feature}/`
2. Write minimal test code
3. Document findings in sandbox README.md
4. Prepare Q&A entry for `spec/implementation_qa.md`
5. Update task status in `spec/tasks.md`

### "Where is the architecture documented?"

- High-level: `README.md` (Architecture section)
- Detailed: `spec/design.md` (all sections)
- Diagrams: Mermaid in both files

### "What decisions have been made?"

- Check `spec/proposals/` for all major decisions
- Each proposal explains background, alternatives, and rationale
- The Sprint Assignment History in `spec/tasks.md` tracks changes

### "What's the business strategy?"

- See `spec/proposals/20260205_business_strategy_and_revenue_model.md`
- Key points: skill ecosystem as moat, hybrid revenue model, cross-platform expansion (internal strategy, not for README)

## Output Format

When answering project questions, provide:

1. **Direct answer** — specific, actionable
2. **Relevant file references** — where to find more detail
3. **Next steps** — what to do with this information
4. **Related skills** — which skill to use for follow-up work

## Updating This Skill

As the project progresses, update this skill with:
- Current sprint status changes
- New completed/pending items
- New proposals or decisions
- Updated parallelization matrix
