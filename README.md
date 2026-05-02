# platform_repos/harnesses / agents-sdk-python

The **v1 active runtime** for tinyhat — a pinned mirror of the
[OpenAI Agents SDK Python](https://github.com/openai/openai-agents-python)
+ the tinyhat-side `Agent` composition that wraps it. This is the
only harness whose model loop tinyhat actually executes today
(`agent_service.run_agent` → `Runner.run` per turn, with
`ShellTool` for the per-run sandbox).

The other curated rows (`codex-cli`, `opencode`, `goose`) are
vendored as **future-option** mirrors and wait for their per-harness
runner adapter to ship in v1.x. See PR #120 follow-up issues.

This directory in the main tinyloop repo is the **first-commit
payload** for the eventual system repo
`github.com/tinyloophub/tld--sys--tinyhat-harness-agents-sdk-python`. The
contents that ship in this PR:

| File | Purpose |
| --- | --- |
| `HARNESS.md` | AgentSkills-shaped manifest with the pinned upstream ref + capability metadata. |
| `LICENSE` | Copy of upstream MIT LICENSE (Copyright 2025 OpenAI). |
| `sbom.spdx.json` | SPDX 2.3 SBOM covering the directly vendored upstream package. |
| `patches/` | Empty — the wrapper applies no tinyhat-side patches at v1. |

The actual upstream is **not** vendored here in the main repo —
that's the system repo's job once it exists. The platform-side
adapter that wires the harness to our `Agent` composition lives in
`backend/domains/tinyhat/services/agent_service.py`.

See [HARNESS.md](./HARNESS.md) for the manifest body, and
`Tinyhat Docs/engineering-playbook/tinyhat-runtime-architecture.md`
§"Harness — the agent's runtime loop" (Drive) for the full
architecture.
