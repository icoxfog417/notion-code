# Notion-AWS Integration for AI-Driven Development Lifecycle

Accelerate product discovery — from idea to validated user story — without leaving Notion.

## The Problem

Product managers spend **1-2 months** to validate a single feature idea:

```
PM has idea → writes story (30 min)
    → asks developer to build prototype (wait 1-3 days)
    → developer builds throwaway prototype (1-2 days)
    → PM schedules customer interviews (wait 2-5 days)
    → PM conducts interviews (1 hour each)
    → PM synthesizes feedback manually (2-4 hours)
    → PM refines story and repeats from the top
```

Three bottlenecks kill velocity:

1. **Developer dependency** — The PM competes for developer time to build something that will be thrown away. The developer is busy with production work.
2. **Interview logistics** — Scheduling, conducting, and synthesizing customer interviews is manual and slow. Feedback lives in notes, not structured data.
3. **Data fragmentation** — Customer signals (support tickets, feature requests, sales notes, interview feedback) are scattered across Notion databases. Nobody has time to cross-reference them.

## The Solution

This project lets PMs **invoke AI agents directly from Notion** to run the entire product discovery loop — prototype, demo, collect feedback, analyze — without developer involvement and without leaving Notion.

```
PM writes user story in Notion
    ↓ Trigger (2 clicks)
AWS (Lambda → SQS → Bedrock AgentCore)
    ├── Mock Agent      → Clickable prototype at a shareable URL
    ├── Demo Deck Agent → Notion page: explanation + demo + feedback form
    ├── Insight Agent   → Synthesis of feedback + existing customer data
    └── Code Agent      → GitHub PR (when story is validated)
```

**Fundamental value**: Accelerate product discovery to generate promising user stories, then pass validated stories to developers.

