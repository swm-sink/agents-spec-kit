---
name: scaffold-skill
description: Scaffold a new Claude Code skill — produces a SKILL.md in the conventional folder layout. Use when the user asks to "create a skill", "scaffold a skill", "add a new skill", or describes a behavior they want triggered automatically.
---

# Scaffold Skill

## When to use

Use this skill when the user wants to **create a new Claude Code skill** for a project. Concrete trigger phrases: "scaffold a skill", "create a skill that does X", "I want a skill for Y", "add a SKILL.md", "make a skill".

Skip this skill when:
- The user is asking about an existing skill, not creating one. Use `validate-agent-bundle` or `agent-reviewer` instead.
- The user wants a subagent, not a skill. Use `scaffold-subagent`.
- The user is asking how skills work. Just answer them — don't scaffold.

## How to use

1. **Get the brief.** You need: skill name (lowercase-hyphens), one-paragraph purpose, target project root. If any are missing, ask one consolidated question.

2. **Decide the folder location.** Default: `<project>/.claude/skills/<name>/SKILL.md`. If the user says "user-level" or "personal", use `~/.claude/skills/<name>/SKILL.md`.

3. **Read the templates and reference docs:**
   - `templates/skill-template/SKILL.md` for the body shape.
   - `research/claude-code-skills-reference.md` for valid frontmatter (only documented fields — see §3).

4. **Write the description carefully.** This is the single most important field. The description is always in Claude's context; the body only loads when invoked. A weak description means the skill never fires. Apply these rules:
   - Lead with what the skill does (verb + object).
   - Add the trigger condition: "Use when ...".
   - Include 2–4 concrete keywords or example phrases the user might say.
   - Keep it under ~250 characters. Long descriptions don't help triggering.
   - **Anti-pattern:** "Helps with X." That's not a trigger.

5. **Write the body** following the template's five sections:
   - **When to use** — restate trigger and add explicit anti-triggers ("Skip when...").
   - **How to use** — numbered steps.
   - **Examples** — at least two realistic user messages and the expected behavior.
   - **Anti-patterns** — at least two failure modes the skill should avoid.
   - **References** — only include if the skill has additional `references/*.md` files.

6. **Decide on `allowed-tools`.** Omit unless the skill must tightly scope tool access. If you do scope, list explicit tool names (e.g., `Read, Grep, Bash(git *)`).

7. **Decide on additional files.** If the body would exceed ~500 lines, move detail into `references/<name>.md` files and link to them from the body. The skill loads those on demand.

8. **Write the file(s).** Use the Write tool. Verify the parent directory exists first (`mkdir -p` if needed).

9. **Validate.** Read your own output: does the description trigger reliably for the brief? Are all five body sections present? Run `validate-agent-bundle` if available.

10. **Report.** Print the path written and a one-line description summary.

## Examples

### Example 1 — straightforward skill
**User says:** "Scaffold a skill called `git-pre-push-checklist` that reminds Claude to run tests and lint before pushing."
**You should:**
- Confirm the name is `git-pre-push-checklist`, project root is current dir.
- Write `.claude/skills/git-pre-push-checklist/SKILL.md` with description `"Pre-push checklist for git: tests, lint, and changelog. Use before running 'git push' or when the user says 'ready to push' or 'looks good to ship'."`
- Body: When to use (trigger phrases), How to use (5 steps: stash check, tests, lint, changelog, then push), Examples, Anti-patterns ("Don't push without confirming the user has stashed WIP"), References (none).
- Report: `.claude/skills/git-pre-push-checklist/SKILL.md written.`

### Example 2 — skill that needs a reference doc
**User says:** "Make a skill for translating our internal error codes."
**You should:**
- Ask for the project root and confirm "internal error codes" — do they have a code table?
- If yes, write `SKILL.md` with a brief description and lookup steps, plus `references/error-codes.md` containing the full table. Body links to references.
- Tell the user how the progressive-disclosure model means the table only loads when needed.

## Anti-patterns

- **Don't write a 500-line SKILL.md.** Move detail to `references/`. The body counts against the token budget once invoked.
- **Don't pad descriptions with marketing.** "World-class skill that revolutionizes..." doesn't help triggering. Concrete keywords do.
- **Don't add `allowed-tools` "just in case".** Omitting it inherits sane defaults. Adding it is a constraint to maintain.
- **Don't invent frontmatter fields.** Stick to `name`, `description`, `allowed-tools`. Fields like `model`, `arguments`, `paths` are documented in some sources but inconsistently supported — leave them out unless you've verified them in the project's Claude Code version.

## References

See `research/claude-code-skills-reference.md` (in the agents-spec-kit repo root) for the full skills format reference, including the progressive-disclosure model and frontmatter authority.
