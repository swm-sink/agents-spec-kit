---
description: Create the specification for an AI agent (a bundle of Claude Code skills, subagents, or both) from a natural-language description.
handoffs:
  - label: Plan the agent
    agent: speckit.agent.plan
    prompt: Plan the agent. Decide skills vs subagents, tool scoping, and emit agent-plan.md.
    send: true
  - label: Clarify the spec
    agent: speckit.clarify
    prompt: Clarify open questions in the agent spec
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

The text the user typed after `/speckit.agent.specify` IS the agent description. Treat it as authoritative — do not ask the user to repeat it unless empty.

This command is the **agent variant** of `/speckit.specify`. Use it when the feature being built IS an AI agent (skills + subagents) rather than ordinary application code. For non-agent features, use `/speckit.specify` instead.

Given the agent description, do this:

1. **Generate a concise short name** (2–4 words) for the branch:
   - Action-noun where possible. Examples: "issue-triager" → `issue-triager`; "PR review agent" → `pr-reviewer`.
   - Prefer the agent's primary verb + domain.

2. **Find the next feature number** the same way `/speckit.specify` does:
   - `git fetch --all --prune`
   - Search remote branches, local branches, and `specs/` directories for `N-<short-name>` matches; take the highest N and use N+1.
   - Run `{SCRIPT}` with `--number N+1` and `--short-name "<short-name>"` plus the description.
   - Parse JSON output for `BRANCH_NAME` and `SPEC_FILE` paths.

3. **Load the agent-spec template** at `templates/agent-spec-template.md`. (NOT `spec-template.md` — this is the agent variant.)

4. **Read the research artifacts** if present, since they shape what the spec can promise:
   - `research/claude-code-skills-reference.md`
   - `research/claude-code-subagents-reference.md`
   - `research/design.md`

5. **Execute the spec workflow**:
   1. Parse the user description.
   2. Identify the agent's **purpose** (one sentence) and **out-of-scope** boundaries.
   3. Decompose into **capabilities** (one capability = one piece of user value). Each capability gets a priority (P1/P2/P3) and acceptance scenarios in Given/When/Then form. Do NOT name skills or subagents yet — that's a planning decision.
   4. Identify **triggering**: how does the agent know to act? Is it manual (slash command), automatic (skill description match), delegated (subagent dispatch by parent), or a mix?
   5. Identify **inputs and outputs** at the agent boundary (what it reads, what it emits, what it must never touch).
   6. List the **tools the agent needs** as intent (e.g., "reads files in src/", "runs git commands", "calls GitHub API"), not specific Claude Code tool names.
   7. Note **memory/state** if the agent is stateful; otherwise skip.
   8. Define **evaluation scenarios and success criteria** — how we'll know the agent works.
   9. Capture **risks and guardrails**.
   10. List **dependencies** and **assumptions**.

6. **Apply the spec-kit clarification rules** (same as `/speckit.specify`):
   - Make informed guesses for unspecified details; document in Assumptions.
   - Maximum 3 `[NEEDS CLARIFICATION: ...]` markers, prioritized by scope > security > UX > technical.
   - If there are clarifications, present them in the same options-table format as `/speckit.specify` step 6.c.

7. **Write the spec** to SPEC_FILE (which the script created — but rename it to `agent-spec.md` instead of `spec.md` if the script wrote `spec.md`). Use the `templates/agent-spec-template.md` structure exactly, replacing placeholders with concrete content.

8. **Quality validation**:
   Create `FEATURE_DIR/checklists/agent-requirements.md` with these items:

   ```markdown
   # Agent Specification Quality Checklist: [AGENT NAME]

   **Purpose**: Validate agent spec before /speckit.agent.plan.

   ## Content quality
   - [ ] No implementation details (no skill names, no tool names, no Claude Code internals)
   - [ ] Focused on what the agent does, not how it's built
   - [ ] All mandatory sections completed

   ## Capability completeness
   - [ ] Each capability has acceptance scenarios in Given/When/Then form
   - [ ] Capabilities are independently testable (you could ship just P1 and have value)
   - [ ] Out-of-scope is explicit

   ## Triggering
   - [ ] Trigger phrases or contexts are concrete (not "when relevant")
   - [ ] Anti-triggers documented (situations that look similar but should NOT fire)

   ## Evaluation
   - [ ] At least 3 evaluation scenarios listed
   - [ ] At least one scenario tests a failure mode the agent must NOT exhibit
   - [ ] Success criteria are measurable

   ## Notes
   - Items marked incomplete require spec updates before /speckit.agent.plan
   ```

   Run validation. If items fail, fix the spec and re-run (max 3 iterations).

9. **Report completion**: branch name, agent-spec path, checklist results, and prompt the user to run `/speckit.agent.plan` next.

## Quick guidelines

- The spec describes **what the agent does and how we'd know**, not which skills/subagents implement it. Naming skills here is a planning leak.
- "An agent that helps with code reviews" is not enough — push for the specific user, the trigger, and at least three concrete capabilities.
- If the user hasn't said how the agent gets invoked (slash command, automatic skill, subagent), make a reasonable default and call it out in Assumptions.
- The agent-spec template lives at `templates/agent-spec-template.md`. Always use it; do not regenerate the structure from memory.
