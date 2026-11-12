# Decisions

Record of the main decisions for First Mate. One entry per decision, numbered in order.

Rules:
- Add a new entry for every decision that a future session might otherwise question or redo.
- Never rewrite an old entry's substance. To change course, add a new entry and set the old
  one's status to `Superseded by D-xxx`.
- Status values: `Accepted`, `Provisional` (default picked, not yet proven in practice),
  `Superseded by D-xxx`.

Template:

```
## D-000 — Title
- Date: YYYY-MM-DD
- Status: Accepted | Provisional | Superseded by D-xxx
- Context: what forced the decision
- Decision: what was chosen
- Alternatives: what was rejected and why
- Consequences: what this commits us to
```

---

## D-001 — Strands Agents SDK (Python) as the agent framework
- Date: 2026-11-12
- Status: Accepted
- Context: Need a framework for a coordinator + specialist design that supports local
  models, MCP tools, human approval, and sessions. Learning agent engineering is a goal.
- Decision: Build on the Strands Agents SDK in Python.
- Alternatives: Not recorded; chosen by the owner before this record began.
- Consequences: Always check current Strands docs before writing against its APIs.

## D-002 — Local-first; cloud is opt-in by config only
- Date: 2026-11-12
- Status: Accepted
- Context: The assistant handles personal data and must work with no internet model
  provider. The owner also wants to be able to use a cloud API when they choose.
- Decision: The default config runs fully locally with no API keys. Cloud providers are
  selected by editing model profiles in config and setting an env var; no code change.
  Cloud SDKs are optional install extras. An active cloud profile is shown in the
  interface. A test guards that the default config needs no keys.
- Alternatives: Cloud-first with a local fallback — rejected, wrong privacy default.
  Local-only — rejected, the owner wants the cloud option.
- Consequences: Every model reference goes through the model factory and a named profile.
  No provider-specific code outside the factory.

## D-003 — Ollama as the first provider, gpt-oss:20b as the main model
- Date: 2026-11-12
- Status: Provisional
- Context: Primary machine has 16GB VRAM. gpt-oss:20b (~14GB) supports tools and thinking
  and fits at <= 32k context.
- Decision: Ollama at localhost:11434 with gpt-oss:20b; qwen3:8b as fast fallback.
- Alternatives: Other local runtimes — not evaluated.
- Consequences: Provisional until Phase 0 proves reliable tool calling through Strands.
  Fallback order if it is flaky: Ollama's OpenAI-compatible endpoint, then qwen3:8b.

## D-004 — Coordinator uses agents-as-tools, one level deep
- Date: 2026-11-12
- Status: Accepted
- Context: Need delegation that a 20B local model can drive reliably within 32k context.
- Decision: Specialists are exposed to the coordinator as tools; specialists do not
  delegate further. Agents stay small (3-6 tools). Graph/Workflow for fixed pipelines;
  Swarm only if it proves useful.
- Alternatives: Swarm / deeper hierarchies — deferred as harder to debug and heavier on
  context.
- Consequences: New capabilities are added as new specialists, not by growing one agent.

## D-005 — Prefer MCP servers for tools; own tools are written as MCP servers
- Date: 2026-11-12
- Status: Accepted
- Context: Tools should be reusable outside this project.
- Decision: Use existing MCP servers where possible; write custom tools as MCP servers.
- Alternatives: Framework-native tool functions only — rejected, not reusable by other
  MCP clients.
- Consequences: MCP server definitions live in config. Tool schemas count against the
  context budget, which reinforces the 3-6 tools per agent limit.

## D-006 — Core library first, then CLI, then HTTP API, then web UI
- Date: 2026-11-12
- Status: Accepted
- Context: Multiple interfaces are planned (CLI, API, web, maybe chat bots).
- Decision: All logic lives in the core package; interfaces are thin. Build order:
  CLI -> FastAPI -> React web UI.
- Alternatives: UI-first — rejected, couples logic to one interface.
- Consequences: No interface may contain agent logic.

