---
name: tinyhat-agents-sdk-python
description: The v1 active runtime — OpenAI Agents SDK driven by Tinyhat's sandbox composition.
license: MIT
metadata:
  tinyhat:
    harness:
      upstream_url: https://github.com/openai/openai-agents-python
      upstream_ref: v0.14.1
      upstream_sha: 3dffa4ba9351ee38b964af43b50a2fe70f1a89f6
      system_repo_url: https://github.com/tinyloophub/tld--sys--tinyhat-harness-agents-sdk-python
      sbom: ./sbom.spdx.json
      slsa_level: 1
      sandbox_tiers: [0, 1, 2]
      agent_skills_features:
        - progressive_disclosure
        - scripts
        - references
        - assets
      mcp_support: false
      otel_support: false
      supported_providers:
        - openai
---

# tinyhat-agents-sdk-python

The **v1 active runtime** for tinyhat. A pinned mirror of
[`openai/openai-agents-python`](https://github.com/openai/openai-agents-python)
plus the tinyhat-side composition that wires it into the platform's
`Agent` shape (skill mounts, function tools, IdentityContext,
`gated_api_call`).

A note on the word "harness": in the runtime sense, a harness is a
*complete model-loop runner* (think Codex CLI, OpenHands). The
OpenAI Agents SDK is a *framework library* — tinyhat composes its
own `Agent` per turn and runs `Runner.run` in-process. For v1 that
composition IS our runtime; until per-harness runner adapters ship
(Codex CLI, OpenHands — see PR #120 follow-up issues), this is the
only harness whose loop tinyhat actually executes. The other rows
in `tinyhat_harness` (codex-cli, opencode, goose) are vendored as
future-option mirrors so the moment a runner adapter lands, the
metadata is already there.

This file is the AgentSkills-shaped manifest the platform parses
at vendor time. The frontmatter records:

- **upstream_url / upstream_ref / upstream_sha** — the exact
  upstream tag + commit we vendored. Bumping = vendoring a new
  ref + writing a new `tinyhat_harness_version` row.
- **sandbox_tiers** — the tinyhat skill tiers this harness can
  drive. Tier 0/1/2 covers everything declarative + Python +
  Bun/Node, which matches the upstream `ShellTool` shape.
- **agent_skills_features** — the AgentSkills 1.0 features
  this harness honours (`progressive_disclosure`, `scripts`,
  `references`, `assets`).
- **mcp_support / otel_support** — false at v1; enabled in a
  follow-up version row when we land them.
- **slsa_level** — SLSA build provenance level. v1 ships at L1;
  L3 is the post-launch target for self-hosted shipments.
- **supported_providers** — `["openai"]` at v1. The OpenAI Agents
  SDK is OpenAI-API-only at this version; planned support for
  Anthropic + Google via the SDK's extended-models layer post-v1.

## Vendor compliance

The upstream is licensed under MIT (Copyright 2025 OpenAI). We
preserve the LICENSE file at the root of this mirror (alongside
this `HARNESS.md`). MIT does not require a separate NOTICE file
and the upstream does not ship one.

Significant tinyhat-side modifications live as patch files under
`patches/`, named descriptively (e.g.
`patches/0001-tinyhat-add-OTel-spans.patch`). At the v1 launch
this directory is empty — the wrapper applies no patches; the
tinyhat-side adapter logic lives in
`backend/domains/tinyhat/services/agent_service.py` in the main
tinyloop repo, not here.

## SBOM

`sbom.spdx.json` is a [SPDX 2.3](https://spdx.github.io/spdx-spec/)
document covering the upstream package's transitive dependency
tree at the pinned version. It's regenerated on every version
bump.

## Where the runtime hookup lives

The harness's `Agent`-shaped composition is built in
`backend/domains/tinyhat/services/agent_service.py` in the main
tinyloop repo:

- `_build_agent` — picks the model + ModelSettings out of the
  agent's `tinyhat_agents.model_provider / model_name /
  model_options` columns.
- `_build_shell_tool` — composes the in-sandbox capability
  surface (skill mounts, network policy, memory limit).
- `harness_resolver_service` — joins `tinyhat_agents` to this
  harness through `harness_version_id` and refuses any
  `model_provider` outside the `supported_providers` list above.

That split is the load-bearing thing: this directory carries the
upstream code we mirror; the tinyloop-side `agent_service` is the
thin adapter that does *not* live in the mirror, so we never have
to fork upstream history to maintain the platform integration.

## Links

- Upstream: <https://github.com/openai/openai-agents-python>
- Tinyhat runtime architecture (Drive):
  `Tinyhat Docs/engineering-playbook/tinyhat-runtime-architecture.md`
  §"Harness — the agent's runtime loop".
- Issue introducing the schema: `tinyloophub/tinyloop#100`.
