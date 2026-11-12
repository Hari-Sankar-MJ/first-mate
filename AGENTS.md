# AGENTS.md

Instructions for any AI coding agent (or human) working in this repo. Tool-agnostic: nothing
here assumes a particular assistant.

## Read first, every session

1. `log.md` — the **Current state** block at the top says where the work stands and what the
   exact next step is.
2. `Plan.md` — what we are building and why. Stable; changes only when the plan changes.
3. `decisions.md` — why things are the way they are. Check it before re-opening a settled
   question.

## How to work with the owner

- Start with a short plan for the current phase and wait for an OK before building.
- Build in small steps. Commit often with clear, conventional messages. Explain non-obvious
  decisions.
- Before writing Strands code, read the current Strands docs. Do not rely on memory for APIs
  (Ollama model import path, agents-as-tools, Graph/Workflow, MCP client, structured output,
  interrupts, session managers).
- Ask when a decision is genuinely the owner's (scope, trade-offs). Otherwise pick a sensible
  default, say what you chose, and record it in `decisions.md`.

## Non-negotiables

- **Local-first.** The default configuration must run fully on the local machine: no API
  keys, no calls to hosted model providers. Cloud models are an opt-in config change, never
  a code change, and never a requirement. See `Plan.md` → "Local-first, cloud-optional".
- **No unapproved side effects.** Anything that writes, sends, deletes, or runs a shell
  command goes through the human approval gate.
- **No secrets or personal data in git.** Secrets come from environment variables. Personal
  data lives in the gitignored `data/` directory. This applies to `log.md` and
  `decisions.md` too — the repo is public.

## Before ending a session

1. Rewrite the **Current state** block at the top of `log.md` (phase, what works, what is
   broken, exact next step, commands to run).
2. Add a dated entry under **Session log** in `log.md`, newest first: what was done, how it
   was verified, open issues.
3. Add a `decisions.md` entry for every decision made. Never edit an old entry's substance;
   add a new one and mark the old one `Superseded by D-xxx`.
4. If the plan itself changed, update `Plan.md`.
5. Commit.

A session is not finished until someone with no access to the conversation could resume
from these files alone.