## D-007 — Human approval for all side effects; gate lands in Phase 1
- Date: 2026-11-12
- Status: Accepted
- Context: Web content can carry prompt injection and reaches the coordinator through
  specialist answers. The original plan put approvals in Phase 2, after a file-writing
  agent in Phase 1.
- Decision: Write/send/delete/shell actions require approval via Strands interrupts. The
  gate is built in Phase 1, and no side-effecting tool is added before it exists. Approval
  prompts show the exact tool name and arguments. Web-reading agents are read-only. No
  unsupervised shell tool.
- Alternatives: Approvals in Phase 2 — rejected, leaves a window with ungated writes.
- Consequences: Phase 1 is larger; Phase 2 is smaller.

## D-008 — Phone access over Tailscale with an API token
- Date: 2026-11-12
- Status: Accepted
- Context: The owner wants to use the assistant and grant approvals from a phone.
- Decision: Responsive, installable web UI reached over Tailscale. No public exposure.
  API requires an auth token even inside the tailnet. Ollama's port is never exposed.
- Alternatives: Public hosting — rejected. Telegram/Slack bot — possible later addition.
- Consequences: Server-side session storage is needed in Phase 3.

## D-009 — Tooling: Python 3.12, uv, ruff, pytest, Pydantic-validated YAML config, MIT
- Date: 2026-11-12
- Status: Provisional
- Context: The plan did not name tooling. Defaults proposed by the agent, accepted in
  principle by the owner; not yet exercised.
- Decision: Python 3.12; uv for environments and dependencies; ruff for lint/format;
  pytest; YAML config validated by Pydantic models with env-var overrides; MIT license.
- Alternatives: pip + venv, Poetry — uv is faster and simpler. Other licenses — MIT is
  the simplest for a portfolio project.
- Consequences: Confirm the Python version against Strands' supported versions in Phase 0.

## D-010 — Tests and CI from Phase 0; stub model in CI, real-model evals local
- Date: 2026-11-12
- Status: Accepted
- Context: The original plan put tests and CI in Phase 2. CI runners cannot host a 14GB
  model.
- Decision: Lint + unit tests in GitHub Actions from Phase 0, using a stub model. Evals
  against a real model are run locally and by hand; results go in `log.md`.
- Alternatives: Tests later — rejected, harder to retrofit. Cloud model in CI — rejected,
  needs secrets and contradicts D-002.
- Consequences: The model factory must make a stub model easy to inject.

## D-011 — One shared model profile for all agents until Phase 4
- Date: 2026-11-12
- Status: Accepted
- Context: On 16GB VRAM, gpt-oss:20b cannot stay loaded alongside another large model;
  mixing models would swap on every delegation.
- Decision: Default config assigns the same profile to every agent until routing work in
  Phase 4. Per-agent profiles remain supported in config.
- Alternatives: Small model for specialists now — rejected for swap latency.
- Consequences: Phase 4 routing must account for swap cost.

## D-012 — Persistence in two steps; retrieval memory deferred to Phase 4
- Date: 2026-11-12
- Status: Accepted
- Context: The original plan listed session storage and an embeddings model without
  assigning either to a phase.
- Decision: Local file-based sessions for the CLI in Phase 1; server-side sessions with
  the API in Phase 3; notes memory/retrieval with nomic-embed-text in Phase 4.
- Alternatives: Server-side storage from the start — rejected, no server exists yet.
- Consequences: Session storage sits behind an interface so the backend can change.

## D-013 — Tool-agnostic project docs; log.md and decisions.md are committed
- Date: 2026-11-12
- Status: Accepted
- Context: Work will be picked up by different sessions and possibly different coding
  assistants. The repo is a public portfolio project.
- Decision: Working instructions live in `AGENTS.md` and name no specific assistant.
  `log.md` (state + history) and `decisions.md` (this file) are committed and kept free of
  secrets and personal data.
- Alternatives: Assistant-specific instruction files; keeping the log private — rejected,
  less portable and the record is useful to readers of the repo.
- Consequences: Every session ends by updating `log.md` and, where relevant, this file.
