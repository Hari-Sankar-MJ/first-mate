# First Mate

Personal, local-first multi-agent assistant: a coordinator delegates to specialist agents.
Built with the Strands Agents SDK (Python). Goals: (1) a tool I use daily, (2) a way to
learn agent engineering properly, (3) a polished GitHub portfolio project.

Companion files: `AGENTS.md` (how to work in this repo), `log.md` (current state and
session history), `decisions.md` (decision record).

## Principles
- Local and private by default; cloud models are an opt-in config choice.
- Core library separate from interfaces: core -> CLI first -> HTTP API -> web UI later.
- Everything configurable: models, providers, agents, tools, MCP servers. Nothing hardcoded.
- Portable: must run on another machine with a different model by editing config only.
- Portfolio quality: clear README, architecture diagram, tests, CI, demo GIF, license.
- Safety first: side-effecting actions need human approval.

## Local-first, cloud-optional
- The default config runs completely locally: no API keys, no calls to hosted model
  providers. A fresh clone plus Ollama is enough.
- Switching an agent (or all agents) to a cloud API is a config edit: point it at a
  different model profile and set that provider's key in an env var. No code changes.
- Cloud provider SDKs are optional install extras, not core dependencies, so a local-only
  install pulls in nothing it does not use.
- Nothing leaves the machine by default: telemetry/traces export locally unless an external
  endpoint is configured.
- When any cloud profile is active, the interface says so visibly (which agent, which
  provider), so it is never a surprise that data is leaving the machine.
- A test asserts that the default config loads and builds its models with no provider keys
  set.

## Environment (primary machine)
- Windows 11, Ryzen 7 9800X3D, RTX 5080 (16GB VRAM), 32GB RAM
- Ollama at http://localhost:11434; main model gpt-oss:20b (~14GB, tools + thinking).
- Keep context <= 32k (OLLAMA_CONTEXT_LENGTH=32768). `ollama ps` should show 100% GPU.
- Existing fix: CUDA_VISIBLE_DEVICES=0 stops Ollama falling back to CPU.
- Other models available: qwen3:8b (fast fallback), nomic-embed-text (embeddings).

## Tooling
- Python 3.12, managed with uv (`pyproject.toml`, venv in `.venv/`).
- ruff (lint + format), pytest.
- Config: YAML files validated with Pydantic models; env vars for secrets and overrides.
- License: MIT.

## Architecture
- Config (YAML + env vars): named model profiles, agent definitions (name, description,
  system prompt, model profile, tools), MCP servers.
- Model factory: profile -> Strands model object. Ollama first; design for other providers
  (OpenAI-compatible, Anthropic, Bedrock, LiteLLM) without code changes elsewhere.
- Agents: small, single-purpose, 3-6 tools each. Coordinator uses agents-as-tools, one level
  deep. Use Strands Graph/Workflow for fixed pipelines; Swarm only if it proves useful.
- Tools: prefer MCP servers (filesystem, web fetch/browser, search) over custom code.
  Write my own tools as MCP servers so other MCP clients can reuse them.
- Structured outputs (Pydantic models) for anything the code consumes.
- Observability: OpenTelemetry traces + debug logging of tool calls.
- Persistence: local file-based sessions for the CLI first; server-side session storage
  with the HTTP API so conversations continue across devices.

## Model routing
- Each agent references a named model profile; different agents may use different models
  (small/fast, main reasoning, coder, cloud).
- Until Phase 4, every agent in the default config uses the same model profile. On 16GB
  VRAM gpt-oss:20b can't co-reside with another large model, so mixing models would swap
  on every delegation.
- Later: rule-based routing by task type, then optionally a coordinator tool requesting a
  tier ("fast" | "smart" | "cloud"). Group work by model; use Ollama keep_alive
  deliberately.

## Safety
- Web-reading agents are read-only (web content may contain prompt injection).
- Web content still reaches the coordinator through a specialist's answer, so the approval
  gate is the real defence: approval prompts show the exact tool name and arguments, never
  a model-written summary.
- Side-effecting actions (write, send, delete, shell, external coding agents) require
  explicit human approval via Strands interrupts. No unsupervised shell tool.
- No side-effecting tool is added to any agent before the approval gate exists.
- Secrets via env vars only. Never commit secrets or personal data.

## Testing
- Unit tests use a fake/stub model: fast, deterministic, no Ollama needed. These run in CI.
- Evals run against a real model and are local and manual (CI can't host a 14GB model).
  Results are recorded in `log.md`.

## Remote access (phone)
- Plan for phone access: responsive web UI (installable as a home-screen web app) reached
  over Tailscale. No public exposure. Ollama's port is never exposed.
- API requires an auth token even inside Tailscale.
- Approvals for risky actions are surfaced in the UI so they can be done from the phone.
- Possible later alternative interface: a Telegram or Slack bot.

## External coding agents (optional, much later)
- Headless coding-agent CLIs may be wrapped as optional specialist tools behind a common
  "coding agent" interface so the provider is swappable.
- Opt-in via config, off by default (cloud-backed). Always approval-gated.
- Check the provider's terms and billing for third-party use before building this.

## Layout
src/first_mate/   config.py, models.py, orchestrator.py, agents/, tools/, cli.py, api/ (later)
web/              (later) React + TypeScript UI over the API, streaming responses
config/           default.yaml + example profiles (local and cloud)
tests/, evals/    unit tests + a small set of agent eval cases
docs/             architecture, setup, how to add an agent

## Phases
Each phase is done only when its exit criteria are met and recorded in `log.md`.

0. Skeleton: pyproject + uv, ruff, pytest, CI (GitHub Actions: lint + unit tests),
   .gitignore, LICENSE, config loading, model factory, CLI, one agent with one trivial tool.
   - Exit: `first-mate` CLI answers a prompt by calling the tool on gpt-oss:20b, 5 runs out
     of 5. Unit tests pass in CI with a stub model. Default config needs no API keys.
   - Fallback if gpt-oss:20b tool calling is unreliable through Strands' Ollama provider:
     try Ollama's OpenAI-compatible endpoint, then qwen3:8b. Record the outcome in
     `decisions.md`.
1. Coordinator + 2-3 specialists I'd actually use (e.g. files/notes helper, web researcher,
   repo helper); MCP tools; approval gate; local file sessions; one shared model profile.
   - Exit: coordinator delegates to the right specialist on a small set of scripted
     prompts. A write action pauses for approval and does nothing when denied. A CLI
     conversation resumes after restart.
2. Tracing, evals, broader tests, one example cloud profile, README + architecture diagram
   + demo.
   - Exit: a trace shows coordinator -> specialist -> tool spans. Eval set runs with one
     command. Switching to the example cloud profile works by config edit only.
3. HTTP API (FastAPI, streaming) with server-side sessions, then web UI; Tailscale phone
   access.
   - Exit: a conversation started in the CLI continues from the phone; an approval can be
     granted from the phone; unauthenticated requests are rejected.
4. Routing rules, more agents, notes memory/retrieval using nomic-embed-text, optional
   external coding agents.
   - Exit criteria to be defined when Phase 3 closes.

## Conventions
- Repo `first-mate`; Python package `first_mate`. Python venv in .venv/.
- .gitignore: .venv/, .env, data/, *.pdf. Personal data lives in gitignored data/.
- Small, conventional commits. Keep README and docs current as features land.

## Repo description (GitHub)
Local multi-agent assistant built with Strands Agents and Ollama. A coordinator delegates to
specialist agents, all running privately on your own machine.
Topics: strands-agents, ollama, multi-agent, local-llm, ai-agents, gpt-oss, mcp, python
