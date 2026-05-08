# Claude Code Subagents: Complete Reference

A comprehensive guide to understanding, building, and using Claude Code subagents for the agents-spec-kit toolkit.

**Version**: Based on Claude Code documentation as of May 2026
**Sources**: https://code.claude.com/docs/en/sub-agents.md, https://code.claude.com/docs/en/agent-teams.md, https://code.claude.com/docs/en/tools-reference.md

---

## 1. What is a Subagent?

### Definition & Core Concept

A **subagent** is a specialized AI assistant that handles specific tasks within Claude Code. Unlike a slash command, skill, or CLAUDE.md memory that apply locally, a subagent is a distinct Claude instance spawned by the parent with its own isolated context window, custom system prompt, and independent tool restrictions. (source: https://code.claude.com/docs/en/sub-agents.md)

When the main Claude Code session encounters a task matching a subagent's description, it automatically delegates to that subagent, which completes the work in its own context and returns only a summary to the parent. This isolation preserves the parent's context window by keeping verbose output (search results, logs, test runs) out of the main conversation.

### Subagent vs Related Concepts

| Concept | Scope | Context | Persistence | Invocation |
|---------|-------|---------|-------------|-----------|
| **Subagent** | Single task, delegated | Own context window | Restartable within session | Automatic (via description match) or explicit |
| **Skill** | Reusable workflow | Parent's context | Repeatable across sessions | Manual via `Skill` tool or Claude choice |
| **CLAUDE.md** | Project memory | Loaded at start | Persistent across sessions | Auto-loaded, no explicit invoke |
| **Slash command** | Interactive utility | Same as session | Session-scoped only | Manual only (e.g., `/agents`, `/init`) |

### Lifecycle

1. **Spawn**: Main Claude Code session uses the `Agent` tool to spawn a subagent, passing a task description and optional parameters.
2. **Initialize**: The subagent loads its definition (frontmatter fields + markdown body as system prompt), establishes its own context window, and inherits permissions from the parent.
3. **Isolation**: The subagent works independently in a fresh context, unable to see the parent's conversation history (unless using fork mode—see Section 9).
4. **Return**: Upon completion, the subagent returns a summary message to the parent. Tool calls and detailed output remain in the subagent's transcript.
5. **Cleanup**: When the session ends or the subagent finishes, subagent transcripts persist in `~/.claude/projects/{project}/{sessionId}/subagents/` for up to 30 days (configurable via `cleanupPeriodDays`).

### Parent vs. Child Context

**What the subagent sees:**
- Its own system prompt (the markdown body of the definition)
- Basic environment details (working directory, project files)
- The task description passed at spawn time
- Project context (CLAUDE.md, MCP servers from parent's configuration, if applicable)

**What the subagent does NOT see:**
- The parent's conversation history
- Previous messages in the main session
- The parent's system prompt

This isolation is the key benefit: exploration and debugging work stays out of the parent's limited context window.

---

## 2. File Layout & Scope

Subagents are defined as Markdown files with YAML frontmatter. The location determines scope and priority.

### Directory Structure

```
~/.claude/agents/                          # User-level (priority 4)
  code-reviewer.md
  test-runner.md

.claude/agents/                            # Project-level (priority 3)
  db-migrator.md
  security-auditor.md

<managed-settings>/.claude/agents/         # Org-managed (priority 1)
  compliance-checker.md
```

### Scope & Priority Table

| Location | Scope | Priority | Visibility | How to Create | Notes |
|----------|-------|----------|-----------|---------------|-------|
| Managed settings `.claude/agents/` | Organization-wide | 1 (highest) | All users in org | Deployed by admin | Overrides all other definitions |
| `--agents` CLI flag | Current session only | 2 | This session only | Pass JSON at launch | Useful for automation; not persisted |
| `.claude/agents/` | Project-specific | 3 | This project + team | Interactive `/agents` or manual file | Committed to version control; shared with team |
| `~/.claude/agents/` | Personal, all projects | 4 | Your machine only | Interactive `/agents` or manual file | Available in every project; not shared |
| Plugin's `agents/` directory | Plugin scope | 5 (lowest) | Where plugin enabled | Bundled with plugin | Cannot use hooks, mcpServers, permissionMode |

**Conflict Resolution**: When multiple scopes define an agent with the same name, the higher-priority definition wins. For example, if both `.claude/agents/test-runner.md` and `~/.claude/agents/test-runner.md` exist, the project-level version is used.

**Best Practices**:
- **Project subagents** (`.claude/agents/`): Check into version control for team use and evolution.
- **User subagents** (`~/.claude/agents/`): Personal tools reused across projects.
- **CLI-defined** (`--agents` flag): Quick testing or headless automation.
- **Managed subagents**: Enforce org standards (e.g., compliance, security policies).

---

## 3. Agent Definition Frontmatter

### Format & Structure

```yaml
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code changes for quality, security, and maintainability.
tools: Read, Grep, Glob, Bash
model: sonnet
permissionMode: acceptEdits
memory: project
effort: high
isolation: worktree
color: blue
---

# Markdown system prompt here.
You are a senior code reviewer...
```

The frontmatter (YAML block between `---`) defines configuration. The markdown below becomes the subagent's system prompt.

### Supported Frontmatter Fields

All fields except `name` and `description` are optional and inherit defaults.

| Field | Type | Required | Default | Example | Notes |
|-------|------|----------|---------|---------|-------|
| `name` | string | **Yes** | N/A | `code-reviewer` | Unique identifier; lowercase letters and hyphens only |
| `description` | string | **Yes** | N/A | `"Expert code review specialist. Use proactively..."` | Claude reads this to decide when to delegate. Must be clear and specific. |
| `tools` | string or list | No | Inherit all | `Read, Grep, Glob, Bash` or `["Read", "Bash"]` | Allowlist of tools. Omit or use `*` to inherit all tools from parent. Use `Agent(worker, researcher)` syntax to restrict subagent spawning. See Section 5. |
| `disallowedTools` | string or list | No | None | `Write, Edit` | Tools to exclude. Applied before `tools` is resolved. If both are set, `disallowedTools` first, then `tools` allowlist. |
| `model` | string | No | `inherit` | `sonnet`, `opus`, `haiku`, `claude-opus-4-7`, or `inherit` | Model for this subagent. Default inherits parent's model. Overridable per-invocation. |
| `permissionMode` | string | No | `inherit` | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan` | Permission checking mode. Parent's `bypassPermissions` or `acceptEdits` always takes precedence and cannot be overridden. |
| `maxTurns` | integer | No | Unlimited | `10` | Maximum agentic turns before subagent stops. Limits reasoning depth. |
| `skills` | list | No | None | `["api-conventions", "error-handling"]` | Skills to preload into context at startup. Full content is injected, not just names. |
| `mcpServers` | list | No | None | `["github", "slack"]` or inline definitions | MCP servers available to this subagent. Can reference already-configured servers or define inline. Inline servers connect at startup, disconnect at finish. Ignored for plugin subagents. |
| `hooks` | object | No | None | See Section 6 | Lifecycle hooks (PreToolUse, PostToolUse, Stop). Ignored for plugin subagents. |
| `memory` | string | No | None | `user`, `project`, or `local` | Persistent memory directory scope. See Section 3. |
| `background` | boolean | No | `false` | `true` | Run as background task by default. Claude can still override at invocation time. |
| `effort` | string | No | Inherit session | `low`, `medium`, `high`, `xhigh`, `max` | Reasoning effort level. Overrides session's effort when active. |
| `isolation` | string | No | None | `worktree` | Run in isolated git worktree for file changes. Auto-cleanup if no changes made. |
| `color` | string | No | Default | `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` | Display color in task list and transcript. |
| `initialPrompt` | string | No | None | `"Run tests and report failures"` | Auto-submitted as first turn when agent runs as main session via `--agent` flag. |

[observed, undocumented] The `effort` field maps to model-specific reasoning levels (e.g., "extended thinking" for Opus).

### Model Resolution Order

When a subagent is invoked, the model is resolved in this order (first match wins):

1. `CLAUDE_CODE_SUBAGENT_MODEL` environment variable (if set)
2. Per-invocation `model` parameter (passed by parent when spawning)
3. Subagent definition's `model` frontmatter field
4. Parent session's model

This allows dynamic model selection without redefining the subagent file.

---

## 4. Description-Writing for Parent Dispatch

Claude reads each subagent's `description` field to decide when to delegate. A well-written description is critical for reliable triggering.

### Description Patterns

**Pattern 1: "Use this agent when..."**
```yaml
description: "Use this agent when you need to review code for quality, security, and performance."
```
This pattern directly tells Claude the condition for delegation. It's the clearest and most reliable.

**Pattern 2: "Use proactively..."**
```yaml
description: "Code review specialist. Use proactively after code changes to catch issues early."
```
The "use proactively" convention signals to Claude that it should delegate without waiting for an explicit request. This is effective when a task naturally follows another (e.g., reviewing after writing code).

**Pattern 3: Domain-specific expertise**
```yaml
description: "SQL query optimizer. Specializes in writing efficient database queries for performance."
```
Describe the domain and what makes this subagent unique. Claude delegates when it detects tasks in that domain.

**Pattern 4: Include examples or keywords**
```yaml
description: "API endpoint debugger. Use for troubleshooting HTTP errors, status codes, or request/response issues."
```
Concrete keywords ("HTTP errors", "status codes") make matching more reliable than abstract descriptions.

### Anti-Patterns (Avoid)

- **Too generic**: "General purpose agent" (Claude will not delegate predictably)
- **Too narrow**: "Fixes bugs in the auth module at line 42" (too specific; reusability lost)
- **No use case**: "My helper agent" (Claude doesn't know when to use it)
- **Unclear domain**: "Does things" (unusable)

### Best Practices

1. **Start with a verb + domain**: "Code reviewer", "Database analyst", "Test runner"
2. **Add specific trigger conditions**: "When analyzing data", "After code changes", "For performance issues"
3. **Keep it to 1–2 sentences**: Longer descriptions don't improve delegation reliability.
4. **Use "proactively" for reflexive tasks**: Tasks that logically follow other work.
5. **Test via conversation**: Ask Claude to name the agent when it would delegate; iterate if it doesn't match your expectation.

---

## 5. Tool Scoping & Isolation

### Tool Restrictions

Subagents inherit all tools from the parent by default. Restrict tools using `tools` (allowlist) or `disallowedTools` (denylist).

#### Allowlist (Recommended)

```yaml
---
name: code-reviewer
description: Review code for quality and security
tools: Read, Grep, Glob, Bash
---
```
This subagent can **only** use Read, Grep, Glob, and Bash. It cannot edit files, write files, or call MCP tools. Allowlists are explicit and secure.

#### Denylist

```yaml
---
name: general-worker
description: A capable agent for most tasks
disallowedTools: Write, Edit
---
```
This subagent inherits all tools **except** Write and Edit. It keeps Bash, MCP tools, and everything else.

#### Combined (Rare)

If both are set, `disallowedTools` is applied first, then `tools` is resolved against the remaining set. A tool listed in both is removed.

```yaml
---
name: specialized-agent
tools: Read, Bash, Grep
disallowedTools: Bash
---
```
Result: subagent gets only `Read` and `Grep` (Bash removed by denylist).

### Agent Tool Syntax

When a subagent is the main session (launched with `--agent` flag), it can spawn other subagents using the `Agent` tool. Restrict which subagents can be spawned with `Agent(agent_type)` syntax:

```yaml
---
name: coordinator
description: Coordinates work across specialized agents
tools: Agent(worker, researcher), Read, Bash
---
```
This subagent can spawn only `worker` and `researcher` subagents. Attempting to spawn any other type fails, and the subagent sees only the allowed types in its prompt.

To allow spawning any subagent:
```yaml
tools: Agent, Read, Bash
```

To block all subagent spawning:
```yaml
tools: Read, Bash
# Agent is omitted; cannot spawn any subagents
```

This restriction applies only when the agent runs as a main session. Regular subagents cannot spawn other subagents, so `Agent(...)` syntax has no effect in subagent definitions.

### MCP Server Scoping

Use `mcpServers` to give a subagent access to MCP servers not available in the main conversation:

```yaml
---
name: browser-tester
description: Tests features in a real browser
mcpServers:
  # Inline definition: scoped to this subagent only
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  # Reference by name: reuses parent's connection
  - github
---
```

Inline servers (defined here) connect when the subagent starts and disconnect when finished. Named references (e.g., `github`) reuse the parent session's existing connection.

Benefit: Keep expensive or security-sensitive servers out of the main conversation context entirely.

[observed, undocumented] Plugin subagents cannot define `mcpServers` (same restriction applies to `hooks` and `permissionMode`).

### Worktree Isolation

Set `isolation: worktree` to run the subagent in a temporary git worktree, giving it an isolated copy of the repository:

```yaml
---
name: experimental-refactor
description: Refactor code in isolation for review
isolation: worktree
---
```

**Behavior:**
- Subagent creates a new branch and worktree.
- File edits are made to the worktree, not the main checkout.
- The worktree is automatically cleaned up if the subagent makes no changes.
- If changes are made, the branch is left for review and integration.

**Use case:** When a subagent's changes might conflict with main-session work, or when you want to review changes before integrating.

### Background Subagents

By default, subagents run in the foreground (blocking). Set `background: true` to run in the background:

```yaml
---
name: continuous-linter
description: Lints code and reports issues
background: true
---
```

**Foreground** (blocking):
- Main conversation waits for subagent to finish.
- Permission prompts and clarifying questions surface to the user.

**Background** (concurrent):
- Subagent runs while you continue working in the main session.
- Pre-approval: Claude Code prompts for all needed permissions upfront; subagent gets auto-deny for anything not pre-approved.
- Results arrive as a message when the subagent finishes.
- If permission is missing, tool call fails but subagent continues (no interactive prompts).

Use background for verbose operations (testing, linting, research) that don't require interactive steering.

---

## 6. Hooks: Lifecycle & Conditional Execution

### Hooks in Subagent Frontmatter

Define hooks directly in the subagent's markdown file. These hooks run only while that subagent is active.

```yaml
---
name: code-reviewer
description: Review code for quality and best practices
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
---
```

**Common Events for Subagents:**

| Event | Matcher | Fires When | Example |
|-------|---------|-----------|---------|
| `PreToolUse` | Tool name (e.g., `Bash`, `Edit`) | Before tool use | Validate command before execution |
| `PostToolUse` | Tool name | After tool completes | Run linter after file edits |
| `Stop` | (none) | When subagent finishes | Cleanup, final reporting |

When a subagent's `Stop` hook fires, it is automatically converted to `SubagentStop` at runtime. (source: https://code.claude.com/docs/en/sub-agents.md)

### Project-Level Hooks for Subagent Events

Configure hooks in `settings.json` that respond to subagent lifecycle in the main session:

```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [
          { "type": "command", "command": "./scripts/setup-db.sh" }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          { "type": "command", "command": "./scripts/cleanup-db.sh" }
        ]
      }
    ]
  }
}
```

| Event | Matcher | Fires When |
|-------|---------|-----------|
| `SubagentStart` | Agent name (e.g., `db-agent`) | When subagent begins execution |
| `SubagentStop` | Agent name | When subagent completes |

### Example: Read-Only Database Queries

Combine `tools: Bash` with a `PreToolUse` hook to allow only SELECT queries:

```yaml
---
name: db-reader
description: Execute read-only database queries
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---
```

Hook script (`./scripts/validate-readonly-query.sh`):
```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Block SQL write operations
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE)\b' > /dev/null; then
  echo "Blocked: Only SELECT queries allowed" >&2
  exit 2
