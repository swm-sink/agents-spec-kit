---
description: Design an evaluation specification for an AI agent — scenarios, scorers, datasets, and a run plan.
handoffs:
  - label: Implement the evaluation
    agent: speckit.implement
    prompt: Implement the evaluation spec (e.g., wire up Inspect AI tasks)
---

## User Input

```text
$ARGUMENTS
```

## Outline

This command produces `eval-spec.md` for the current agent feature — a runnable evaluation specification shaped after Inspect AI (UKGovernmentBEIS/inspect_ai) conventions, but plain Markdown so it works regardless of which eval runtime the user picks.

1. **Locate the feature**:
   - Current branch → `specs/<branch-name>/`.
   - Read `agent-spec.md` (capabilities, evaluation scenarios, success criteria).
   - Read `agent-plan.md` if present (so we know which skills/subagents we're evaluating).
   - If `agent-spec.md` is missing, ERROR: "Run /speckit.agent.specify first."

2. **Use the bundled `agent-eval-designer` subagent** when the eval design needs depth (more than five scenarios, multi-turn evaluation, or a real dataset). For a simple agent, do it inline. The decision rule:
   - >5 scenarios OR external dataset OR multi-turn → dispatch `agent-eval-designer`.
   - Otherwise → produce the eval spec yourself.

3. **Produce `specs/<branch-name>/eval-spec.md`** with this structure:

   ```markdown
   # Agent Evaluation Spec: [AGENT NAME]

   **Branch**: `[###-agent-name]` | **Spec**: [link to agent-spec.md] | **Plan**: [link]

   ## What we're evaluating
   [The agent and the slice of capabilities under test in this eval.]

   ## Scenarios
   For each evaluation scenario from agent-spec.md (E-001, E-002, ...) plus any new ones surfaced during planning:

   ### S-001 — [scenario name]
   - **Setup**: [Initial state — files, environment, prior conversation.]
   - **Input**: [What's given to the agent.]
   - **Expected behavior**: [What the agent should do, observable from outside.]
   - **Scorer**: [How we score pass/fail — exact match, regex, model-graded with rubric, human review.]
   - **Pass threshold**: [N of M trials pass — e.g., 9/10.]
   - **Maps to**: [SC-NNN success criterion(s) from spec.]

   [...repeat for each scenario...]

   ## Dataset
   - **Source**: [Hand-written / sampled from production / synthetic / external benchmark.]
   - **Size**: [N items.]
   - **Storage**: `specs/<branch>/eval-data/` (or external path).
   - **Format**: JSONL with fields [list fields].

   ## Scorers
   For each scorer kind referenced in scenarios:

   ### [scorer-name]
   - **Type**: [exact / regex / model-graded / structural / human]
   - **Definition**: [The rule, regex, or grading rubric.]

   ## Run plan
   - **Runtime**: [Inspect AI / a custom Python harness / a shell script / "manual one-shot review".]
   - **Frequency**: [On every scaffold change / weekly / one-shot.]
   - **Pass bar**: [Aggregate threshold for the eval as a whole.]
   - **Cost / latency budget**: [If model calls cost real money — note it.]

   ## Out of scope
   [Anything we explicitly are NOT evaluating in v1 of the agent.]

   ## Risks
   - **Risk**: [Eval-specific risk — e.g., dataset is too small, scorer is gameable.] **Mitigation**: [How we address it.]
   ```

4. **Quality validation** — check the eval spec against:
   - Every `SC-NNN` in `agent-spec.md` is referenced by at least one scenario via "Maps to".
   - Every scenario has a scorer and a pass threshold.
   - At least one scenario tests a failure mode (an anti-pattern the agent must NOT exhibit).
   - The pass bar in Run plan is concrete (e.g., "≥80% across all scenarios"), not "looks reasonable".

5. **Report completion**:
   - Path to `eval-spec.md`.
   - Number of scenarios and any missing success-criteria coverage.
   - Suggest the user run `/speckit.implement` to wire up the eval (e.g., turn each scenario into an Inspect AI task) — this is out of v1 scope for this command itself.

## Guidelines

- Eval spec is not Eval implementation. We produce a plan a human or a follow-on agent can execute. The runtime choice (Inspect AI, promptfoo, hand-rolled) is the user's.
- Scorers should be the simplest thing that works. Exact match > regex > model-graded > human. Reach for model-graded only when output is open-ended.
- Anti-pattern scenarios are non-optional. If the agent has a failure mode worth noting in `agent-spec.md` Risks, it deserves a scenario.