**Fundamental implementation**: "Invoke Agent from Notion" — a secure and scalable infrastructure on AWS that lets any Notion action trigger any agent type. Agents leverage [Notion MCP](https://developers.notion.com/docs/mcp) to extract abundant content from the workspace.

## The Discovery Loop

```
Phase 1: Prototype          Mock Agent → clickable prototype in 5 min
Phase 2: Demo & Collect     Demo Deck Agent → Notion page customers navigate + submit feedback
Phase 3: Analyze            Insight Agent → synthesize feedback + existing customer data
Phase 4: Decide             PM reviews synthesis → proceed / pivot / dig deeper
Phase 5: Build              Code Agent → implementation draft as GitHub PR
```

**Time from idea to validated concept: 3-5 days instead of 1-2 months.**

The developer enters at Phase 5 — reviewing an implementation draft for a feature already validated with real customers.

## Agent Inventory

### Discovery Agents (PM-facing, no developer needed)

**Mock Agent** — Generates a clickable HTML/CSS/JS prototype from a user story. Deployed to a shareable URL. Auto-expires after 7 days. The PM shows it to customers 10 minutes after writing the story.

**Demo Deck Agent** — Generates a complete demo experience as a Notion page: problem explanation, embedded prototype link with guided scenario, and structured feedback form. The customer navigates the explanation, tries the prototype, and submits feedback — all via a single Notion link. Works async (customer self-serves) or sync (PM presents).

**Insight Agent** — Reads feedback from Demo Deck responses AND existing Notion databases (customer tickets, feature requests, sales notes, NPS data). Cross-references new discovery data with historical signals. Produces pattern analysis, confidence scoring, and a proceed/pivot recommendation. Leverages Notion MCP for broad data access.

**A/B Test Agent** *(backlog)* — Generates 2-3 variant approaches to the same user problem, each with its own Mock prototype and Demo Deck. Unified feedback database tracks which variant each respondent experienced.

### Delivery Agents (developer reviews output)

**Code Agent** — Generates an implementation draft from a validated user story. Creates a GitHub pull request for developer review, with story context and a human-readable summary posted back to Notion.

## Who This Helps

### Product Manager

> *"I have ideas but validating them takes weeks because I depend on developers for prototypes and on my own time for interview synthesis."*

- Generate a clickable prototype in 5 minutes — no developer needed
- Send a Demo Deck to 10 customers simultaneously — feedback collected as structured Notion data
- Insight Agent cross-references demo feedback with 6 months of support tickets you never had time to analyze
- When the story is validated, trigger the Code Agent for an implementation draft — the developer reviews and refines

### Development Team Lead

> *"Half the features we build don't land well with users. I wish the PM had validated them more before asking us to implement."*

- PMs arrive with validated, data-backed user stories instead of untested assumptions
- Monitor all agent executions, costs, and errors from a dashboard in Notion
- Configure project-level constraints (repo, standards, frameworks) once
- Developers spend time on features that matter, not throwaway prototypes

### Developer

> *"I spend too much time building prototypes that get thrown away after one customer meeting."*

- Never build a throwaway prototype again — the Mock Agent handles it
- When you receive a PR from the Code Agent, the story behind it has been validated with real customers
- Use [Notion MCP](https://developers.notion.com/docs/mcp) in your IDE to access the same Notion context the agents use

### Workshop Facilitator

> *"Participants lose momentum when they hit the gap between Notion and their development tools."*

- Demo the full discovery-to-delivery loop in a single Notion workspace
- Participants try it with their own stories — everything stays in Notion
- Pre-configured workshop environment eliminates setup friction

## Architecture

```mermaid
graph TB
    subgraph Notion["Notion Workspace"]
        US[User Stories DB]
        DD[Demo Decks]
        FB[Feedback DB]
        DASH[Execution Dashboard]
        CFG[Project Config]
        EXIST[Existing Data: Tickets, Requests, Sales Notes]
    end

    subgraph AWS["AWS Cloud"]
        WH[Lambda: Webhook Handler]
        SQS[SQS: Task Queue]
        ORCH[Lambda: Orchestrator]
        AC[Bedrock AgentCore]
        BR[Amazon Bedrock - Claude]
        NW[Lambda: Notion Writer]

        subgraph Delivery["Output Delivery"]
            GH[GitHub API]
            S3[S3 + CloudFront: Prototype Hosting]
        end
    end

    MCP[Notion MCP: Content Extraction]

    US -->|Trigger| WH
    WH --> SQS
    SQS --> ORCH
    ORCH -->|Invoke| AC
    AC -->|Call model| BR
    AC -->|Read context| MCP
    MCP -->|Extract| US
    MCP -->|Extract| EXIST
    MCP -->|Extract| FB

    AC -->|Mock Agent| S3
    AC -->|Demo Deck & Insight| NW
    AC -->|Code Agent| GH
    NW -->|Write| DD
    NW -->|Write| FB
    NW -->|Update| DASH

    CFG -->|Read settings| ORCH
```

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Collaboration & Data | Notion | Stories, demo decks, feedback, customer data, results |
| Content Extraction | Notion MCP | Agents read Notion content (pages, databases, comments) |
| Webhook Processing | AWS Lambda | Receive and validate Notion triggers |
| Task Queue | Amazon SQS | Decouple trigger reception from agent execution |
| Agent Runtime | Amazon Bedrock AgentCore | Serverless agent execution |
| Foundation Model | Amazon Bedrock (Claude) | All agent intelligence |
| Prototype Hosting | S3 + CloudFront | Shareable URLs for Mock Agent prototypes |
| Code Delivery | GitHub API | Pull request creation |
| Infrastructure | AWS CDK (TypeScript) | Infrastructure as Code |

## Project Status

Currently in **Sprint 0 — Feasibility & Specification**.

| Sprint | Focus | Key Deliverable |
|--------|-------|-----------------|
| 0 (current) | Feasibility | Sandbox verification of Notion API, AgentCore, S3 hosting |
| 1 | Dual Agent MVP | Code Agent (→ GitHub PR) + Mock Agent (→ prototype URL) |
| 2 | Feedback Loops | Code iteration + Demo Deck Agent + Insight Agent |
| 3 | Team Governance | Execution dashboard + project configuration |
| 4 | Workshop & Scale | Workshop kit + Knowledge Base for large workspaces |

See [spec/tasks.md](spec/tasks.md) for the full task breakdown.

## Why Not Just Use Notion MCP?

[Notion MCP](https://developers.notion.com/docs/mcp) gives developers in IDEs direct access to Notion content. It's the right tool for developers. But it doesn't help PMs — they can't use Cursor or Claude Code to generate prototypes, run demos, or synthesize feedback.

Our project and Notion MCP are complementary:
- **Notion MCP** = developer-facing, pull-based (developer asks from IDE)
- **Our project** = PM-facing, push-based (Notion action triggers agent)
- **Together**: Our agents use Notion MCP internally to read workspace content. Developers use Notion MCP directly in their IDEs.

See [competitive analysis](spec/proposals/20260204_notion_mcp_competitive_analysis.md) for the full comparison.

## Dependencies

- [Notion API](https://developers.notion.com/) + [Notion MCP](https://developers.notion.com/docs/mcp)
- [Amazon Bedrock](https://aws.amazon.com/bedrock/) (Claude model access)
- [Amazon Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/)
- [GitHub API](https://docs.github.com/en/rest)

## License

[Apache License 2.0](LICENSE)