fi

exit 0
```

Exit code 2 blocks the operation and feeds the error message back to Claude. (source: https://code.claude.com/docs/en/sub-agents.md)

---

## 7. Built-in vs. Custom Subagents

### Built-in Subagents

Claude Code includes several built-in subagents that Claude automatically uses when appropriate. Each inherits the parent's permissions with tool restrictions. (source: https://code.claude.com/docs/en/sub-agents.md)

#### Explore
- **Model**: Haiku (fast, low-latency)
- **Tools**: Read-only (Denied: Write, Edit)
- **Purpose**: File discovery, code search, codebase exploration
- **Trigger**: Claude needs to search or understand code without making changes
- **Thoroughness**: Claude specifies `quick`, `medium`, or `very thorough` when invoking

Explore keeps search results out of your main conversation, preserving context.

#### Plan
- **Model**: Inherits from main conversation
- **Tools**: Read-only
- **Purpose**: Codebase research during plan mode
- **Trigger**: When in plan mode and Claude needs to understand the codebase
- **Mechanism**: Prevents infinite nesting while gathering context

#### General-purpose
- **Model**: Inherits from main conversation
- **Tools**: All tools
- **Purpose**: Complex, multi-step tasks requiring both exploration and action
- **Trigger**: Task requires exploration + modification, complex reasoning to interpret results, or multiple dependent steps

#### Other Built-in Agents
- `statusline-setup` (Sonnet): Invoked when running `/statusline` command
- `claude-code-guide` (Haiku): Invoked when asking questions about Claude Code features

### When to Write a Custom Subagent

Write a custom subagent when:

1. **Repeated patterns**: You keep spawning the same kind of worker with the same instructions
2. **Special tool restrictions**: You need to enforce specific tool access (e.g., read-only database access)
3. **Specialized prompts**: You want a focused system prompt for a specific domain (e.g., security review, data analysis)
4. **Cost control**: You want to route work to a cheaper, faster model (e.g., Haiku)
5. **Team reuse**: The subagent is used across projects or by teammates

If you need a reusable workflow that runs in the main conversation context (not isolated), use a **Skill** instead.

---

## 8. Annotated Example: Code Reviewer Subagent

Here's a complete, production-ready subagent definition with annotations:

```markdown
---
# Metadata: unique identifier, must be lowercase with hyphens
name: code-reviewer

