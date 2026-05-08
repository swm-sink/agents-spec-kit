# Claude Code Skills: Comprehensive Reference

**Last updated:** May 8, 2026  
**Status:** Based on official Claude Code documentation  
**Sources:** [code.claude.com/docs/en/skills.md](https://code.claude.com/docs/en/skills.md), [Agent Skills Standard](https://agentskills.io/specification)

---

## 1. What is a Skill?

A **skill** is a reusable set of instructions that extends Claude's capabilities. Unlike CLAUDE.md (persistent project memory), skills load only when invoked, keeping context lean. Unlike slash commands (fixed logic), skills are prompt-based—Claude orchestrates them using its tools.

### Definition

From the official docs: *"Skills extend what Claude can do. Create a `SKILL.md` file with instructions, and Claude adds it to its toolkit. Claude uses skills when relevant, or you can invoke one directly with `/skill-name`."* (source: [code.claude.com/docs/en/skills.md](https://code.claude.com/docs/en/skills.md))

### How Claude invokes skills: Progressive disclosure model

Skills follow a **progressive disclosure** pattern:
1. **Skill description** (frontmatter + `when_to_use`) is **always in context** so Claude knows what's available
2. **Full skill body** loads **only when invoked** (automatically or manually)
3. Once loaded, skill content **stays in context** for the rest of the session (counting against token budget)

This means small descriptions cost almost nothing, but long skill bodies accrue token costs on every turn after invocation.

### Skills vs. related concepts

| Concept | Lives where | Who invokes | When loaded | Context cost |
|---------|-----------|-----------|----------|------------|
| **Skill** | `.claude/skills/<name>/SKILL.md` | You (`/name`) or Claude (auto) | On invocation | Full body stays loaded |
| **Slash command** | Built-in or `.claude/commands/` | You (`/name`) | On demand | Executes fixed logic |
| **Subagent** | `.claude/agents/<name>/` | Claude (auto) | When delegated | Isolated context, summary returned |
| **CLAUDE.md** | `.claude/CLAUDE.md` | Always | Always loaded | Always in context (set once, reuse) |

Skills are best when you repeatedly paste the same instructions, checklist, or procedure. Unlike CLAUDE.md, a skill's content costs nothing until used, so reference material and long docs belong in skill supporting files.

---

## 2. File layout

### Where skills live and scope

Skills are discovered from these locations (in override order):

| Location | Path | Scope | Notes |
|----------|------|-------|-------|
| **Enterprise** | Managed settings | All org users | Admin-deployed via managed settings |
| **Personal** | `~/.claude/skills/<skill-name>/SKILL.md` | All projects | Available everywhere |
| **Project** | `.claude/skills/<skill-name>/SKILL.md` | This project only | Committed to version control |
| **Plugin** | `<plugin>/skills/<skill-name>/SKILL.md` | Where plugin enabled | Namespaced as `plugin-name:skill-name` |

When skills share names, enterprise overrides personal, personal overrides project. Plugin skills cannot conflict (namespace-protected).

### Directory structure

Each skill is a directory with `SKILL.md` as the required entrypoint:

```
my-skill/
├── SKILL.md                 # Main instructions (required)
├── template.md              # Template for Claude to fill in (optional)
├── reference.md             # Detailed reference (loaded on demand, optional)
├── examples/
│   └── sample.md            # Example outputs showing expected format
└── scripts/
    ├── validate.sh          # Script Claude can execute
    └── helper.py            # Supporting script in any language
```

**Best practice:** Keep `SKILL.md` under 500 lines; move detailed reference material to separate files.

### Naming rules

- **Directory name** becomes the skill command: `/skill-name` invokes `~/.claude/skills/skill-name/SKILL.md`
- **Allowed characters:** Lowercase letters, numbers, hyphens only; max 64 characters
- **Legacy support:** Files in `.claude/commands/` still work (treated as skills with same rules)
- **Plugin namespacing:** Plugin skills automatically namespaced as `plugin-name:skill-name` (no conflict risk)
- **Case sensitivity:** Skill names are case-insensitive for invocation (`/MySkill` = `/myskill`)

### Live change detection

Claude Code watches skill directories for changes. Edits to `~/.claude/skills/`, `.claude/skills/`, or nested `.claude/skills/` (from `--add-dir`) take effect **within the current session** without restarting. Creating a new top-level skills directory requires a session restart so the watcher can initialize.

---

## 3. SKILL.md frontmatter

Every skill file starts with YAML frontmatter between `---` delimiters, followed by markdown content. All frontmatter fields are **optional** (no required fields).

### Complete frontmatter reference

| Field | Type | Default | Description | Max length |
|-------|------|---------|-------------|-----------|
| `name` | string | Directory name | Display name. Lowercase, numbers, hyphens; max 64 chars. | 64 |
| `description` | string | First paragraph of body | What the skill does and when to use it. Claude uses this to decide when to apply the skill. | 1,536* |
| `when_to_use` | string | (none) | Additional context: trigger phrases, example requests. Appended to `description`; counts toward 1,536 char cap. | *shared |
| `argument-hint` | string | (none) | Hint during autocomplete. Example: `[issue-number]` or `[filename] [format]`. | (none) |
| `arguments` | array or string | (none) | Named positional arguments for `$name` substitution. Space-separated string or YAML list. Names map to positions in order. | (none) |
| `disable-model-invocation` | boolean | `false` | If `true`, only you can invoke (manual `/name`). Claude cannot use automatically. | (none) |
| `user-invocable` | boolean | `true` | If `false`, hides from `/` menu. Only Claude can invoke (background knowledge). | (none) |
| `allowed-tools` | array or string | (none) | Tools Claude can use without permission when skill is active. Space-separated string or YAML list. Example: `Read Grep Bash(git *)` | (none) |
| `model` | string | (session model) | Model to use when skill is active. Options: any valid model or `inherit` to keep session model. Override applies for one turn only. | (none) |
| `effort` | string | (session effort) | Effort level: `low`, `medium`, `high`, `xhigh`, `max`. Overrides session setting for skill duration. | (none) |
| `context` | string | (none) | Set to `fork` to run in isolated subagent context. Requires a task prompt in body. | (none) |
| `agent` | string | `general-purpose` | Which subagent type to use when `context: fork` is set. Options: `Explore`, `Plan`, `general-purpose`, or custom agent name. | (none) |
| `hooks` | object | (none) | Hooks scoped to this skill's lifecycle. See [Hooks in skills](/en/hooks#hooks-in-skills-and-agents). | (none) |
| `paths` | array or string | (none) | Glob patterns limiting when skill auto-activates. Example: `["src/**/*.ts", "*.json"]`. Only applies to auto-invocation, not manual `/name`. | (none) |
| `shell` | string | `bash` | Shell for `` !`command` `` and ` ```! ` blocks: `bash` or `powershell`. | (none) |

*`description` + `when_to_use` combined are capped at **1,536 characters** in the skill listing to reduce context usage. The budget scales dynamically (1% of context window, fallback 8,000 chars). Adjust `SLASH_COMMAND_TOOL_CHAR_BUDGET` env var to change the limit.

### Example frontmatter

```yaml
---
name: summarize-changes
description: Summarizes uncommitted changes and flags anything risky. Use when the user asks what changed, wants a commit message, or asks to review their diff.
disable-model-invocation: false
allowed-tools: Bash(git *)
effort: high
---
```

---

## 4. Body conventions

### How description triggers invocation

The `description` field (and optional `when_to_use` field) tell Claude **when to load the skill**. Claude matches keywords from these fields against your natural language requests. A well-written description makes the skill reliable; vague descriptions mean Claude might never use it.

**Progressive disclosure:** Claude sees the description always, but only loads the full body when the skill is invoked. This means:
- Put **key use cases first** in the description (truncation happens mid-sentence if budget is tight)
- Reference detailed docs in supporting files (loaded on demand)
- Keep the body concise—every line is a recurring token cost once loaded

### Recommended structure

Effective skill bodies follow this pattern:

```markdown
## [Task title]

[One-line summary of what the skill does]

## When to use

[Describe the scenario where Claude should invoke this automatically]

## Instructions

1. [Step 1]
2. [Step 2]
...

## Example

[Show expected input/output]

## Anti-patterns

[Warn against common misuses]
```

**Best practices:**
- **State what to do** rather than narrating how or why
- **Use the same conciseness test** as CLAUDE.md: if you wouldn't repeat it every turn, move it to a supporting file
- **Reference files:** Include markdown links to supporting files so Claude knows when to load them (`See [reference.md](reference.md) for complete API details`)
- **Keep `SKILL.md` under 500 lines** and move detailed material to `reference.md`, `examples.md`, etc.

### Dynamic context injection

Use `` !`<command>` `` syntax to inject live data into the skill before Claude sees it. The command runs **immediately** (not by Claude), and the output replaces the placeholder:

```markdown
## Current state
!`git diff HEAD`

## Instructions
Summarize the changes above...
```

For multi-line commands, use ` ```! ` fence:

````markdown
## Environment
```!
node --version
npm --version
git status
```
````

This is **preprocessing** (runs before Claude), not something Claude executes. Claude only sees the final rendered content with actual data.

Can be disabled in settings with `"disableSkillShellExecution": true` (most useful for managed settings).

### String substitutions

Skills support dynamic placeholders that expand when the skill runs:

| Variable | Description | Example |
|----------|-------------|---------|
| `$ARGUMENTS` | All arguments passed when invoking skill | `/my-skill foo bar` → `$ARGUMENTS` = `foo bar` |
| `$ARGUMENTS[N]` | Specific argument by 0-based index | `$ARGUMENTS[0]` = first arg; `$ARGUMENTS[1]` = second |
| `$N` | Shorthand for `$ARGUMENTS[N]` | `$0` = first; `$1` = second |
| `$name` | Named argument from `arguments` list | With `arguments: [issue, branch]`, `$issue` = 1st arg |
| `${CLAUDE_SESSION_ID}` | Current session ID | Useful for logging/correlation |
| `${CLAUDE_EFFORT}` | Current effort level | `low`, `medium`, `high`, `xhigh`, `max` |
| `${CLAUDE_SKILL_DIR}` | Directory containing `SKILL.md` | Works from any cwd; useful for bundled scripts |

Multi-word arguments need shell-style quoting: `/my-skill "hello world"` passes `"hello world"` as a single argument.

---

## 5. Tool & permission scoping

### How `allowed-tools` works

The `allowed-tools` field in a skill's frontmatter pre-approves specific tools. While the skill is active, Claude can use those tools **without prompting** you for approval. This is a **convenience**, not a restriction: all other tools remain callable, and your permission settings still govern approval for tools not listed.

```yaml
---
name: commit
description: Stage and commit the current changes
allowed-tools: Bash(git add *) Bash(git commit *) Bash(git status *)
---
```

**Syntax:**
- Space-separated string or YAML array
- Tool name alone: `Read` (all uses)
- Tool with patterns: `Bash(git *)` (git commands only)

For project-level skills (`.claude/skills/`), `allowed-tools` takes effect after you accept the workspace trust dialog. Review project skills before trusting a repo.

### Interaction with project permissions

Skills inherit permission rules from `.claude/settings.json`. If your settings have a **deny rule** for a tool, that rule applies even if the skill tries to grant permission. To block a skill from using certain tools, add deny rules in permissions rather than modifying the skill.

**Three ways to control skill access:**

1. **Disable all skills** via deny rule:
   ```json
   "permissions": { "deny": ["Skill"] }
   ```

2. **Allow or deny specific skills**:
   ```json
   "permissions": {
     "allow": ["Skill(commit)", "Skill(review-pr *)"],
     "deny": ["Skill(deploy *)"]
   }
   ```

3. **Hide individual skills** from Claude (but not user menu):
   ```yaml
   # In SKILL.md:
   disable-model-invocation: true  # Claude can't use; you can invoke manually
   user-invocable: false           # You can't use; only Claude sees/invokes
   ```

---

## 6. Distribution

### How skills ship in plugins

Skills in plugins are **automatically namespaced** to avoid conflicts:

```
my-plugin/
├── skills/
│   ├── my-skill/
│   │   └── SKILL.md
│   └── another-skill/
│       └── SKILL.md
└── plugin.json
```

When the plugin is enabled, these become `/my-plugin:my-skill` and `/my-plugin:another-skill`. The namespace prevents collisions with project or personal skills of the same name.

### How users install skills

**Manual:**
1. Copy skill directory to `~/.claude/skills/<skill-name>/` (personal) or `.claude/skills/<skill-name>/` (project)
2. Restart Claude Code session (for personal skills) or just use it (for project skills, which are auto-discovered)

**Via plugin:**
1. Install plugin: `/plugin add <plugin-name>`
2. Skills are auto-discovered and namespaced

**Via managed settings (enterprise):**
Admin deploys skills organization-wide through managed settings files

### Agent Skills open standard

Claude Code skills follow the **Agent Skills open standard**, which is supported by 30+ tools (Codex, Cursor, Gemini CLI, JetBrains Junie, AWS Kiro, etc.). This means skills are **portable across AI coding tools**—write once, use everywhere.

The standard specifies the SKILL.md format (YAML frontmatter + markdown), directory structure, and required/optional fields. Claude Code extends the standard with additional features (invocation control, subagent execution, dynamic context injection).

Source: [agentskills.io](https://agentskills.io) and [What Is the Agent Skills Open Standard?](https://www.agensi.io/learn/agent-skills-open-standard)

---

## 7. Authoring best practices

### Description-writing patterns that trigger reliably

**Good descriptions** include:
- **Action words:** "Summarize," "Deploy," "Validate," "Migrate," "Review"
- **Trigger scenarios:** "Use when the user asks...", "Automatically load when working with...", "Useful for checking..."
- **Key use cases first** (descriptions are truncated at 1,536 chars)
- **Concrete examples:** Mention file types, domains, or tools (`"Use when working with GitHub PRs"`)

**Poor descriptions fail to trigger because:**
- Too vague: `"Helper tool"` (Claude doesn't know when to use it)
- Buried keywords: Key use case is the 5th sentence (description truncated before then)
- Too technical: `"YAML parser optimizer"` (doesn't match natural language)

### TRIGGER / SKIP heuristics [observed, undocumented]

Skilled authors often include explicit hints in the description:

```yaml
description: |
  Summarizes uncommitted changes and flags risks. 
  
  TRIGGER WHEN: User asks "What did I change?", "Commit message?", "Review my diff"
  SKIP WHEN: No uncommitted changes, user explicitly asks for no summary
```

This helps Claude understand edge cases, though Claude's matching is flexible and natural language (not rule-based).

### Testing if a skill works

**Verify discovery:**
```bash
/skills  # Opens menu; your skill should appear
```

**Test auto-invocation:** Ask Claude something matching the description:
- Skill: `description: "Summarizes changes and flags risks"`
- Test: Ask "What changed?"
- Expected: Claude invokes the skill

**Test manual invocation:**
```
/skill-name
/skill-name with arguments
```

**Verify skill is loaded:**
- After first invocation, check if Claude mentions the skill or its content
- If not, description might not match well enough to trigger auto-invoke

### Common failure modes

| Problem | Cause | Fix |
|---------|-------|-----|
| Skill never invokes | Description too vague or keywords buried | Rewrite description; put use case first |
| Triggers too often | Description too broad | Make description more specific; add `disable-model-invocation: true` |
| Arguments not working | Syntax error in `$0`, `$1`, or `$ARGUMENTS` | Verify syntax; test with simple skill |
| Long body slows session | Token cost after loading | Move detailed docs to supporting files; reference from `SKILL.md` |
| Descriptions truncated in menu | Too many skills or long descriptions | Trim descriptions; use `skillOverrides` to set low-priority skills to `"name-only"` |
| Skill stops working after compaction | Auto-compaction drops old skills | Re-invoke skill after compaction if needed |

---

## 8. Annotated example: `/claude-api` bundled skill

The `/claude-api` skill is a bundled skill that ships with Claude Code. It's invoked automatically when code imports `anthropic` or `@anthropic-ai/sdk`, and can be manually invoked with `/claude-api` or `/claude-api migrate` or `/claude-api managed-agents-onboard`.

**Why this example is instructive:**
- Bundled skills are prompt-based (not fixed logic)
- Demonstrates argument handling (`migrate`, `managed-agents-onboard`)
- Shows selective auto-invocation (triggers on import, not all the time)
- Illustrates tool scoping (`allowed-tools`)

**Likely structure** (reconstructed from docs; bundled skills may differ internally):

```yaml
---
name: claude-api
description: |
  Build, debug, and optimize Claude API and Anthropic SDK apps. 
  Covers tool use, streaming, batches, structured outputs, and common pitfalls. 
  Load Claude API reference material for your project's language (Python, TypeScript, Java, Go, Ruby, C#, PHP, cURL) 
  and Managed Agents reference.
when_to_use: |
  TRIGGER WHEN: Code imports `anthropic` or `@anthropic-ai/sdk`, or user asks about Claude API, SDK, or Managed Agents
  Use `/claude-api migrate [old-model] [new-model]` to upgrade code between model versions
  Use `/claude-api managed-agents-onboard` for interactive walkthrough to create a Managed Agent
allowed-tools: Read Grep Bash(*)
effort: high
arguments:
  - command
---

# Claude API Skill

Load reference material for the Claude API and Anthropic SDK based on your project language.

## Available commands

- **Default (no args):** Load language-specific API reference
- **`migrate [old-model] [new-model]`:** Upgrade existing Claude API code to a newer model version
- **`managed-agents-onboard`:** Interactive walkthrough for creating a new Managed Agent

## Instructions

### For default invocation:
1. Detect project language from imports and file extensions
2. Load appropriate SDK reference for Python, TypeScript, Java, Go, Ruby, C#, PHP, or cURL
3. Provide guidance on tool use, streaming, batches, structured outputs
4. Flag common pitfalls for this language

### For `migrate` command:
Ask the user which files to scan and which model to target. Then update:
- Model IDs in code
- Thinking configuration (if changed between versions)
- Tool use syntax (if updated)
- Vision or batch API parameters (if deprecated)

### For `managed-agents-onboard` command:
Guide the user through interactive steps:
1. Define the agent's role and capabilities
2. Configure tools and permissions
3. Set up memory/context
4. Deploy and test

## See also

- [Claude API Documentation](https://platform.claude.com/docs)
- [Agent Skills – Anthropic](https://agentskills.io)
```

**Annotation notes:**
- `description` + `when_to_use` together form the auto-invocation trigger (mentions imports)
- `allowed-tools: Read Grep Bash(*)` grants permissions without prompting (needed for analysis)
- `arguments` defines named placeholders (`$command`) for `/claude-api migrate ...` usage
- High effort level (`effort: high`) signals that this is a complex, reasoning-heavy skill
- Body uses markdown headers to organize commands and scenarios
- Supporting docs (API reference) would be external (implied, not bundled in this example)

Source: [code.claude.com/docs/en/commands.md](https://code.claude.com/docs/en/commands.md)

---

## 9. Open questions & gotchas

### Undocumented behaviors [observed]

1. **Skill content stays in context indefinitely** — Once a skill loads, its body remains in the conversation context forever (within a session), not just for one turn. This is a token-cost design choice, but not explicitly documented in how-to guides.

2. **`disable-model-invocation: true` hides description** — When you set this flag, Claude doesn't see the skill's description in its context at all, only you can invoke it manually. This differs from `user-invocable: false`, where Claude sees the description but can't invoke.

3. **Permission syntax for skills uses tool-like patterns** — You can use patterns like `Skill(name *)` for prefix matching, similar to `Bash(git *)`, but this is only documented in the permissions section, not the skills section.

4. **Auto-compaction may drop old skills** — After context compaction, skills are re-attached with a shared 25,000-token budget across all skills; older/lower-priority skills can be dropped entirely if you've invoked many. Re-invoke manually to restore full content.

### Ambiguities in public docs

1. **Frontmatter field case sensitivity** — Not explicitly stated whether `name`, `description`, etc. are case-insensitive. Tested behavior: lowercase is expected (YAML convention).

2. **When do nested `.claude/skills/` directories load?** — The docs mention automatic discovery from subdirectories (monorepos), but edge cases (symlinks, relative paths) are not detailed.

3. **Maximum skill body size** — No documented limit on skill file size; guidance is pragmatic (keep under 500 lines, move rest to supporting files).

4. **How "paths" glob patterns interact with subagents** — The `paths` field restricts auto-invocation, but unclear if it affects subagent preloading. Likely no effect, but untested edge case.

### Common gotchas

**Gotcha 1: Description truncation**
Your skill's description can be truncated mid-sentence if budget is tight. If a skill never triggers, the keyword might be in the truncated part. Fix: Put key use case first, be concise.

**Gotcha 2: `allowed-tools` grants, doesn't restrict**
Setting `allowed-tools: Bash(git *)` doesn't prevent Claude from using other bash commands; it only grants permission for git commands without prompting. Your deny rules in settings override it.

**Gotcha 3: Token cost of loaded skills**
Once a skill loads, every line counts on every turn. If a skill never gets invoked again after the first time, the token cost is pure waste. Move reference material to supporting files.

**Gotcha 4: Shell injection commands run before Claude**
`` !`git diff` `` runs on your machine immediately, not by Claude. If the command fails or has side effects, that happens synchronously. No error handling by Claude—use && or || in the command.

**Gotcha 5: Subagents don't inherit project skills by default**
When you use `context: fork` or explicitly invoke a subagent, the subagent doesn't automatically know about your project's skills. You can preload them via `skills` field in the subagent config.

---

## 10. Quick reference: Common skill patterns

### Pattern: Task skill (manual trigger)

```yaml
---
name: deploy
description: Deploy to production
disable-model-invocation: true  # Only you invoke manually
allowed-tools: Bash(*)
---

Deploy steps:
1. Run tests: `npm test`
2. Build: `npm run build`
3. Deploy: `npm run deploy`
```

### Pattern: Reference skill (Claude auto-invokes)

```yaml
---
name: api-conventions
description: API design patterns and conventions for this codebase
user-invocable: false  # Only Claude sees this; you don't invoke manually
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

### Pattern: Skill with arguments

```yaml
---
name: fix-issue
description: Fix a GitHub issue by number
arguments: [issue-number]
allowed-tools: Bash(gh *)
---

Fix GitHub issue $0:

1. Read issue: `gh issue view $0`
2. Create branch: `git checkout -b fix/$0`
3. Implement fix
4. Push and create PR
```

### Pattern: Skill with forked subagent

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
allowed-tools: Read Grep
---

Research $ARGUMENTS:

1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with file references
```

---

## References

- **Official documentation:** [code.claude.com/docs/en/skills.md](https://code.claude.com/docs/en/skills.md)
- **Agent Skills standard:** [agentskills.io/specification](https://agentskills.io/specification)
- **Agent Skills adoption:** [What Is the Agent Skills Open Standard?](https://www.agensi.io/learn/agent-skills-open-standard)
- **Subagents guide:** [code.claude.com/docs/en/sub-agents.md](https://code.claude.com/docs/en/sub-agents.md)
- **Hooks guide:** [code.claude.com/docs/en/hooks-guide.md](https://code.claude.com/docs/en/hooks-guide.md)
- **Plugins guide:** [code.claude.com/docs/en/plugins.md](https://code.claude.com/docs/en/plugins.md)
- **Best practices:** [code.claude.com/docs/en/best-practices.md](https://code.claude.com/docs/en/best-practices.md)
