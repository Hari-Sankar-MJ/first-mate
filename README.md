# First Mate

A personal multi-agent assistant that runs entirely on your own machine. A coordinator
agent delegates work to small specialist agents. Built with the
[Strands Agents SDK](https://strandsagents.com/) and [Ollama](https://ollama.com/).

> **Status: planning.** No runnable code yet. Phase 0 (skeleton) is next. See
> [`log.md`](log.md) for the current state.

## What it is

- **Local by default.** The default configuration needs no API keys and makes no calls to
  hosted model providers. Your data stays on your machine.
- **Cloud when you choose.** Any agent can be pointed at a cloud model API by editing
  config and setting an environment variable. No code changes.
- **Coordinator and specialists.** One coordinator hands tasks to single-purpose agents
  (for example a files/notes helper, a web researcher, a repo helper), each with a small
  set of tools.
- **Configurable.** Models, providers, agents, tools and MCP servers are defined in YAML.
  Moving to another machine or model is a config edit.
- **Safe.** Anything that writes, sends, deletes or runs a command waits for human
  approval. Agents that read the web are read-only.
- **Tools over MCP.** Tools are MCP servers where possible, so other MCP clients can reuse
  them.

## Planned architecture

```
            CLI  /  HTTP API  /  web UI        (thin interfaces)
                        |
                   Coordinator
          ______________|______________
         |              |              |
   files/notes     web researcher   repo helper     (specialists, as tools)
         |              |              |
              MCP servers / tools
                        |
          Model factory (named profiles from config)
                        |
        Ollama (default)   |   cloud API (opt-in)
```

## Roadmap

| Phase | Scope |
|-------|-------|
| 0 | Skeleton: tooling, CI, config loading, model factory, CLI, one agent with one tool |
| 1 | Coordinator + 2-3 specialists, MCP tools, approval gate, local sessions |
| 2 | Tracing, evals, example cloud profile, architecture docs, demo |
| 3 | HTTP API with streaming, web UI, phone access over Tailscale |
| 4 | Model routing, more agents, notes memory, optional external coding agents |

Details and exit criteria for each phase are in [`Plan.md`](Plan.md).

## Requirements (planned)

- Python 3.12 and [uv](https://docs.astral.sh/uv/)
- [Ollama](https://ollama.com/) with a tool-capable model. Developed against `gpt-oss:20b`
  on a 16GB GPU; smaller models can be selected in config.

Setup and usage instructions will be added when Phase 0 lands.

## Project docs

- [`Plan.md`](Plan.md) — goals, principles, architecture, phases
- [`decisions.md`](decisions.md) — record of the main decisions and why they were made
- [`log.md`](log.md) — current state and session-by-session history
- [`AGENTS.md`](AGENTS.md) — working instructions for contributors and AI coding agents

## License

MIT (license file to be added in Phase 0).
