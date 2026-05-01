# hermes-upgrade-installation-operator

This package contains a Hermes skill plus companion routing and behavior policy files.

## Files

- `SKILL.md` — the passive skill prompt loaded by Hermes via `skill_view(...)`; also contains embedded fallback policies for non-policy-aware environments.
- `ROUTING.md` — agent-side routing policy. This tells Hex when to load the skill.
- `NUDGE.md` — skill execution behavior policy. This tells Hex how to behave after the skill is loaded.
- `metadata.yaml` — provider-agnostic metadata describing package roles, portability modes, load order, and Brainstack hints.

## Purpose

This skill guides safe installation, upgrade, repair, rollback, reinstall, migration, and verification workflows.

It requires the agent to verify current installed state, official upstream instructions, target version or desired state, environment risk, rollback readiness, and deterministic verification before producing mutating commands.

## Installation

1. Copy this directory into the Hermes skills directory.
2. Install or replace the skill body with `SKILL.md`.
3. Import or save `ROUTING.md` into Brainstack as an `agent_routing_policy` memory.
4. Import or save `NUDGE.md` into Brainstack as a `skill_behavior_policy` memory.
5. Do not store `SKILL.md` as ordinary Brainstack memory unless Hermes requires that fallback.

## Brainstack handling

Brainstack should treat `ROUTING.md` and `NUDGE.md` as policy-like memory entries, not as user preference memories.

Recommended labels:

```text
TYPE: agent_routing_policy
SCOPE: skill_activation
SKILL: hermes-upgrade-installation-operator
```

```text
TYPE: skill_behavior_policy
SCOPE: only_when_skill_loaded
SKILL: hermes-upgrade-installation-operator
```

## Runtime flow

```text
User message
-> Hex checks Brainstack routing policies
-> matching routing policy found
-> Hex calls skill_view(name="hermes-upgrade-installation-operator")
-> Hex loads SKILL.md
-> Hex applies NUDGE.md only for this skill invocation
-> Hex follows the skill protocol
```

## Required protocol

```text
Phase 1 — Understanding Proof
-> classify intent
-> classify environment
-> verify current state
-> verify official source basis
-> define target state
-> identify risk and rollback
-> define verification plan
-> STOP and wait for confirmation

Phase 2 — Output Generation
-> exact commands
-> atomic checkpoints
-> verification
-> rollback
-> change summary
```

## Replacement guidance

- Replace the existing `SKILL.md` for this skill.
- Adopt `ROUTING.md` if you want this trigger policy.
- Adopt `NUDGE.md` if you want the associated behavior guidance.
- Keep routing and nudge separate. They are policy artifacts, not normal user memories.

## Portability Modes

This package supports two installation styles.

### Optimized policy mode

Use this when the agent or memory provider supports external policy storage.

- Install `SKILL.md` as the skill body.
- Store `ROUTING.md` as a skill activation policy.
- Store `NUDGE.md` as skill execution guidance.
- Apply `NUDGE.md` only when this skill is loaded.

This is the recommended mode for Hermes + Brainstack.

### Embedded fallback mode

Use this when the agent does not support external routing or behavior policy storage.

- Install only `SKILL.md`.
- Use the `Embedded Policies (Fallback Mode)` section inside `SKILL.md` for routing and usage guidance.
- Keep `ROUTING.md` and `NUDGE.md` as human/auditable references.

This mode is less modular, but more portable.

## Provider-Agnostic Policy Handling

For non-Brainstack memory providers, translate policy intent rather than schema details:

- `ROUTING.md` means: "When should this skill be activated?"
- `NUDGE.md` means: "How should the agent behave after this skill is activated?"
- Neither file should be treated as a normal user preference.
- Neither file should be summarized into generic memory.

Suggested provider mappings:

| Provider style | Routing storage | Nudge storage |
|---|---|---|
| File-based | `ROUTING.md` | `NUDGE.md` |
| Vector DB | policy document tagged `skill_activation` | policy document tagged `skill_execution` |
| Key-value / SQL | record type `skill_routing_policy` | record type `skill_behavior_policy` |
| Prompt-only | embedded fallback section in `SKILL.md` | embedded fallback section in `SKILL.md` |
| Brainstack | operating/canonical policy | operating/canonical policy |
