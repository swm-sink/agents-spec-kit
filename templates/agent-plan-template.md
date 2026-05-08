# Agent Implementation Plan: [AGENT NAME]

**Branch**: `[###-agent-name]` | **Date**: [DATE] | **Spec**: [link to agent-spec.md]

> Translates `agent-spec.md` (the WHAT) into the concrete files we will scaffold (the HOW). The `/speckit.agent.scaffold` step reads this plan to emit `.claude/skills/` and `.claude/agents/`.

## Summary

[One paragraph: the agent's purpose, its top-level shape (N skills + M subagents), and the key delivery decisions.]

## Skill / subagent inventory

For each capability in `agent-spec.md`, decide: skill, subagent, or both. Use this rubric:

- **Skill** = invoked when its description matches; runs in the parent's context; good for short procedures, checklists, format references.
- **Subagent** = isolated context; receives a self-contained prompt; returns one summary; good for long-running exploration, parallel work, isolation from the parent's context budget.
- **Both** = a skill that delegates to a subagent for the heavy lifting.

| ID | Capability (from spec) | Form | Name | Rationale |
|---|---|---|---|---|
| C1 | [capability 1] | skill | `[skill-name]` | [why a skill, not a subagent] |
| C2 | [capability 2] | subagent | `[subagent-name]` | [why isolation matters here] |
| C3 | [capability 3] | both | `[skill]` → `[subagent]` | [why this composition] |

## Skill designs

For each row marked `skill` or `both` above:

### `[skill-name]`
- **Path**: `.claude/skills/[skill-name]/SKILL.md`
- **Description (frontmatter)**: [the exact one-line description Claude sees, tuned for triggering — see `research/claude-code-skills-reference.md` §authoring]
- **allowed-tools**: [comma-separated tools, or omit to inherit; cite reasoning]
- **Body sections**: When to use / How to use / Examples / Anti-patterns / References — note any deviations.
- **Reference files**: [list any `references/*.md` files this skill needs and what's in them]

## Subagent designs

For each row marked `subagent` or `both` above:

### `[subagent-name]`
- **Path**: `.claude/agents/[subagent-name].md`
- **Description (frontmatter)**: [exact description, tuned for parent dispatch — see `research/claude-code-subagents-reference.md` §4]
- **tools**: [Read, Write, Edit, ...] — minimum needed
- **model**: `inherit` unless there's a reason to pin (e.g., haiku for cheap classification, opus for hard reasoning)
- **System prompt outline**: [bullet list of what the body covers — role, job, how to operate, what NOT to do, briefing pattern]

## Tool & permission decisions

| Decision | Choice | Why |
|---|---|---|
| Default model | inherit | [or pin reason] |
| Worktree isolation | no | [or yes — when subagent edits files we want isolated] |
| Background execution | no | [or yes — for long subagent runs] |
| MCP servers required | none | [or list — and how the user installs them] |

## Files emitted by `/speckit.agent.scaffold`

```
.claude/
├── skills/
│   ├── [skill-1]/SKILL.md
│   └── [skill-2]/SKILL.md
└── agents/
    ├── [subagent-1].md
    └── [subagent-2].md
```

If the project doesn't already have `.claude/`, scaffold creates it.

## Validation gate

Before the scaffold step writes files, the `validate-agent-bundle` skill must pass:
- Every SKILL.md has a non-empty `name` and `description`.
- Every subagent .md has a non-empty `name` and `description`.
- Descriptions are not generic ("does things", "general purpose helper").
- Tool lists are minimum-necessary.
- No skill or subagent name collides with a built-in (`Explore`, `Plan`, `general-purpose`).

## Evaluation plan summary

Reference `eval-spec.md` (produced by `/speckit.agent.eval`). Note here:
- Number of scenarios: [N]
- Pass threshold: [%]
- Who runs the eval: [user / CI / one-shot during scaffold]

## Complexity tracking

> Fill ONLY if any decision violates the constitution or our own design principles.

| Violation | Why needed | Simpler alternative rejected because |
|---|---|---|
| [e.g., introduces a custom MCP server] | [justification] | [why a plain skill wouldn't work] |
