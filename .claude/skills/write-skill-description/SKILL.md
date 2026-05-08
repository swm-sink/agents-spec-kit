---
name: write-skill-description
description: Rewrite a SKILL.md or subagent description so it triggers reliably. Use when the user asks "why isn't my skill firing", "is this description good", "help me write a skill description", "improve this trigger", or paraphrases of those.
---

# Write Skill Description

## When to use

Use this skill when the user has a SKILL.md or subagent description that **isn't triggering reliably** or **hasn't been written yet** and they want help phrasing it. Concrete trigger phrases: "why isn't my skill firing", "this skill never gets picked", "improve this description", "is this trigger good", "help me write the description".

Skip this skill when:
- The user wants to write the *body* of a skill, not the description. Use `scaffold-skill`.
- The skill is firing too aggressively and they want to suppress it. The pattern there is different — narrow the description and add explicit "Skip when..." rules.

## How to use

1. **Get the artifacts.** Ask for: (a) the current description (or "none yet"), (b) the skill or subagent's purpose in plain English, (c) example user messages that should trigger it, (d) example messages that should NOT trigger it.

2. **Diagnose** if a description was provided. Apply this checklist:
   - [ ] Does it lead with an action verb? ("Reviews", "Scaffolds", "Validates", "Translates")
   - [ ] Does it state a trigger condition? ("Use when...", "Use proactively after...")
   - [ ] Does it contain at least 2 concrete keywords from the example trigger messages?
   - [ ] Is it under ~250 chars?
   - [ ] Is it free of marketing words ("powerful", "world-class", "revolutionary")?
   - [ ] Does it avoid being generic ("helps with X", "general purpose")?

   For each failure, note the symptom and the rewrite.

3. **Apply one of these patterns** based on the skill's invocation style:

   **A. Action skills** (do something on user request):
   `"<Verb> <object>. Use when the user asks to <concrete phrase 1>, <phrase 2>, or <phrase 3>."`
   Example: `"Scaffold a new Claude Code skill. Use when the user asks to 'create a skill', 'scaffold a skill', or 'add a SKILL.md'."`

   **B. Reflexive skills** (fire automatically after a triggering event):
   `"<Domain> <role>. Use proactively after <event> or when the user says <phrase>."`
   Example: `"Pre-push checklist. Use proactively before 'git push' or when the user says 'ready to ship'."`

   **C. Reference skills** (provide knowledge when a topic comes up):
   `"<Topic> reference. Loads <what kind of info>. Use when the conversation involves <keywords>."`
   Example: `"Internal error code reference. Loads the error-code → meaning table. Use when the conversation involves error codes ENXX, internal errors, or 'what does error N mean'."`

   **D. Subagent dispatch descriptions** (parent reads to delegate):
   `"Use this agent when <condition>. <Role> for <scope>."`
   Example: `"Use this agent when reviewing dependencies for known vulnerabilities. Scans package.json, requirements.txt, go.mod, Cargo.toml."`

4. **Add explicit "skip when" guidance only if** the user reports false-positive triggering. Don't pad short descriptions with skip rules — they cost tokens on every turn.

5. **Test mentally.** For each example trigger message, ask: "If I were Claude reading this description plus that user message, would I match?" For each non-trigger, ask: "Would I avoid matching?" If you can't answer yes/no confidently, the description needs more keywords or sharper conditions.

6. **Output format:**

   ```
   ## Original
   <user's current description, or "none">

   ## Issues
   - <bullet>
   - <bullet>

   ## Rewritten
   <new description>

   ## Why this version triggers more reliably
   <2-3 bullets explaining the rewrite>
   ```

## Examples

### Example 1 — generic description
**User pastes:** `description: Helps with code review.`
**You diagnose:** No verb-first action, no trigger phrases, no keywords, generic. Won't fire reliably.
**You rewrite:** `description: "Reviews staged git changes for security and style issues. Use after 'git add', when the user asks for a code review, or when they say 'how's this look'."`

### Example 2 — over-broad
**User says:** "My skill triggers way too often."
**You diagnose:** Description probably matches everything. Ask for the current text. If it's something like "Helps with development tasks", that's the problem — it's not the *fire-too-often* fix that's needed, it's *fire-correctly*.
**You rewrite:** apply pattern C and add a `## Skip when` line in the body (not the description) that explicitly excludes the false-positive cases.

### Example 3 — subagent description
**User pastes:** `description: A useful agent for code stuff.`
**You diagnose:** Parent dispatcher will never reliably pick this. Apply pattern D.
**You rewrite:** `description: "Use this agent when reviewing TypeScript code for type-safety issues, missing null checks, or inconsistent error handling. Returns a findings list with file:line references."`

## Anti-patterns

- **Don't pad with adjectives.** "Powerful, intelligent code reviewer" doesn't trigger — "Reviews staged changes for X, Y, Z" does.
- **Don't write a paragraph.** The description budget is tight. If you need more, the body is for that.
- **Don't promise unsupported behavior.** "Auto-fixes all issues" when the skill only *reports* them is a description bug that lives forever.
- **Don't optimize for one example.** A description that nails one phrase but misses related variations is brittle. Use 2–4 keywords spanning likely phrasings.

## References

- `research/claude-code-skills-reference.md` §authoring — description conventions for skills.
- `research/claude-code-subagents-reference.md` §4 — dispatch patterns for subagent descriptions.
