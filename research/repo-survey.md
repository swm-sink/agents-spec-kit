# Agent Ecosystem Survey (May 2026)

## Methodology

Repos were selected based on (a) GitHub star momentum and active maintenance as of May 2026, (b) direct relevance to building, orchestrating, evaluating, or distributing AI agents, and (c) having a concrete, studyable artifact (a file format, registry shape, dispatcher, or pattern) that an agents-spec-kit fork should learn from. Star counts were verified via web search snapshots from late April / early May 2026; treat them as approximate. Repos with no commits in the last 12 months were excluded.

---

## Category 1: Claude Code Skills & Subagent Collections

### 1. anthropics/skills — ~5k stars
- **What it is:** Anthropic's official public repo of Agent Skills (PDF, docx, pptx, skill-creator, doc-coauthoring, etc.).
- **Why it matters:** This is the canonical SKILL.md format. Any spec-kit for skills must be bidirectionally compatible with what ships in this repo.
- **Study:** `template/SKILL.md` and `skills/skill-creator/SKILL.md` — the YAML frontmatter (`name`, `description`) plus the `scripts/`, `references/`, `assets/` folder convention.

### 2. hesreallyhim/awesome-claude-code — ~41k stars
- **What it is:** The de facto community curated list for Claude Code skills, hooks, slash-commands, agent orchestrators, plugins, and CLAUDE.md setups.
- **Why it matters:** It is where any new spec-kit will be discovered or ignored. The taxonomy it uses (skills vs hooks vs commands vs orchestrators) is the mental model the community already accepts.
- **Study:** `README_ALTERNATIVES/README_FLAT_ALL_AZ.md` — flat A–Z entry format used as the canonical entry schema.

### 3. wshobson/agents — ~10k stars
- **What it is:** 185 specialized subagents + 16 multi-agent workflow orchestrators + 153 skills + 100 commands, packaged as 80 single-purpose Claude Code plugins.
- **Why it matters:** Best-in-class real-world example of bundling agents/skills/commands as a marketplace. The plugin-per-capability granularity is a strong design constraint to consider.
- **Study:** `.claude-plugin/marketplace.json` and `plugins/agent-teams/` directory layout.

### 4. wshobson/commands — ~2.3k stars
- **What it is:** 57 production-ready Claude Code slash commands, organized as workflows (multi-agent) and tools (single-purpose).
- **Why it matters:** Demonstrates the workflows-vs-tools split and how to invoke commands by namespace prefix (`/workflows:` vs `/tools:`).
- **Study:** `workflows/` vs `tools/` directory split and the front-matter pattern for slash commands.

### 5. VoltAgent/awesome-claude-code-subagents — ~6k stars
- **What it is:** A 100+ subagent collection bundled into installable plugin packs (voltagent-lang, voltagent-infra, etc.).
- **Why it matters:** Shows how to ship subagents as `claude plugin install <pack>` units — the "domain pack" model.
- **Study:** `CLAUDE.md` and the per-pack manifest layout, especially how tool permissions are scoped per subagent.

### 6. davepoon/claude-code-subagents-collection (buildwithclaude) — ~5k stars
- **What it is:** Hub of skills/agents/commands/hooks for Claude Code, Claude Desktop, Agent SDK, and OpenClaw, paired with a `bwc-cli` discovery tool and a hosted marketplace at buildwithclaude.com.
- **Why it matters:** Pairing a spec format with a CLI installer (bwc-cli) is the integration pattern we should clone — spec without distribution is dead on arrival.
- **Study:** `subagents/sql-expert.md` example file + bwc-cli registry/index format.

### 7. obra/superpowers — ~3k stars
- **What it is:** An "agentic skills framework & software development methodology" with subagent-driven-development, working across Claude Code, Codex CLI, Cursor, Gemini CLI, OpenCode, and Copilot CLI.
- **Why it matters:** Most aggressive multi-agent IDE-portable skill design in the wild; defines a "skill-as-reference-guide" philosophy spec-kit should reckon with.
- **Study:** `skills/writing-skills/SKILL.md` and `skills/using-superpowers/SKILL.md` — the meta-skills that teach Claude how to use other skills.