# Description: Claude reads this to decide when to delegate.
# Use "Proactively" pattern to signal automatic delegation after code changes.
description: |
  Expert code review specialist. Proactively reviews code for quality, security,
  and maintainability. Use immediately after writing or modifying code.

# Tool restrictions: allowlist of what this subagent can use.
# Read, Grep, Glob for exploration; Bash for running git diff.
# Notably: no Edit or Write. This subagent only reads, it doesn't modify.
tools: Read, Grep, Glob, Bash

# Model: inherit from parent session (typically Sonnet or Opus).
# Could specify "haiku" for cheaper review if cost is important.
model: inherit

# Permission mode: inherit parent's permission checks.
# Could be "acceptEdits" to auto-approve file operations, but this agent
# doesn't modify files, so it's not needed.
permissionMode: default

# Optional: persistent memory across conversations.
# project scope = shareable via version control (recommended for shared codebases)
memory: project

# Optional: run in background by default (can be overridden per-invocation).
# Not set here; defaults to foreground so review results are interactive.

# Optional: color for display in transcript.
color: blue
---

# System Prompt (Markdown below the frontmatter)
# This is what the subagent "knows" and how it should behave.

You are a senior code reviewer ensuring high standards of code quality, security,
and maintainability.

## When Invoked

1. Use `git diff` to see recent changes
2. Focus exclusively on modified files
3. Begin review immediately without asking for context

