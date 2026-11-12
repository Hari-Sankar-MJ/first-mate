# Log

Working record for First Mate. Read **Current state** first; it is rewritten every session.
**Session log** below it is append-only, newest first.

## Current state

_Last updated: 2026-11-12_

- **Phase:** 0 (Skeleton) — not started. No code exists yet.
- **What exists:** `README.md`, `Plan.md`, `AGENTS.md`, `decisions.md`, `log.md`,
  `.gitignore`.
- **What works:** nothing to run yet.
- **Known issues:** none.
- **Unverified assumptions:**
  - gpt-oss:20b does reliable tool calling through Strands' Ollama provider (D-003).
  - Python 3.12 is supported by the current Strands release (D-009).
  - The models named in `Plan.md` are actually pulled in Ollama on this machine (not
    checked this session; run `ollama list`).
- **Next step:** write a short Phase 0 plan and get the owner's OK. Then, in order:
  1. Read current Strands docs (install, Ollama model provider, defining a tool).
  2. `pyproject.toml` with uv, ruff, pytest; `LICENSE` (MIT); package skeleton
     `src/first_mate/`.
  3. Config loading (YAML + Pydantic) with `config/default.yaml` — local-only, no keys.
  4. Model factory: profile -> model object, with a stub model for tests.
  5. CLI with one agent and one trivial tool.
  6. GitHub Actions: lint + unit tests.
  7. Check Phase 0 exit criteria in `Plan.md` (5/5 tool-calling runs on gpt-oss:20b).
- **Useful commands:** none yet. Add run/test/lint commands here once they exist.

## Session log

### 2026-11-12 — Project setup and plan review
- Reviewed the original `Plan.md` and agreed improvements with the owner.
- Created `AGENTS.md` (tool-agnostic working instructions and session protocol),
  `decisions.md` (seeded with D-001 to D-013), this `log.md`, and `.gitignore`.
- Updated `Plan.md`: added "Local-first, cloud-optional", "Tooling" and "Testing"
  sections; moved the approval gate to Phase 1 and tests/CI to Phase 0; added exit
  criteria per phase; assigned persistence and retrieval memory to phases; fixed all
  agents to one model profile until Phase 4; moved "How to work with me" into `AGENTS.md`.
- Expanded `README.md` and fixed a typo.
- Verification: documentation only; nothing to run.
- Open issues: `LICENSE` file not yet added (Phase 0). Tooling choices in D-009 are
  provisional.