### 8. vijaythecoder/awesome-claude-agents — ~5k stars
- **What it is:** A 24-agent "orchestrated dev team" for Claude Code with explicit Tech Lead, Project Analyst, and Team Configurator roles.
- **Why it matters:** Cleanest open example of a hierarchical orchestrator pattern (one orchestrator agent that dispatches to specialists).
- **Study:** `agents/orchestrators/team-configurator.md` — the agent that bootstraps team composition based on detected stack.

### 9. anthropics/claude-plugins-official — ~1k stars
- **What it is:** Anthropic's curated directory of high-quality official Claude Code plugins.
- **Why it matters:** Defines the official plugin marketplace JSON schema that third-party kits must align with.
- **Study:** `.claude-plugin/marketplace.json` schema and how plugins declare their skills/commands/agents.

---

## Category 2: Major Agent Frameworks

### 10. langchain-ai/langgraph — ~31k stars
- **What it is:** Graph-based runtime for stateful, resilient language agents; Python and TypeScript.
- **Why it matters:** Reference implementation of agents-as-state-machines; checkpoints, interrupts, human-in-the-loop. The graph schema is widely copied.
- **Study:** Graph + node + edge YAML/Python definitions and the `interrupt()` / checkpointer interface.

### 11. langchain-ai/langchain — ~135k stars
- **What it is:** The "agent engineering platform" — broadest LLM app framework, now positioned as infrastructure layer.
- **Why it matters:** Sets community expectations for tool definition shape, message types, and provider abstraction.
- **Study:** `langchain_core/tools` schema and the chat-model + tool-calling protocol.

### 12. crewAIInc/crewAI — ~50k stars
- **What it is:** Role-playing multi-agent framework focused on collaborative agent crews.
- **Why it matters:** Popularized YAML-defined agents and tasks — directly relevant to "agent-as-spec" thinking.
- **Study:** `agents.yaml` and `tasks.yaml` declarative configuration; the role/goal/backstory triad.

### 13. microsoft/autogen — ~50k stars (maintenance mode)
- **What it is:** Programming framework for agentic AI from Microsoft Research; now superseded by microsoft/agent-framework but still widely studied.
- **Why it matters:** Conversational multi-agent patterns (GroupChat, UserProxy) influenced almost every other framework. Now a cautionary tale on framework lifecycle.
- **Study:** `GroupChatManager` and the speaker-selection logic.

### 14. microsoft/agent-framework — ~5k stars
- **What it is:** Microsoft's unified successor to AutoGen + Semantic Kernel, supporting Python and .NET, with declarative YAML agent definitions.
- **Why it matters:** First-class **declarative configuration** for agents (instructions, tools, memory, orchestration topology in version-controlled YAML) plus native MCP and A2A protocol support — most spec-kit-aligned framework on the market.
- **Study:** YAML agent declaration format and the orchestration patterns (sequential, concurrent, handoff, group chat, Magentic-One).

### 15. openai/openai-agents-python — ~26k stars
- **What it is:** Lightweight multi-agent SDK from OpenAI, the production-ready evolution of Swarm.
- **Why it matters:** Defines minimalist agent + handoff + guardrail primitives. Cleanest "small core" model.
- **Study:** `Agent`, `Handoff`, and `Runner` primitives — the canonical handoff-as-tool-call pattern.

### 16. huggingface/smolagents — ~26k stars
- **What it is:** Barebones agent library where agents "think in code" (CodeAgent).
- **Why it matters:** Pioneered the code-as-action paradigm — agents emit Python rather than JSON tool calls. Highly relevant for tool-spec design.
- **Study:** `CodeAgent` execution loop and the sandboxed code interpreter integration.

### 17. agno-agi/agno — ~30k stars
- **What it is:** High-performance, model-agnostic framework for multi-modal agents and agent platforms (formerly Phidata).
- **Why it matters:** Performance-first design (claims 5000x faster instantiation than LangGraph). Multi-modal native (text, image, audio, video).
- **Study:** `Agent` class signature and the function-calling abstraction over OpenAI/Anthropic schemas.

### 18. pydantic/pydantic-ai — ~17k stars
- **What it is:** Type-safe AI agent framework "with the FastAPI feeling" from the Pydantic team.
- **Why it matters:** Type-driven agent IO and structured output is increasingly the norm; spec-kit should bake in pydantic-style schema validation.
- **Study:** The `Agent[Deps, OutputType]` generic signature and tool registration via `@agent.tool`.

