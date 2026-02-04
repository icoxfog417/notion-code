# Proposal: Claude Code Ecosystem Design — Agent SDK + Skills Architecture

**Date**: 2026-02-04
**Author**: Claude Agent (Co-Founder Review)
**Status**: Proposed

## Background

The current design (v0.2.0) specifies agent runtimes built with the **Strands Agents framework** — each agent is a Python `Agent` class with a `BedrockModel`, connected to AgentCore Gateway for tool access via MCP. This works, but requires us to build every agent capability from scratch (system prompts, tool integrations, execution logic).

Meanwhile, Anthropic has released the **Claude Agent SDK** — a library that exposes the full Claude Code runtime (tools, skills, MCP, session management, subagents) as a programmable API. Combined with the **Agent Skills** open standard, this creates an ecosystem where:

1. Agents get Claude Code's built-in tools (Read, Write, Edit, Bash, Glob, Grep, WebFetch, etc.) out of the box
2. Behavior is configured through **SKILL.md** files rather than custom Python code
3. External tools connect via **MCP servers** configured declaratively in `.mcp.json`
4. The skill ecosystem allows **developers to create capabilities that PMs can trigger from Notion**

This proposal recommends replacing the Strands Agents runtime with the Claude Agent SDK to accelerate delivery and enable the skill-based collaboration model.

## Research Summary

### Claude Agent SDK

- **Installation**: `pip install claude-agent-sdk` (Python) / `npm install @anthropic-ai/claude-agent-sdk` (TypeScript)
- **Core API**: `query(prompt, options)` — async generator that returns structured messages
- **Bedrock support**: Set `CLAUDE_CODE_USE_BEDROCK=1` + AWS credentials
- **Built-in tools**: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, Task (subagents), Skill, and more
- **Session management**: Session IDs for context persistence; `resume` parameter for multi-turn
- **Permission model**: `allowed_tools` for fine-grained control; `permission_mode` for broader access
- **Hooks**: PreToolUse, PostToolUse, Stop, SessionStart, SessionEnd — callbacks for validation and logging
- **Subagents**: Define specialized agents with `AgentDefinition(description, prompt, tools)` — invoked via Task tool

### Agent Skills

- **Format**: SKILL.md with YAML frontmatter (`name`, `description`) + Markdown instructions
- **Location**: `.claude/skills/` (project) or `~/.claude/skills/` (user)
- **Loading**: Set `setting_sources=["project"]` to enable; add `"Skill"` to `allowed_tools`
- **Discovery**: Claude auto-discovers skills by description; invokes them when context matches
- **Ecosystem**: Anthropic's official repo (`github.com/anthropics/skills`) + partner skills (Notion, Figma, Atlassian)
- **Notion partnership**: Notion has official skills for Claude — skills for working with Notion content

### AgentCore Runtime with Claude Agent SDK

- Reference implementation exists: `github.com/moritalous/claude-code-on-agentcore`
- Docker container with Node.js + `@anthropic-ai/claude-code` CLI + Python `claude-agent-sdk`
- Entrypoint uses `BedrockAgentCoreApp` from `bedrock_agentcore.runtime`
- MCP servers configured via `.mcp.json` (loaded automatically)
- Skills loaded from `.claude/skills/` directory
- Input/output via `/work/input` and `/work/output` directories
- Session ID routing: same ID → same microVM instance

### AgentCore CDK Constructs

- `agentcore.Runtime` — deploy container-based agents with versioning and endpoints
- `agentcore.Gateway` — unified MCP endpoint with auth, discovery, and policy
- `agentcore.Memory` — long-term memory with extraction strategies
- `agentcore.BrowserCustom` — cloud browser for web interaction
- `agentcore.CodeInterpreterCustom` — secure Python sandbox
- Gateway supports MCP server targets with OAuth credential providers
- Semantic tool search via `McpGatewaySearchType.SEMANTIC`

## Proposal

### 1. Replace Strands Agents with Claude Agent SDK

**Before (Strands)**:
```python
from strands import Agent
from strands.models import BedrockModel
from strands.tools.mcp.mcp_client import MCPClient

agent = Agent(
    model=BedrockModel(model_id="anthropic.claude-sonnet-4-20250514"),
    system_prompt=SYSTEM_PROMPT,
    tools=mcp_client.list_tools_sync()
)
result = agent(prompt)
```

