---
name: validate-agent-bundle
description: Validate a directory of Claude Code skills and subagents for format and quality issues. Use when the user asks to "lint skills", "validate agents", "check the .claude/ folder", "review my agent bundle", or before scaffolding new files into a bundle.
---

# Validate Agent Bundle

## When to use

Use this skill when the user wants to **check their `.claude/skills/` and `.claude/agents/` files for format problems and authoring issues**. Concrete trigger phrases: "validate my skills", "lint the agent bundle", "check the .claude folder", "review my SKILL.md files", "is this a good description".

Skip this skill when:
- The user wants substantive review of what the agent does, not the format. Use `agent-reviewer` (subagent) instead.
- The user is asking to write new files. Use `scaffold-skill` or `scaffold-subagent`.

## How to use

1. **Get the target directory.** Default: `.claude/` at the project root. Could also be a single skill folder or a single subagent file.

2. **Enumerate files:**
   - `find <dir>/skills -name SKILL.md` for skills.
   - `find <dir>/agents -name '*.md'` for subagents.
   - Note: `.claude/agents/` files are flat (one .md per subagent). `.claude/skills/<name>/` are folders.

3. **For each SKILL.md, check:**

   **Frontmatter (block delimited by `---`):**
   - [ ] Has frontmatter at the top of the file.
   - [ ] `name` is present, lowercase + hyphens only, no spaces.
   - [ ] `name` matches the parent folder name.
   - [ ] `description` is present and non-empty.
   - [ ] `description` is concrete: contains an action verb, a trigger condition, and at least one keyword. Not "Helps with X".
   - [ ] `description` is under ~250 chars (long descriptions don't help triggering).
   - [ ] If `allowed-tools` is present, it lists explicit tool names (not `*`).
   - [ ] No undocumented frontmatter fields (verify against `research/claude-code-skills-reference.md` §3 — only treat fields not marked `[observed, undocumented]` as safe).

   **Body:**
   - [ ] Has a "When to use" section with concrete trigger phrases.
   - [ ] Has a "How to use" section with numbered steps.
   - [ ] Has at least one Example.
   - [ ] Has at least one Anti-pattern.
   - [ ] Body is under ~500 lines (move detail to `references/*.md` if longer).

4. **For each subagent .md, check:**

   **Frontmatter:**
   - [ ] Has frontmatter.
   - [ ] `name` present, lowercase + hyphens.
   - [ ] `name` matches the file basename.
   - [ ] `name` does NOT collide with built-ins: `Explore`, `Plan`, `general-purpose`, `claude-code-guide`, `statusline-setup`.
   - [ ] `description` present and starts with a dispatch pattern: "Use this agent when...", "[Role]. Use proactively...", "[Domain] specialist...".
   - [ ] `tools` is either omitted (inherit), `*`, or an explicit list of real Claude Code tool names.
   - [ ] `model` is either omitted, `inherit`, `sonnet`, `opus`, `haiku`, or a full model ID.
   - [ ] No undocumented frontmatter fields (verify against `research/claude-code-subagents-reference.md` §3).

   **Body:**
   - [ ] Body is a one-shot brief, not a chat-style system prompt.
   - [ ] Includes "What you do NOT do" or equivalent anti-patterns section.
   - [ ] Says what the subagent returns to the parent.

5. **Cross-cutting checks:**
   - [ ] No skill name collides with a subagent name (avoids ambiguity in user's `/` menu and Claude's dispatcher).
   - [ ] No two skills share a name. No two subagents share a name.

6. **Produce a findings report:**

   ```markdown
   # Agent Bundle Validation: <path>

   **Skills inspected**: N    **Subagents inspected**: M

   ## Errors (must fix)
   - [path:line] [issue]

   ## Warnings (should fix)
   - [path:line] [issue]

   ## Info
   - [path:line] [observation]
   ```

   Severity rules:
   - **Error**: missing required frontmatter, name collision, undocumented frontmatter field, name format violation.
   - **Warning**: weak description, missing body sections, body over 500 lines, tool list too broad.
   - **Info**: minor stylistic notes.

7. **Suggest fixes inline.** For each error, propose the specific edit that resolves it. Don't just report — make the next step obvious.

## Examples

### Example 1 — clean bundle
**User says:** "Validate `.claude/`."
**You find:** 5 skills, 3 subagents, all conformant. Report shows 0 errors, 1 warning ("description for `git-helper` is generic — recommend rewriting"), 0 info.
**You output:** the report, then a one-line summary: "Bundle clean except for one warning. Fix suggestion: see report."

### Example 2 — collision
**User says:** "Check my subagents."
**You find:** `.claude/agents/explore.md` exists. That collides with the built-in `Explore` subagent.
**You report:** Error severity. Suggest renaming to e.g. `repo-explore.md` and updating the `name` field to match.

## Anti-patterns

- **Don't run autofix.** This skill validates and reports. If the user wants fixes applied, they can ask, or the scaffold/author skills can rewrite.
- **Don't be permissive about names.** A skill named `Git Helper` (with capitals/space) won't be discovered correctly. Flag it as an error, not a warning.
- **Don't grade content quality.** This skill checks format and structure. Substantive review (does this skill actually accomplish its purpose?) belongs to the `agent-reviewer` subagent.

## References

- `research/claude-code-skills-reference.md` — frontmatter authority for skills.
- `research/claude-code-subagents-reference.md` — frontmatter authority for subagents.