### 19. mastra-ai/mastra — ~22k stars
- **What it is:** TypeScript agent framework from the Gatsby team — workflows, agents, RAG, integrations, evals.
- **Why it matters:** Best-in-class TS story; bundles agents into existing React/Next.js apps and exposes them as MCP servers automatically.
- **Study:** `Workflow` definitions with suspend/resume primitives and the auto-generated MCP server endpoints.

### 20. letta-ai/letta — ~17k stars
- **What it is:** Platform for stateful agents with persistent memory (formerly MemGPT).
- **Why it matters:** Defines a portable agent file format (`.af`) for serializing stateful agents — directly inspires "spec-as-agent" portability.
- **Study:** `letta-ai/agent-file` `.af` format — agents as checkpointable, version-controllable files.

### 21. google/adk-python — ~12k stars
- **What it is:** Google's Agent Development Kit — code-first toolkit for building, evaluating, deploying agents (also adk-go, adk-java).
- **Why it matters:** "Designed to be written by both humans and AI" with token-budget-aware context management and built-in dev UI.
- **Study:** ADK developer Skills format and the structured context filtering / artifact lazy-loading.

### 22. strands-agents/sdk-python — ~6k stars
- **What it is:** AWS-backed open-source SDK for model-driven agents, deployable to Lambda/Fargate/EKS/Bedrock AgentCore.
- **Why it matters:** Production-deployment-first design with built-in OpenTelemetry; multi-agent patterns (Graph, Swarm, Workflow) as first-class.
- **Study:** Model-driven `Agent(model=..., tools=...)` minimal API and the Apache-2.0 governance model.

### 23. ag2ai/ag2 — ~4k stars
- **What it is:** Open-governance fork of AutoGen as "AgentOS" after Microsoft moved AutoGen to maintenance mode.
- **Why it matters:** Demonstrates community fork governance for agent infrastructure — relevant precedent for an agents-spec-kit fork.
- **Study:** Their group-chat and conversable-agent abstractions and the migration story from microsoft/autogen.

---

## Category 3: MCP-Related

### 24. modelcontextprotocol/servers — ~60k stars
- **What it is:** Reference implementations of MCP servers maintained by the MCP steering group.
- **Why it matters:** The reference shape for any MCP server an agent might author — spec-kit should make scaffolding these trivial.
- **Study:** The reference server layouts (filesystem, github, postgres) and the standard `tools/list`, `resources/list`, `prompts/list` exposure pattern.

### 25. modelcontextprotocol/python-sdk — ~16k stars
- **What it is:** Official Python SDK for MCP servers and clients.
- **Why it matters:** Defines `FastMCP` decorator-based ergonomics that have become the convention for tool authoring.
- **Study:** `FastMCP` server pattern with `@mcp.tool()`, `@mcp.resource()`, `@mcp.prompt()` decorators.

### 26. modelcontextprotocol/typescript-sdk — ~9k stars
- **What it is:** Official TypeScript SDK for MCP, runs on Node/Bun/Deno.
- **Why it matters:** Most agent runtimes (Cline, Continue, Mastra) integrate via this SDK. TS-first agents-spec-kit consumers will need it.
- **Study:** Server transport interfaces (stdio, SSE, streamable HTTP) and the schema-validation patterns.

### 27. punkpeye/awesome-mcp-servers — ~86k stars
- **What it is:** The community curated list of MCP servers (production + experimental).
- **Why it matters:** The discovery surface — spec-kit-generated servers must conform to whatever pattern this list expects (description + transport + install).
- **Study:** The README entry schema (one-line description, transport, install instructions) used to enroll new servers.

### 28. a2aproject/A2A — ~16k stars
- **What it is:** Linux Foundation–hosted Agent2Agent protocol (originally Google) for cross-runtime agent communication.
- **Why it matters:** Complementary to MCP — MCP is agent↔tools, A2A is agent↔agent. Spec-kit must address both axes.
- **Study:** The A2A capability-discovery and modality-negotiation message types.