**After (Claude Agent SDK)**:
```python
from claude_agent_sdk import query, ClaudeAgentOptions

async for message in query(
    prompt=prompt,
    options=ClaudeAgentOptions(
        cwd="/work",
        setting_sources=["project"],
        allowed_tools=["Skill", "Read", "Write", "Edit", "Bash", "Glob", "Grep",
                        "WebFetch", "Task", "mcp__notion__*", "mcp__github__*"],
        permission_mode="bypassPermissions",
        mcp_servers={
            "notion": {
                "command": "npx",
                "args": ["-y", "@notionhq/notion-mcp-server"],
                "env": {"NOTION_TOKEN": os.environ["NOTION_TOKEN"]}
            }
        }
    )
):
    yield message
```

**Key advantages**:
- Built-in tools (file ops, code execution, web access) available immediately
- Skills loaded from filesystem — no custom tool implementation
- MCP servers configured declaratively — no client code needed
- Session management built-in (resume, context persistence)
- Subagent support for complex multi-step workflows

### 2. Skill-Based Agent Architecture

Instead of 7 separate agent containers with hardcoded system prompts, we deploy a **single base runtime** with different **skill configurations** loaded per invocation context.

**Skill Structure**:
```
agents/
├── Dockerfile                    # Base: Node.js + claude-code + Python
├── main.py                       # AgentCore entrypoint using claude_agent_sdk
├── .mcp.json                     # MCP servers (Notion, GitHub, etc.)
├── CLAUDE.md                     # Base system prompt + project context
├── .claude/
│   └── skills/
│       ├── code-generation/
│       │   └── SKILL.md          # Code Agent behavior
│       ├── prototype-generation/
│       │   └── SKILL.md          # Mock Agent behavior
│       ├── demo-deck/
│       │   └── SKILL.md          # Demo Deck Agent behavior
│       ├── insight-synthesis/
│       │   └── SKILL.md          # Insight Agent behavior
│       └── [user-installed]/     # Developer-created skills
│           └── SKILL.md
└── requirements.txt
```

**Example SKILL.md (Code Generation)**:
```markdown
---
name: code-generation
description: Generate implementation code from a Notion user story and create a GitHub pull request. Use when the task involves code generation, implementation drafts, or PR creation.
---

# Code Generation Skill

Generate production-ready implementation code from user stories.

## Workflow

1. Read the user story from the provided Notion context
2. Analyze acceptance criteria and extract technical requirements
3. Read the project's coding standards and framework preferences
4. Generate implementation code following the project conventions
5. Generate test files for the implementation
6. Create a GitHub pull request with:
   - Descriptive title referencing the user story
   - PR body with acceptance criteria checklist
   - Link back to the Notion page

## Guidelines

- Follow the project's coding standards (provided in context)
- Generate tests alongside implementation
- Use realistic variable names and comments
- Reference the Notion page ID in PR description
- Create feature branches, never push to main
```

**How it works**:
- The orchestrator determines the agent_type from the Notion trigger
- The prompt sent to `query()` includes the user story content + instruction to use the appropriate skill
- Claude auto-discovers the relevant skill by its description and invokes it
- The skill's instructions guide Claude's behavior for that specific task

### 3. MCP Integration via `.mcp.json`

Tool access is configured declaratively rather than in code:

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "NOTION_TOKEN": "${NOTION_TOKEN}"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Gateway integration option**: For production deployments with centralized auth and policy, the Gateway MCP endpoint can be added as an HTTP/SSE server in `.mcp.json`:

```json
{
  "mcpServers": {
    "gateway": {
      "type": "http",
      "url": "${GATEWAY_MCP_ENDPOINT}",
      "headers": {
        "Authorization": "Bearer ${GATEWAY_TOKEN}"
      }
    }
  }
}
```

**Strategy evolution**:
- **Sprint 1 (MVP)**: Direct MCP server connections via `.mcp.json` — simpler, faster to ship
- **Sprint 3+ (Governance)**: Migrate to Gateway as centralized MCP endpoint — adds auth, policy, and observability

### 4. PM-Developer Collaboration Through Skills

This is the strategic differentiator. The skill model creates a natural collaboration boundary:

**Developer workflow**:
1. Developer identifies a capability PMs need (e.g., "generate a React dashboard prototype")
2. Developer creates a SKILL.md with specific instructions, coding patterns, and quality standards
3. Developer tests the skill locally with Claude Code
4. Developer commits the skill to the project's `.claude/skills/` directory
5. The skill is automatically available to the agent runtime on next deployment

