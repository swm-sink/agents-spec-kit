# agents-spec-kit Design

**Status:** Draft v1 (May 2026) — synthesized from `repo-survey.md`, `claude-code-skills-reference.md`, `claude-code-subagents-reference.md`.

## What we're building

A fork-extension of `github/spec-kit` focused on **building AI agents** — specifically Claude Code skills and subagents — through spec-driven development. Two deliverables ship together:

1. **The kit:** new spec-kit slash commands and templates that walk a developer from "I want an agent that does X" to a validated, evaluable, packaged set of skills + subagents on disk.
2. **The bundled examples:** a starter library of working Claude Code skills and subagents that *use* the kit and *help build more agents*. Self-hosting from day one.

## Core insight from research

Three findings shape the design:

- **YAML/Markdown-with-frontmatter is the universal agent-spec medium.** SKILL.md (Anthropic), `agents.yaml` (CrewAI), declarative YAML (MS Agent Framework), `.af` (Letta), `.continue/config.yaml` all converge. Spec-kit's existing `templates/spec-template.md` Markdown shape is already aligned — we just need agent-specific sections.
- **The skill-folder convention is settled:** `<name>/SKILL.md` + optional `scripts/`, `references/`, `assets/`. Anthropic, wshobson/agents, obra/superpowers, davepoon/buildwithclaude all match. Our scaffolds emit exactly this.
- **There's a real gap:** no canonical generator that bridges `SKILL.md` ↔ subagent `.md` ↔ MCP server scaffold ↔ Inspect-AI eval spec from one declarative source. That's our wedge.

## Scope decisions

**In scope (v1):**
- Spec-kit slash commands for the agent workflow: `/speckit.agent.specify`, `/speckit.agent.plan`, `/speckit.agent.scaffold`, `/speckit.agent.eval`.
- Templates: agent-spec template (extends spec-template), skill template, subagent template.
- A starter library of 5 skills and 5 subagents that *help build agents*.
- Quality gates: a validator skill/subagent that checks SKILL.md and agent .md files for the format problems we documented.
- Documentation that points users at the research artifacts.