### 29. ComposioHQ/composio — ~25k stars
- **What it is:** 1000+ pre-built toolkits with auth, search, context management, and a sandboxed workbench for agents.
- **Why it matters:** The pragmatic layer above MCP — solves the auth/oauth/rate-limit problem most spec-kit users will hit.
- **Study:** Toolkit definition format and how Composio normalizes auth across 1000+ APIs.

---

## Category 4: Agent Evaluation & Observability

### 30. UKGovernmentBEIS/inspect_ai — ~5k stars
- **What it is:** UK AI Security Institute's eval framework with 200+ pre-built evals; adopted by Anthropic, DeepMind, xAI.
- **Why it matters:** State-of-the-art for **agent** (not just LLM) evals; first-class tool-use, multi-turn, model-graded evaluation.
- **Study:** The Dataset / Solver / Scorer triad and the `@task` / `@solver` decorators.

### 31. THUDM/AgentBench — ~3k stars
- **What it is:** ICLR'24 benchmark evaluating LLMs as agents across 8 environments (SQL, KG, ALFWorld, WebShop, Aquawar, etc.).
- **Why it matters:** Standard reference for "is your agent actually competent" — spec-kit should make running it easy.
- **Study:** The function-calling task definitions and AgentRL multitask harness integration.

### 32. openai/evals — ~16k stars
- **What it is:** Evals framework + open-source registry of benchmarks from OpenAI.
- **Why it matters:** Defines the YAML eval registry format that many derivative tools (including OpenAI Dashboard) consume.
- **Study:** `evals/registry/` YAML format and the Solver pattern.

### 33. langfuse/langfuse — ~14k stars (YC W23)
- **What it is:** Open-source LLM engineering platform: observability, metrics, evals, prompt management, playground, datasets.
- **Why it matters:** Most-adopted open-source LLM engineering platform; integrates with OpenTelemetry, LangChain, OpenAI SDK, LiteLLM.
- **Study:** Trace/Generation/Span hierarchy and the prompt-versioning API.

### 34. Arize-ai/phoenix — ~9k stars
- **What it is:** Open-source AI observability and evaluation, with native support for OpenAI Agents SDK, Claude Agent SDK, LangGraph, Mastra, CrewAI, LlamaIndex, DSPy.
- **Why it matters:** Vendor-and-language agnostic OTel-based agent tracing. Notebook + cloud + container deployable.
- **Study:** OpenInference span conventions for agent traces.

### 35. AgentOps-AI/agentops — ~5k stars
- **What it is:** Python SDK for AI agent monitoring, LLM cost tracking, benchmarking; integrates with CrewAI, Agno, OpenAI Agents SDK, LangChain, AutoGen, AG2, CamelAI.
- **Why it matters:** Two-line instrumentation, designed specifically for agent (not just LLM) telemetry — session replays, multi-agent interaction graphs.
- **Study:** Their session-replay event model and the auto-instrumentation hooks across frameworks.

### 36. Helicone/helicone — ~5k stars (YC W23)
- **What it is:** Open-source LLM observability + AI gateway, one-line proxy integration; 100+ models via OpenAI-compatible endpoint.
- **Why it matters:** Proxy-based instrumentation requires zero code changes — the lowest-friction observability path for spec-kit users.
- **Study:** AI Gateway routing config and the Helicone-prefixed header convention.

### 37. traceloop/openllmetry — ~6k stars
- **What it is:** OpenTelemetry-based observability for GenAI/LLM apps; supports 20+ providers, frameworks (LangChain, LlamaIndex, CrewAI), vector DBs.
- **Why it matters:** Pure OTel approach (no proprietary SDK lock-in); the most portable observability story.
- **Study:** GenAI semantic conventions and the cross-language instrumentation packages.

---

## Category 5: Prompt Engineering / Agent Patterns

### 38. anthropics/claude-cookbooks — ~42k stars
- **What it is:** Official Anthropic notebooks/recipes including the agent-patterns subdirectory.
- **Why it matters:** The `patterns/agents/` directory codifies the canonical Anthropic patterns (orchestrator-workers, evaluator-optimizer, prompt chaining, routing, parallelization).
- **Study:** `patterns/agents/` notebooks — these are the patterns spec-kit templates should produce by name.