**PM workflow**:
1. PM writes a user story in Notion (business requirements, acceptance criteria)
2. PM changes the Status property to "Generate Prototype" (or any configured trigger)
3. The agent runtime invokes the appropriate skill with the user story as context
4. Results are posted back to Notion

**Why this works**:
- Developers define "how" (coding standards, architecture patterns, framework choices) via skills
- PMs define "what" (user stories, acceptance criteria, business context) via Notion
- The agent bridges the gap — applying developer-defined skills to PM-defined requirements
- No custom code needed to add new agent capabilities — just a new SKILL.md

### 5. Single Runtime vs. Multi-Runtime

**Recommendation**: Start with a **single base runtime** with all skills pre-installed.

**Rationale**:
- Simplifies infrastructure (one container image, one ECR repo, one CDK construct)
- Skill loading is lazy (loaded on-demand, not all at startup)
- Different behavior is determined by the prompt + skill selection, not different containers
- AgentCore handles session isolation (separate microVMs per invocation)

**Future option**: If specific agent types need different resource profiles (e.g., Code Agent needs more compute for test execution), create specialized runtime variants with subsets of skills.

## Impact

### Requirements Changes

- **REQ-AO-002**: Update agent type descriptions to reference skill-based architecture
- **New REQ**: Add requirements for skill management (installation, creation, versioning)
- **New user story**: US-014 — Developer creates and manages skills for the agent runtime
- **Dependencies**: Add Claude Agent SDK as a core dependency

### Design Changes

- **Section 1.2 (Technology Stack)**: Replace Strands Agents with Claude Agent SDK
- **Section 4.4 (Gateway)**: Add note about MVP direct MCP vs. production Gateway
- **Section 4.5 (Agent Definitions)**: Rewrite from Strands Agent pattern to Claude Agent SDK + Skills
- **Section 5.1 (Container Structure)**: Update to include `.claude/skills/`, `.mcp.json`, `CLAUDE.md`
- **Section 5.2 (Agent Entrypoint)**: Rewrite from Strands to Claude Agent SDK `query()`
- **New section**: Skill Configuration — how skills are authored, loaded, and mapped to triggers

### Tasks Changes

- **Sprint 0**: Add sandbox verification tasks for Claude Agent SDK on AgentCore
- **Sprint 1**: Update agent implementation tasks to use Claude Agent SDK
- **Sprint 3+**: Add skill management and developer collaboration features
- **Backlog**: Add custom skill creation tooling

## Alternatives Considered

### Option A: Keep Strands Agents, Add Skills Later

Keep the current Strands-based architecture and build skill-like configurability ourselves.

**Rejected because**: We'd be building what Claude Agent SDK already provides (tool execution, skill loading, session management, MCP integration). Higher engineering cost, slower to ship.

### Option B: Use Claude Agent SDK with Multiple Runtimes per Agent Type

Deploy separate container images per agent type, each with Claude Agent SDK + type-specific skills.

**Deferred to future**: Adds infrastructure complexity for Sprint 1. Start with single runtime; split later if resource profiles diverge.

### Option C: Use AgentCore Gateway for All Tool Access from Day 1

Route all MCP traffic through Gateway even in Sprint 1.

**Deferred to Sprint 3**: Gateway adds configuration overhead. Direct MCP connections in `.mcp.json` are faster to set up for MVP. Migrate to Gateway when governance (auth, policy, audit) becomes a priority.

## Implementation Plan

1. Create this proposal (this document)
2. Update `requirements.md` — add skill ecosystem requirements and user story
3. Update `design.md` — replace Strands with Claude Agent SDK architecture
4. Update `tasks.md` — add SDK verification and skill creation tasks
5. Sprint 0 sandbox: verify Claude Agent SDK on AgentCore with Notion MCP
6. Sprint 1: implement agent runtime with Claude Agent SDK + initial skills

## References

- [Claude Agent SDK Overview](https://platform.claude.com/docs/en/agent-sdk/overview)
- [Agent Skills in the SDK](https://platform.claude.com/docs/en/agent-sdk/skills)
- [MCP Integration](https://platform.claude.com/docs/en/agent-sdk/mcp)
- [Anthropic Skills Repository](https://github.com/anthropics/skills)
- [Claude Code on AgentCore (Reference Implementation)](https://github.com/moritalous/claude-code-on-agentcore)
- [AWS Bedrock AgentCore CDK](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-bedrock-agentcore-alpha-readme.html)
- [Agent Skills Open Standard](https://agentskills.io)