## Review Checklist

- **Code clarity**: Is the code clear and readable?
- **Naming**: Are functions, variables, and classes well-named?
- **DRY principle**: Is there duplicated logic that should be extracted?
- **Error handling**: Are errors handled appropriately?
- **Security**: No exposed secrets, API keys, or SQL injection vulnerabilities
- **Input validation**: User input is validated before use
- **Test coverage**: Are there adequate tests for the changes?
- **Performance**: Are there obvious performance issues?

## Output Format

Organize feedback by priority:

### Critical Issues (must fix)
- [Issue description]
- Current code: [snippet]
- Suggested fix: [improved code]

### Warnings (should fix)
- [Issue description with rationale]

### Suggestions (consider improving)
- [Nice-to-have improvement with rationale]

## Consulting Your Memory

Before starting, check your agent memory for:
- Common patterns and conventions in this codebase
- Recurring issues you've seen before
- Architectural decisions that affect this review

After reviewing, update your memory with:
- New patterns or conventions you discovered
- Common issues to watch for in future reviews
```

### Why This Works

1. **Clear description**: Explicitly says "Use proactively" + defines the trigger ("after code changes") → Claude reliably delegates
2. **Restricted tools**: Only read-only tools + Bash (for git diff). No Edit/Write means the subagent can't accidentally modify code
3. **Reusable across projects**: Placed in `~/.claude/agents/` or `.claude/agents/`, it's available everywhere without duplication
4. **Persistent memory**: Over time, the subagent learns codebase patterns and improves its reviews
5. **Specific prompt**: Detailed checklist + output format make reviews consistent and actionable
6. **Inherit model**: Uses the parent's model (often Sonnet), balancing capability and cost

---

## 9. Composition Patterns

### Subagent-to-Subagent: Not Directly Allowed

Subagents **cannot spawn other subagents**. (source: https://code.claude.com/docs/en/sub-agents.md) Attempting to use the `Agent` tool from within a subagent fails.

**Workarounds**:

1. **Chain subagents from the main session**: Ask Claude to invoke one subagent, then another:
   ```
   Use the code-reviewer subagent to find issues, then use the debugger subagent to fix them.
   ```
   The main session orchestrates; subagents work in sequence.

2. **Use Skills instead**: If you need reusable logic within a subagent's context, define a Skill and reference it in the subagent's `skills` field (frontmatter). Skills run inline and can call other skills.

3. **Design single-purpose subagents**: Rather than nesting, design subagents to be self-contained.

### Subagents vs. Skills: Choosing the Right Tool

| Factor | Subagent | Skill |
|--------|----------|-------|
| **Context** | Own isolated context window | Parent's context |
| **Reusability** | Fixed configuration; can be invoked by parent or discovered via Agent tool | Reusable workflows; discoverable by Claude or manually invoked |
| **Memory** | Can have persistent memory directory | No memory; state reset each session |
| **Tool access** | Restricted via frontmatter | Inherits parent's tools; can use custom MCP tools |
| **Composition** | Cannot nest subagents (directly) | Skills can call other skills |
| **Best for** | Isolating verbose tasks (testing, logging, searching); enforcing tool restrictions | Reusable prompt-based workflows within parent context |

**Decision framework**:
- **Use a Skill** if you want a reusable workflow that stays in the conversation and shares context
- **Use a Subagent** if you want isolation (to save context), tool restrictions, or a specialized model

### Subagents in Plugins

Subagents can be bundled with plugins and distributed across teams. (source: https://code.claude.com/docs/en/sub-agents.md)

**Plugin subagent limitations**:
- Cannot define `hooks`, `mcpServers`, or `permissionMode` (these fields are ignored)
- To use hooks or custom MCP access, copy the agent file into `.claude/agents/` or `~/.claude/agents/`

**Distribution**: Plugin subagents appear in the `/agents` interface and typeahead as `<plugin-name>:<agent-name>`.

---

## 10. Best Practices for Parents Calling Subagents

### Self-Contained Prompts

Subagents don't see the parent's conversation history. Every invocation must be self-contained:

```text
✅ GOOD:
Use the code-reviewer subagent to review the authentication module at src/auth/.
Focus on token handling, session management, and input validation.

