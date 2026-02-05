# Proposal: Event Source Abstraction Layer

**Date**: 2026-02-05
**Author**: Claude Agent
**Status**: Proposed

## Background

The current design hardcodes Notion as the sole trigger source throughout the architecture — from the webhook handler to the data models to the completion handler. While Notion is our launch platform, the architecture should cleanly separate the "event source" concern from the "agent orchestration" concern. This is standard event-driven architecture practice and enables the system to accept triggers from additional workspace tools in the future without redesigning the core pipeline.

This proposal introduces an **Event Source Adapter** pattern that keeps Notion as the primary (and initially only) implementation while making the boundary explicit in the design.

## Proposal

### 1. Event Source Adapter in Webhook Handler

Rename the "Webhook Handler" concept to a more general "Trigger Handler" that delegates to platform-specific adapters:

```
Trigger Handler Lambda
├── Notion Adapter (validates Notion webhook, reads page, resolves trigger)
├── [Future adapters share the same SQS output format]
└── → SQS Message (platform-agnostic invocation request)
```

The SQS message schema becomes the **canonical invocation format** — the contract between event sources and the orchestrator. Any event source adapter must produce this format.

### 2. Data Model Extensions

Add `source_type` to the invocation record and generalize field naming:

| Field | Change | Rationale |
|-------|--------|-----------|
| `source_type` | **New field** (enum: `notion`, extensible) | Identifies which platform triggered the invocation |
| `source_page_id` | **Rename** from `notion_page_id` | Generic identifier for the triggering entity |
| `workspace_id` | Keep as-is | Already generic enough |

The `source_type` determines which completion handler adapter writes results back to the originating platform.

### 3. Completion Handler Adapters

Mirror the trigger side — the completion handler uses `source_type` to determine how to write results back:

```
Completion Handler Lambda
├── Read source_type from invocation record
├── Notion Writer: update Notion page, write to dashboard DB
├── [Future writers for other platforms]
└── Common: update DynamoDB, track costs
```

### 4. Skill Platform Independence

Skills (SKILL.md files) remain platform-agnostic by design. They receive context as structured input and produce artifacts as output. The platform-specific behavior (how to read context, how to deliver results) is handled by MCP servers and the completion handler, not by skills.

This is already the case in the current design — skills use `mcp__notion__*` tools but don't hardcode Notion assumptions in their logic. The MCP configuration (`.mcp.json`) is the only platform-specific piece within the agent runtime.

### 5. README and User-Facing Documents

No changes to the product positioning. The README continues to describe Notion as the primary platform. The event source abstraction is an internal architectural decision, not a product message.

## Impact

- **Requirements**: Add non-functional requirements for trigger extensibility and platform-agnostic skill design
- **Design**: Update data models, webhook handler, and completion handler sections to use the adapter pattern
- **Tasks**: Add backlog items for the abstraction layer; no sprint changes

## Alternatives Considered

### Alternative A: Keep Notion-specific design, refactor later
- **Rejected.** Introducing the abstraction now costs almost nothing (field naming, adapter pattern in design docs). Refactoring later requires data migration and API changes.

### Alternative B: Build full multi-platform support now
- **Rejected.** Over-engineering. We only need the abstraction boundary, not implementations for platforms we don't support yet.

## Implementation Plan

1. Update design.md: data models, webhook handler, completion handler
2. Update requirements.md: add extensibility non-functional requirements
3. Update tasks.md: add backlog items for future adapter implementations
4. README.md: minimal adjustments to architecture description
