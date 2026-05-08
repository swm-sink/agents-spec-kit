---
description: Materialize the agent — write the .claude/skills/ and .claude/agents/ files described in agent-plan.md.
handoffs:
  - label: Design the evaluation
    agent: speckit.agent.eval
    prompt: Design an evaluation for the scaffolded agent
    send: true
  - label: Validate the bundle
    agent: speckit.implement
    prompt: Run validate-agent-bundle on the new files
---

## User Input

```text
$ARGUMENTS
```

## Outline

This command materializes an agent design. It reads `agent-plan.md` and writes real files into the user's project under `.claude/skills/` and `.claude/agents/`.

1. **Locate the plan**:
   - The current branch tells you the feature directory: `specs/<branch-name>/`.
   - Read `agent-plan.md` from there. If only `plan.md` exists, that's the plan — read it.
   - If neither exists, ERROR: "Run /speckit.agent.plan first."

2. **Read the templates** that govern the file shapes:
   - `templates/skill-template/SKILL.md`
   - `templates/subagent-template.md`
   - `research/claude-code-skills-reference.md` and `research/claude-code-subagents-reference.md` — for valid frontmatter only.

3. **Pre-flight checks**:
   - Confirm `.claude/` exists at the project root, or create it.
   - For each skill in the inventory: confirm `.claude/skills/<name>/SKILL.md` does NOT already exist. If it does, ask the user before overwriting.
   - For each subagent in the inventory: confirm `.claude/agents/<name>.md` does NOT already exist. Same overwrite rule.

4. **Use the bundled subagents** to actually write the files (these subagents ship in this repo's own `.claude/agents/`):
   - For each skill row in the inventory, dispatch the `skill-author` subagent with a self-contained brief: the skill's name, description, allowed-tools, body outline, and the path to write to. The subagent writes the file and returns a summary.
   - For each subagent row, dispatch the `subagent-author` subagent the same way.
   - You may dispatch multiple authors in parallel (single message, multiple Agent calls) when the rows are independent.

5. **Validate the bundle**: Invoke the bundled `validate-agent-bundle` skill on the directory `.claude/`. It checks:
   - Required frontmatter fields present and non-empty.
   - Descriptions are concrete (not generic placeholders).
   - Tool lists are minimum-necessary and use real tool names.
   - No name collisions with built-ins.
   - SKILL.md files have all five recommended sections (When to use / How to use / Examples / Anti-patterns / References).
   If validation fails, fix the issues in-place (re-dispatch the relevant author subagent with the validator's feedback) and re-validate.

6. **Optional independent review**: If the user asked for it (e.g., the description of the command included "and review"), dispatch the `agent-reviewer` subagent to read the new files and produce a review report in `specs/<branch-name>/scaffold-review.md`. Otherwise skip.

7. **Update agent-context** if the project has it: run the `update-agent-context.sh` script (the spec-kit one) to refresh whatever AGENTS.md / CLAUDE.md the project uses.

8. **Report completion**:
   - List every file written, with relative paths.
   - Surface any validator warnings that didn't block scaffolding.
   - Prompt next: `/speckit.agent.eval` to design the evaluation.

## Guardrails

- **Never write outside `.claude/skills/` and `.claude/agents/`** without an explicit instruction in the plan. If a skill needs an associated `scripts/foo.sh`, that lives under `.claude/skills/<name>/scripts/foo.sh`.
- **Refuse to overwrite without confirmation**, even if the plan calls for a name that already exists.
- **Don't invent frontmatter fields.** Only use what's in `research/claude-code-skills-reference.md` and `research/claude-code-subagents-reference.md` as documented (not `[observed, undocumented]`) — unless the plan explicitly justified it under Complexity Tracking.
- **Stay self-contained.** This command does not run the agent's evaluation, does not commit the files, does not push. Those are separate steps.