❌ BAD:
Review the auth code. (Subagent doesn't know which files or what was just changed)
```

### The Briefing Pattern

Structure subagent invocations with:

1. **Goal**: What should the subagent accomplish?
2. **Context**: What does it need to know upfront?
3. **Tried**: What has already been tried (if applicable)?
4. **Needed**: What specific output do you want back?
5. **Scope**: Response length cap (optional but helpful for context management).

```text
Goal: Identify performance bottlenecks in the API layer.

Context: The app uses Express.js with PostgreSQL. Database queries in src/db/
are already optimized with indexes.

Tried: Basic profiling with `npm run profile` shows the issue is in middleware.

Needed: List the top 3 slowest middleware functions, with suggestions to optimize each.
Keep findings under 500 words.

Use the code-optimizer subagent to investigate.
```

### "Never Delegate Understanding"

Don't ask subagents to understand context they fundamentally can't gather:

```text
✅ GOOD:
Use the test-runner subagent to run the test suite at tests/unit/ and report
failing tests with their error messages. Keep it under 1000 words.

❌ BAD:
The tests are failing. Fix them using a subagent.
(Subagent can't run tests without knowing the test command, framework, or error context)
```

### Parallelization

Spawn multiple subagents in a single message for independent work:

```text
In parallel using separate subagents:
- Use the code-reviewer subagent to review src/auth/
- Use the test-runner subagent to run the test suite and report failures
- Use the security-auditor subagent to scan for vulnerabilities

Report back when all three are done.
```

Each subagent runs concurrently in its own context, results are synthesized back to the parent.

### Continuation via SendMessage

To continue a subagent's work instead of spawning fresh, use the `SendMessage` tool (requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`):

```text
Continue the code-reviewer subagent's work. Now analyze the authorization layer.
```

The subagent retains its full history and picks up where it stopped. This is cheaper than spawning fresh.

(source: https://code.claude.com/docs/en/sub-agents.md)

---

## 11. Open Questions & Gotchas

### Known Limitations

1. **Subagents cannot spawn subagents**: Direct nesting is forbidden. Chain from the main session instead.

2. **Context isolation is strict**: Subagents don't inherit conversation history. Every spawn is fresh context. Plan accordingly with briefing patterns.

3. **Plugin subagents cannot use hooks/MCP/permissions**: Security restriction. Copy to project or user scope if you need these features.

4. **Forked subagents are experimental**: Enable via `CLAUDE_CODE_FORK_SUBAGENT=1`. Behavior may change.

5. **Bash working directory doesn't persist in subagents**: Each Bash call runs from the project root. `cd` commands don't carry over between tool calls. Use `isolation: worktree` if you need persistent directory state.

6. **Model resolution can be surprising**: Multiple sources (env var, per-invocation param, frontmatter, parent model) compete. Test to verify which wins.

### [Observed, Undocumented] Behaviors

- **Memory auto-load**: If a subagent has `memory: project`, it loads the first 200 lines / 25KB of `MEMORY.md` at startup and is given instructions to maintain it.

- **Background task pre-approval**: Background subagents get pre-approved for permissions upfront (Claude Code prompts once for all needed tools). Foreground subagents prompt per tool use.

- **Effort level mapping**: The `effort` field appears to map to Claude's reasoning levels. Higher effort = more reasoning, higher token cost.

- **Subagent transcript files**: Stored at `~/.claude/projects/{project}/{sessionId}/subagents/agent-{agentId}.jsonl`. Persist for up to 30 days.

### Testing & Debugging Tips

1. **Test the description**: Ask Claude to name which subagent it would use for a task. Iterate until it matches your intention.

2. **Use `/agents` to inspect**: The interactive `/agents` command shows all loaded subagents and their configuration.

3. **Check the transcript**: Subagent work is logged separately. Review the transcript file if the subagent behaves unexpectedly.

4. **Dry-run with plan mode**: Use plan mode in the main session to see Claude's reasoning about which subagent to use.

5. **Start with small tool allowlists**: Restrict tools aggressively at first, then relax if needed. It's easier to add tools than debug unintended side effects.

---

## 12. Summary: Subagent Decision Tree

Use this flowchart to decide when and how to use subagents:

```
Do you need isolated context for a task?
├─ YES: Task is verbose (logging, testing, searching) or you want tool restrictions
│   └─ Does Claude need to dynamically decide when to delegate?
│       ├─ YES: Create a subagent with a clear description
│       │   ├─ Does the subagent need persistent learning?
│       │   │   ├─ YES: Add `memory: project` or `memory: user`
│       │   │   └─ NO: Omit memory field
│       │   ├─ Does the subagent need special tool access?
│       │   │   ├─ YES: Use `tools: ...` allowlist or `mcpServers: ...`
│       │   │   └─ NO: Inherit all tools or use `disallowedTools`
│       │   └─ Does the subagent need file isolation?
│       │       ├─ YES: Add `isolation: worktree`
│       │       └─ NO: Omit isolation field
│       └─ NO: Invoke the subagent explicitly via @-mention or natural language
└─ NO: Task needs parent's conversation context
    └─ Is it a reusable workflow?
        ├─ YES: Write a Skill instead
        └─ NO: Keep it in the main conversation
```

---

## References & Source Links

- [Claude Code Subagents Documentation](https://code.claude.com/docs/en/sub-agents.md)
- [Claude Code Agent Teams Documentation](https://code.claude.com/docs/en/agent-teams.md) (related feature)
- [Claude Code Tools Reference](https://code.claude.com/docs/en/tools-reference.md)
- [Claude Code How It Works](https://code.claude.com/docs/en/how-claude-code-works.md) (context management)
- [Claude Code Hooks Guide](https://code.claude.com/docs/en/hooks) (lifecycle and conditional execution)

**Document Version**: May 2026  
**Last Verified**: 2026-05-08
