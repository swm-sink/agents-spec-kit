# Agent Specification: [AGENT NAME]

**Feature Branch**: `[###-agent-name]`
**Created**: [DATE]
**Status**: Draft
**Input**: User description: "$ARGUMENTS"

> This is the **agent variant** of the spec template. Use it when the feature you're building IS an AI agent (a bundle of Claude Code skills, subagents, or both) rather than ordinary application code. For non-agent features, use `templates/spec-template.md`.

## Agent Purpose *(mandatory)*

**Goal**: [One sentence: what the agent accomplishes for the user.]

**Primary user**: [Who invokes or benefits from this agent — end developer, support engineer, the user themselves, etc.]

**Out of scope**: [What this agent explicitly will not do. Prevents scope creep into "general-purpose helper".]

## Capabilities *(mandatory)*

<!--
  Each capability is something the agent can do, expressed as user value.
  Capabilities map 1:N to skills/subagents — the /speckit.agent.plan step decides
  which capabilities become skills, which become subagents, and which collapse.
  Do NOT name skills or subagents here. That's a planning decision.
-->

### Capability 1 — [short name] (Priority: P1)
**What**: [What the agent does, in plain language.]
**Why this priority**: [Value and rationale.]
**Acceptance**:
- **Given** [trigger / context], **When** [user input], **Then** [agent behavior].
- **Given** [edge case], **When** [user input], **Then** [agent behavior].

### Capability 2 — [short name] (Priority: P2)
**What**: [...]
**Why this priority**: [...]
**Acceptance**:
- **Given** [...], **When** [...], **Then** [...].

### Capability 3 — [short name] (Priority: P3)
**What**: [...]
**Why this priority**: [...]
**Acceptance**:
- **Given** [...], **When** [...], **Then** [...].

## Triggering *(mandatory)*

How does the agent know to act?

- **Invocation mode**: [Manual via slash command / Auto via skill description matching / Subagent dispatched by parent / Mix]
- **Trigger phrases or contexts**: [Concrete utterances or situations that should fire the agent.]
- **Anti-triggers**: [Situations that look similar but should NOT fire the agent.]

## Inputs & Outputs *(mandatory)*

- **Inputs**: [What the agent reads — user message, files, command output, prior conversation, external API.]
- **Outputs**: [What the agent emits — written files, message back to user, returned summary, side effects.]
- **Boundaries**: [What the agent must never read or write.]

## Tools the agent needs *(advisory)*

<!--
  This is intent, not implementation. The /speckit.agent.plan step turns this into
  concrete `allowed-tools` / `tools` frontmatter. List capabilities not specific tool names
  unless one is genuinely required (e.g. "must run git").
-->

- [Reading files in the workspace]
- [Searching the web]
- [Running shell commands — scope: ...]
- [Editing source files in directory X]
- [Calling external API: ...]

## Memory & state *(optional — include if the agent is stateful)*

- [What the agent needs to remember across invocations.]
- [Where state lives — file, vector store, none.]
- [Privacy / retention.]

## Evaluation *(mandatory)*

How will we know the agent works?

### Scenarios to evaluate
- **E-001**: [Realistic user scenario the agent must handle.] **Pass**: [Observable outcome.]
- **E-002**: [Failure mode the agent must NOT exhibit.] **Pass**: [Agent declines / escalates / asks for clarification.]
- **E-003**: [Edge case.] **Pass**: [Outcome.]

### Success criteria
- **SC-001**: [Measurable outcome — e.g., "Agent correctly classifies 9/10 sample issues into the right label set."]
- **SC-002**: [Quality bar — e.g., "Agent never proposes destructive git operations without a confirmation prompt."]
- **SC-003**: [Latency / cost — e.g., "P95 response under 30 seconds at default model."]

## Risks & guardrails *(mandatory)*

- **Risk**: [What could go wrong.] **Mitigation**: [How the agent prevents it.]
- **Risk**: [What could go wrong.] **Mitigation**: [...]

## Dependencies *(optional)*

- [External services the agent calls.]
- [Other agents/skills it composes with.]
- [Required Claude Code version, MCP servers, etc.]

## Assumptions

- [Things assumed about the user's environment or workflow that, if false, change the agent's design.]
