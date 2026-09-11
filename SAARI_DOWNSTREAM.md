<!--
  SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
  SPDX-License-Identifier: Apache-2.0
-->

# Saariusly Stoned Downstream

This fork preserves the gateway-owned subscription providers used by the
Saariusly Stoned OpenClaw sandbox fleet while continuing to take upstream
OpenShell releases. It is maintained independently and is not an NVIDIA support
channel.

## Pinned Release Pair

Use these two immutable tags together:

- `saariuslystoned/OpenShell` at `v0.0.106-saari.2`
- `saariuslystoned/NemoClaw` at `v0.0.114-saari.8`

Do not install this downstream from a moving `main`, `latest`, or feature
branch. The paired NemoClaw release checks for the required OpenShell inference
flags and registered provider before it mutates a sandbox.

## Mitzo Production Release

Mitzo uses a separate downstream line based on upstream OpenShell `v0.0.116`:

- immutable release: `dimakis/OpenShell` at `v0.0.116-mitzo.1`
- maintenance branch: `codex/mitzo-oauth-v0.0.116`
- required provider name: `mitzo-personal-subscription`
- required provider type: `openai-codex-oauth`

The Mitzo release carries the subscription OAuth broker and `inference.local`
routing that complement Mitzo's own broker-binding enforcement. Deploying the
Mitzo application alone is insufficient: the gateway must run this downstream
release (or a later explicitly validated port) for attended subscription login
and credential-free sandbox inference to work.

Never replace this gateway with a stock OpenShell upgrade. Start each upgrade
from the new immutable upstream tag, port the downstream commits, and prove the
provider login, stored refresh handles, refresh/reauthorization state machine,
`inference.local` routing, and credential-free inference in an isolated gateway
before changing production.

## Maintained Provider Scope

This release adds gateway-owned OAuth and routing for:

- `codex-subscription`, used by `gpt-5.6-sol` and `gpt-5.6-terra`
- `grok-subscription`, used by `grok-4.6`

It also normalizes Codex subscription responses, keeps route-cache behavior
fail-closed, proves both subscription providers through OpenClaw, and isolates
unbound static credentials from sandbox materialization.

The gateway holds provider credentials, but it has no gateway-wide inference
selection. Each sandbox receives its provider and model as a create-time route.
The Codex proxy normalizes generic OpenAI Responses clients to the narrower
subscription request schema while preserving streaming, tools, reasoning, and
safe text options.

The paired NemoClaw release owns these named sandbox roles:

| Role | Live sandbox | Provider | Model |
|------|--------------|----------|-------|
| FORGE | `spark02-forge` | `codex-subscription` | `gpt-5.6-sol` |
| RANGER | `spark02-ranger` | `codex-subscription` | `gpt-5.6-terra` |
| SPARK | `spark02-assistant` | `grok-subscription` | `grok-4.6` |

## Upstream Update Contract

For each new upstream release:

1. Fetch the immutable upstream OpenShell and NemoClaw release tags.
2. Create one isolated downstream worktree per repository from those tags.
3. Port the subscription-provider commits and reconcile changed upstream
   provider, router, persistence, sandbox, and CLI semantics.
4. Run `mise run pre-commit`, the focused provider/router tests, and
   `mise run ci` before publishing the OpenShell tag.
5. Publish new paired `-saari.N` tags and record their exact commits in the
   release notes.
6. Install only the paired tags, then verify one live sandbox at a time with
   rollback snapshots available.

For Mitzo releases, apply the same sequence to the `-mitzo.N` tag and update the
Mitzo repository's `docs/operations/openshell-subscription-upgrades.md` record
with the upstream base, downstream tag, exact commit, validation evidence, and
rollback path.

Upstream remains the source for general OpenShell behavior. This downstream
owns the subscription-provider compatibility port and the proof needed to keep
it working across upstream upgrades.