### 39. dair-ai/Prompt-Engineering-Guide — ~74k stars
- **What it is:** Guides, papers, lectures, notebooks for prompt engineering, context engineering, RAG, AI agents.
- **Why it matters:** The reference taxonomy of prompting techniques most engineers know — naming alignment matters for spec-kit's documentation.
- **Study:** `guides/` directory structure and the technique → use-case mapping.

### 40. neural-maze/agentic-patterns-course — ~3k stars
- **What it is:** From-scratch implementation of Andrew Ng's 4 agentic patterns (reflection, tool use, planning, multi-agent).
- **Why it matters:** Cleanest minimal reference for the canonical 4 patterns — useful as a tutorial baseline that spec-kit can scaffold to.
- **Study:** `notebooks/reflection_pattern.ipynb` and `tool_pattern/tool_agent.py` — pattern-per-file organization.

### 41. humanlayer/12-factor-agents — ~10k stars
- **What it is:** Manifesto + content repo defining 12 production engineering principles for LLM agents (own your context, unify execution state, etc.).
- **Why it matters:** Sets the discourse around "good agents are mostly software." Strongly opinionated counterweight to framework-heavy approaches; explicitly **not a framework** ("shadcn for agents").
- **Study:** `content/factor-XX-*.md` — each factor is a discrete principle file. The non-framework, customize-yourself stance is a viable spec-kit positioning option.

---

## Category 6: Coding-Agent Reference Implementations

### 42. Aider-AI/aider — ~42k stars
- **What it is:** AI pair programming in your terminal, git-native, automatic descriptive commits.
- **Why it matters:** Pioneered repo-level edit primitives (search/replace blocks, whole-file edits) and the leaderboard-driven approach to coding-agent eval.
- **Study:** The edit-format prompts (`coders/editblock_coder.py`) — these are reusable as Claude skills.

### 43. All-Hands-AI/OpenHands — ~65k stars
- **What it is:** AI-driven development platform with composable Python SDK, CLI, GUI, and REST API. Agents modify code, run commands, browse, call APIs.
- **Why it matters:** Most complete open OSS coding-agent stack; OpenHands 1.0 introduced a clean software-agent-sdk separation that's a great architectural reference.
- **Study:** `software-agent-sdk` agent loop and the Agent Computer Interface (ACI) abstractions in `openhands-aci`.

### 44. SWE-agent/SWE-agent — ~18k stars (NeurIPS 2024)
- **What it is:** Princeton/Stanford project that turns LMs into software engineers that fix GitHub issues; SoTA on SWE-Bench.
- **Why it matters:** The Mini-SWE-Agent variant achieves 65% on SWE-bench verified in 100 lines — proof that minimal, well-spec'd agents beat sprawling frameworks.
- **Study:** `mini` variant — the minimal command/observation interface for code-editing agents.

### 45. cline/cline — ~50k stars (5M+ installs)
- **What it is:** Open-source autonomous coding agent (originally Claude Dev) running as VS Code/JetBrains/Cursor sidebar with MCP support.
- **Why it matters:** Most-installed open coding agent; demonstrates the IDE-extension distribution model and how MCP gets consumed in practice.
- **Study:** MCP server discovery/install flow and the human-approval-per-step UX.

### 46. continuedev/continue — ~33k stars
- **What it is:** Source-controlled AI checks enforceable in CI, powered by the open-source Continue CLI; VS Code + JetBrains.
- **Why it matters:** Pioneered `.continue/` config-as-code for AI assistants — directly analogous to spec-driven development for agents.
- **Study:** `config.yaml` / `.continue/` directory layout — agents/rules/models/prompts as version-controlled files.

### 47. stitionai/devika — ~19k stars
- **What it is:** First open-source Agentic Software Engineer (started as Devin alternative); Agent Core orchestrating sub-agents (planning, research, coding, patching, reporting).
- **Why it matters:** Reference for the orchestrator + named-sub-agents pattern with explicit phase boundaries.
- **Study:** `ARCHITECTURE.md` — clean diagram of the sub-agent graph and message flow.

---

## Category 7: Memory / RAG for Agents

