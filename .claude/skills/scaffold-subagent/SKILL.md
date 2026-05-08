---
name: scaffold-subagent
description: Scaffold a new Claude Code subagent — produces a single .md file under .claude/agents/ with valid frontmatter and a system prompt. Use when the user asks to "create a subagent", "add an agent", "scaffold a subagent", or describes a task they want delegated to an isolated context.
---

# Scaffold Subagent

## When to use

Use this skill when the user wants to **create a new Claude Code subagent**. Concrete trigger phrases: "scaffold a subagent", "create an agent that X", "I want a code-reviewer agent", "add a subagent for Y", "make a researcher agent".

Skip this skill when:
- The user wants a skill, not a subagent. Use `scaffold-skill`. Skill = runs in parent context. Subagent = isolated context.
- The user is asking how subagents work conceptually. Just answer.
- The user wants to invoke an existing subagent. They want the `Agent` tool, not this skill.

## How to use

1. **Get the brief.** You need:
   - Subagent name (lowercase-hyphens).
   - One-sentence purpose.
   - Target project root.
   - Whether it should "use proactively" (fire without being asked) or only on direct request.
   - Tools it needs to do its job (read-only? edit files? run shell?).
   - Project-level (`./.claude/agents/`) or user-level (`~/.claude/agents/`).

2. **Read the templates and reference docs:**
   - `templates/subagent-template.md` for body shape.
   - `research/claude-code-subagents-reference.md` for valid frontmatter (only documented fields — `name`, `description`, `tools`, `model`).

3. **Write the description carefully.** This is what the **parent** Claude reads to decide whether to delegate. Apply these patterns from the reference (§4):
   - **Pattern 1:** "Use this agent when [condition]."
   - **Pattern 2:** "[Role]. Use proactively after [trigger]." (for reflexive tasks)
   - **Pattern 3:** "[Domain] specialist. Specializes in [specific keywords]."
   - Anti-patterns: "General purpose agent", "My helper", "Does things". These never fire reliably.

4. **Pick `tools` minimally:**
   - Read-only research/review subagent: `Read, Grep, Glob, WebSearch, WebFetch`.
   - Code-editing subagent: add `Write, Edit`.
   - Shell access: add `Bash` — and consider scoping with `Bash(git *)` if the reference allows it for the user's Claude Code version.
   - Avoid `*` (everything) unless the subagent genuinely needs every tool.

5. **Pick `model: inherit`** unless there's a clear reason:
   - `haiku` — cheap classification, format conversion, log triage.
   - `opus` — hard reasoning, deep code review, multi-file refactoring.
   - Otherwise stick with `inherit` so the subagent matches the parent's quality bar.

6. **Write the body** following the template:
   - **Your job** — 1–3 sentences of role + scope.
   - **How you operate** — numbered steps.
   - **What you produce** — describe the response shape (the parent only sees this).
   - **What you do NOT do** — anti-patterns.
   - **Briefing the parent should give you** — what a good prompt to this subagent contains.

7. **Validate naming:** lowercase + hyphens, no collision with built-ins (`Explore`, `Plan`, `general-purpose`), no collision with existing files in `.claude/agents/`.

8. **Write the file.** Single `.md` under `.claude/agents/<name>.md`. No folder needed (subagents don't have references/scripts the way skills do).

9. **Report.** Path written, the description, the tools list, and a reminder: "The parent Claude must use the Agent tool to dispatch this subagent — it doesn't auto-trigger from user messages."

## Examples

### Example 1 — read-only review subagent
**User says:** "Scaffold a subagent called `dependency-auditor` that scans package files for known-vulnerable versions."
**You should:**
- Write `.claude/agents/dependency-auditor.md`.
- Description: `"Use this agent when reviewing dependencies for known vulnerabilities. Scans package.json, requirements.txt, go.mod, Cargo.toml. Use proactively before merging dependency-update PRs."`
- Tools: `Read, Grep, Glob, WebSearch`.
- Model: `inherit`.
- Body explains: read package files, look up CVEs via web search, return a findings list.

### Example 2 — file-mutating subagent with scoped tools
**User says:** "I want a `formatter` subagent that runs prettier on JS files."
**You should:**
- Description: `"Use this agent when the user asks to format JavaScript or TypeScript files. Runs prettier and reports changed files."`
- Tools: `Read, Bash(npx prettier *), Edit`.
- Note: `Bash(...)` scoping syntax should be verified for the project's Claude Code version. If it's not supported, fall back to `Read, Bash, Edit` and call it out.

## Anti-patterns

- **Don't grant `*` for tools.** Subagents should have the smallest tool set that works. The blast radius of a runaway subagent is bounded by its tools.
- **Don't write a chat-style system prompt.** The body is a one-shot brief, not a conversation. The subagent does its work, returns one summary, exits.
- **Don't reference the parent's session state.** The subagent has no memory of the parent's conversation. Anything it needs must arrive in its prompt.
- **Don't pin `model: opus` reflexively.** It's expensive. Pin only when you have evidence the task needs it.

## References

See `research/claude-code-subagents-reference.md` for the full subagent format reference, including all frontmatter fields, isolation modes, and parent-dispatch patterns.
