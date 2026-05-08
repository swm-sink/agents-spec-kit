---
description: Plan an AI agent — decide skills vs subagents, tool scoping, model choices, and emit agent-plan.md.
handoffs:
  - label: Scaffold the agent
    agent: speckit.agent.scaffold
    prompt: Scaffold the skills and subagents from the plan
    send: true
  - label: Design the evaluation
    agent: speckit.agent.eval
    prompt: Design the evaluation for this agent
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
---

## User Input

```text
$ARGUMENTS
```

## Outline

This is the **agent variant** of `/speckit.plan`. It reads `agent-spec.md` (produced by `/speckit.agent.specify`) and emits `agent-plan.md` — the design that `/speckit.agent.scaffold` will materialize.

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for `FEATURE_SPEC`, `IMPL_PLAN`, `SPECS_DIR`, `BRANCH`. Treat `FEATURE_SPEC` as the path to `agent-spec.md` (the script may write `spec.md` — read whichever exists).

2. **Load context**:
   - `agent-spec.md` from FEATURE_SPEC.
   - `memory/constitution.md`.
   - `templates/agent-plan-template.md` (already copied to IMPL_PLAN — but the path may be `plan.md`; reuse it as `agent-plan.md`).
   - `research/claude-code-skills-reference.md` and `research/claude-code-subagents-reference.md` — your sources of truth for valid frontmatter fields and naming rules.

3. **Build the skill / subagent inventory**. For each capability in the spec, decide one of:
   - **skill** — short procedure or reference, runs in parent context.
   - **subagent** — needs isolated context, long-running, parallel, or burns tokens.
   - **both** — a skill that delegates to a subagent for heavy work.

   Apply this rubric:
   - Capability fits in <500 lines of guidance and doesn't blow up the parent's context → **skill**.
   - Capability requires ranging over many files, or running for a while, or returning a single concise summary → **subagent**.
   - Capability has both a short user-facing entry point AND a long inner job → **both**.

   Fill the inventory table in `agent-plan-template.md`.

4. **Design each skill** (use `templates/skill-template/SKILL.md` shape):
   - Path: `.claude/skills/<name>/SKILL.md`.
   - Frontmatter: `name`, `description`. Add `allowed-tools` only if you want to scope tool access.
   - The description must contain the **trigger condition in plain language** plus concrete keywords. See `research/claude-code-skills-reference.md` §authoring for patterns. Bad: "Helps with reviews." Good: "Reviews staged git changes for security and style issues. Use after `git add` or when the user asks for a code review."
   - Body sections: When to use / How to use / Examples / Anti-patterns / References (when needed).

5. **Design each subagent** (use `templates/subagent-template.md` shape):
   - Path: `.claude/agents/<name>.md`.
   - Frontmatter: `name`, `description`, `tools`, `model: inherit` (default).
   - Description must follow the dispatch patterns in `research/claude-code-subagents-reference.md` §4 — start with "Use this agent when..." or "...specialist. Use proactively..." plus concrete keywords.
   - Tools: minimum-necessary. List explicit tool names (e.g., `Read, Grep, Glob, Bash`). Don't grant `Write`/`Edit` unless the subagent must mutate files.
   - Model: `inherit` unless there's a documented reason to pin (e.g., haiku for cheap classification, opus for hard reasoning).

6. **Tool & permission decisions** — fill the decisions table in the plan template. Default model `inherit`, no worktree, no background unless justified.

7. **Validate naming**:
   - Lowercase + hyphens only.
   - No collision with built-in subagents (`Explore`, `Plan`, `general-purpose`).
   - No two skills or subagents share a name.
   - No skill and subagent share a name (avoid ambiguity for the user).

8. **Constitution check**: re-read `memory/constitution.md`. If any planned skill/subagent violates a principle, note it in the Complexity Tracking table with justification or rework the design.

9. **Write the plan** to `agent-plan.md` (rename IMPL_PLAN's `plan.md` to `agent-plan.md` if needed). Fill every section of `templates/agent-plan-template.md`.

10. **Validate the plan against the spec**:
    - Every capability in `agent-spec.md` is covered by at least one row in the inventory.
    - Every "tool the agent needs" intent in the spec is reflected in at least one skill/subagent's `tools` (or explicitly justified as not needed).

11. **Report completion**: plan path, skill/subagent count, any unresolved decisions, and prompt the user to run `/speckit.agent.scaffold` to write the files.

## Guidelines

- Plans don't write the skill/subagent files — `/speckit.agent.scaffold` does that. Plans say what *will* be written.
- Resist creating a subagent for everything. Subagents have overhead — isolated context, separate model spin-up, the parent has to brief them. Use them when isolation buys something real.
- Tool lists and model overrides should match what's documented in `research/claude-code-skills-reference.md` and `research/claude-code-subagents-reference.md`. If you want to use a frontmatter field marked `[observed, undocumented]` there, call it out as a Complexity Tracking entry.