**Out of scope (v1):**
- MCP server scaffold generator (research notes captured for v2).
- A2A protocol scaffolding.
- Eval execution runtime (we emit Inspect-AI-shaped spec; running it is the user's job).
- Bidirectional CrewAI / Letta `.af` adapters (v2).
- Plugin marketplace publish flow.

## What ships in this repo (v1)

```
agents-spec-kit/
├── research/                                    # Already shipped
│   ├── repo-survey.md
│   ├── claude-code-skills-reference.md
│   ├── claude-code-subagents-reference.md
│   └── design.md                                # This file
├── templates/
│   ├── commands/
│   │   ├── agent.specify.md                     # NEW: /speckit.agent.specify
│   │   ├── agent.plan.md                        # NEW: /speckit.agent.plan
│   │   ├── agent.scaffold.md                    # NEW: /speckit.agent.scaffold
│   │   └── agent.eval.md                        # NEW: /speckit.agent.eval
│   ├── agent-spec-template.md                   # NEW
│   ├── agent-plan-template.md                   # NEW
│   ├── skill-template/                          # NEW (folder template)
│   │   ├── SKILL.md
│   │   └── references/example.md
│   └── subagent-template.md                     # NEW
├── .claude/                                     # Bundled, self-hosting examples
│   ├── skills/
│   │   ├── scaffold-skill/SKILL.md              # Build a new skill
│   │   ├── scaffold-subagent/SKILL.md           # Build a new subagent
│   │   ├── validate-agent-bundle/SKILL.md       # Lint a skills/subagents bundle
│   │   ├── write-skill-description/SKILL.md     # Author triggerable descriptions
│   │   └── design-eval-spec/SKILL.md            # Sketch an Inspect-AI eval
│   └── agents/
│       ├── agent-architect.md                   # Designs an agent from a goal
│       ├── skill-author.md                      # Writes a SKILL.md from a brief
│       ├── subagent-author.md                   # Writes a subagent .md from a brief
│       ├── agent-reviewer.md                    # Reviews skill/subagent quality
│       └── agent-eval-designer.md               # Drafts an evaluation plan
└── AGENTS.md                                    # Updated with agent-building section
```

## The agent-building workflow (user POV)

```
1. /speckit.agent.specify  "An agent that triages incoming GitHub issues..."
      → produces specs/NNN-issue-triage/agent-spec.md
        (capability description, success criteria, skill/subagent inventory,
         tool requirements, eval scenarios — all WITHOUT implementation)

2. /speckit.agent.plan
      → produces specs/NNN-issue-triage/agent-plan.md
        (decisions: which skills, which subagents, which tools, model picks,
         worktree isolation, deployment target)

3. /speckit.agent.scaffold
      → emits actual files into the user's project:
          .claude/skills/triage-issue/SKILL.md
          .claude/agents/issue-triager.md
        Uses the bundled `skill-author` / `subagent-author` subagents.
        Validates with `validate-agent-bundle` skill before writing.

4. /speckit.agent.eval
      → produces specs/NNN-issue-triage/eval-spec.md
        (Inspect-AI-shaped task specification + dataset stub)
```

Each step has a checklist gate (mirroring the existing spec-kit pattern from `templates/commands/specify.md`).

## The five bundled skills

Each is a real working skill that uses the format documented in `research/claude-code-skills-reference.md` and validates clean against `validate-agent-bundle`.

| Skill | Triggers when | Reads | Emits |
|---|---|---|---|
| **scaffold-skill** | User asks to create a new Claude Code skill | brief, target directory | `<name>/SKILL.md` plus optional `references/`, `scripts/` |
| **scaffold-subagent** | User asks to create a new Claude Code subagent | brief, target directory, allowed-tools intent | `.claude/agents/<name>.md` |
| **validate-agent-bundle** | User asks to lint or validate skills/subagents | a directory of `.claude/skills/` and/or `.claude/agents/` | findings list with severity |
| **write-skill-description** | User asks how to phrase a SKILL.md description, or descriptions aren't triggering | existing description text | rewritten description tuned for reliable triggering |
| **design-eval-spec** | User asks how to evaluate an agent | agent spec + scenarios | Inspect-AI-shaped task specification |

## The five bundled subagents

Each follows the description-writing patterns documented in `research/claude-code-subagents-reference.md`.

| Subagent | Use when (parent's view) | Tools | Model |
|---|---|---|---|
| **agent-architect** | Parent needs to turn a vague goal into a concrete agent design (which skills, which subagents, which tools) | Read, WebSearch, WebFetch | inherit |
| **skill-author** | Parent needs to write a working SKILL.md from a one-paragraph brief | Read, Write, Edit | inherit |
| **subagent-author** | Parent needs to write a working subagent .md from a one-paragraph brief | Read, Write, Edit | inherit |
| **agent-reviewer** | Parent needs an independent review of skill/subagent quality before shipping | Read, Grep, Glob | inherit |
| **agent-eval-designer** | Parent needs to draft an evaluation plan (datasets, scorers, success thresholds) for an agent | Read, WebSearch | inherit |

## Cross-cutting principles

- **No runtime, just spec.** Per the research takeaway "patterns > frameworks." We emit Markdown/YAML; LangGraph, OpenAI Agents SDK, CrewAI etc. are the runtime.
- **Self-hosting.** The kit is built using its own scaffolds. If `scaffold-skill` can't produce `validate-agent-bundle`, neither work.
- **Format conservatism.** Every frontmatter field used in our templates must appear in `claude-code-skills-reference.md` or `claude-code-subagents-reference.md`. Anything marked `[observed, undocumented]` there is gated behind a comment.
- **Constitution compatibility.** We extend, not replace, `memory/constitution.md` and the existing `templates/spec-template.md` shape so that an agents-spec-kit project is also a valid spec-kit project.

## Open questions for v1

- Should the slash commands be `/speckit.agent.specify` (namespaced under speckit) or `/speckit.agent` (single command with subcommand args)? Current plan: namespaced; matches existing `/speckit.specify` style.
- Skill validator: regex-based vs. ask the LLM. Current plan: validator skill prompts Claude to check, since structural rules are nuanced; future v2 could add a Python validator script in `scripts/`.
- Where does `eval-spec.md` live — under `specs/NNN-feature/` or a separate `evals/` tree? Current plan: under `specs/NNN-feature/` to keep one feature = one folder.

## What the next milestone produces

After this design lands:
1. Build the 4 new slash command files in `templates/commands/`.
2. Build the agent spec/plan templates in `templates/`.
3. Build the 5 skills under `.claude/skills/`.
4. Build the 5 subagents under `.claude/agents/`.
5. Update `AGENTS.md` and `README.md` with an "Agent-building extension" section.
6. Final commit and push.