### 48. mem0ai/mem0 — ~54k stars
- **What it is:** Universal memory layer for AI agents; hybrid graph + vector + key-value datastore. Selected by AWS as the exclusive memory provider for their Agent SDK.
- **Why it matters:** De facto standard for portable agent memory in 2026; CrewAI, Flowise, Langflow all integrate natively.
- **Study:** Memory add/search/update API and the hybrid extraction → embedding → graph pipeline.

### 49. getzep/graphiti — ~20k stars
- **What it is:** Real-time temporal knowledge graphs for AI agents — facts evolve over time with provenance and ontology support.
- **Why it matters:** State-of-the-art temporal memory (better than static KGs for agents in changing environments).
- **Study:** Temporal edge model (valid_from / invalid_at) and bi-temporal fact storage.

### 50. run-llama/llama_index — ~43k stars
- **What it is:** Leading document agent and OCR platform; the canonical RAG framework.
- **Why it matters:** RAG is the most common tool spec-kit-generated agents will need; LlamaIndex sets the conventions for Document, Node, Index, QueryEngine.
- **Study:** Workflow primitives and the AgentWorkflow event-driven multi-agent pattern.

---

## Cross-cutting takeaways

- **YAML/Markdown-with-frontmatter is the dominant agent-spec medium.** SKILL.md (Anthropic), `agents.yaml`/`tasks.yaml` (CrewAI), declarative agent YAML (Microsoft Agent Framework), `.continue/config.yaml`, and the `.af` agent file format (Letta) all converge on declarative, version-controllable spec files. agents-spec-kit should pick one canonical shape and provide bidirectional adapters to the others.
- **The "skill folder" convention is set.** `<name>/SKILL.md` + `scripts/`, `references/`, `assets/` (Anthropic) is now the de facto layout, also adopted by obra/superpowers, wshobson/agents, davepoon/buildwithclaude, and Google ADK developer Skills. Spec-kit should produce this layout out of the box.
- **MCP is the universal tool protocol; A2A is the emerging agent-to-agent protocol.** Any spec-kit must scaffold MCP servers (stdio + streamable HTTP) and at least anticipate A2A. Both are now multi-vendor: Anthropic, Google, Microsoft, AWS Strands all support them.
- **Observability has standardized on OpenTelemetry GenAI semantic conventions.** Phoenix, Langfuse, OpenLLMetry, AgentOps-TS, LangSmith all emit OTel. Spec-kit should generate code with OTel hooks pre-wired, not custom SDKs.
- **Marketplaces are how skills/agents distribute.** anthropics/claude-plugins-official, wshobson/agents marketplace.json, buildwithclaude.com, VoltAgent's plugin packs, and obra/superpowers-marketplace all use a `marketplace.json`-style index. Spec-kit needs a `publish` story.
- **The "spec → code" precedent is github/spec-kit itself.** Spec Kit (~71k stars, 50+ countries) proves the appetite for treating specs as executable artifacts feeding a multi-phase pipeline. agents-spec-kit's job is to extend that pipeline with agent-specific phases (capability spec → skill scaffold → MCP tool spec → eval spec → observability config).
- **Patterns > frameworks.** humanlayer/12-factor-agents, neural-maze/agentic-patterns-course, and the `claude-cookbooks/patterns/agents/` notebooks all preach minimalism. The opportunity for agents-spec-kit is to be a *spec*, not a runtime — let CrewAI/LangGraph/OpenAI Agents SDK be the runtime.
- **Coding agents are the canary use case.** Aider, OpenHands, Cline, Continue, SWE-agent, Devika are the most-stressed open implementations. Their edit-format prompts and ACI conventions are reusable as Claude skills and should ship as scaffold templates.
- **Memory is no longer optional.** Mem0 (54k★, AWS-default), Letta (.af), Zep/Graphiti (20k★) signal that stateful agents are 2026's baseline. Spec-kit should have a `memory:` block in its agent spec — even if the value is `none`.
- **Three gaps spec-kit can fill:** (1) no canonical **agent spec → MCP server scaffold** generator exists, (2) no spec format **bridges** SKILL.md + CrewAI YAML + Letta `.af` + ADK skills, (3) no widely-adopted **eval-spec-as-code** for agents (Inspect AI is the closest but Python-only). A spec-kit that emits all three from one declarative source would be uniquely positioned.
