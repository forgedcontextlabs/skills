# system-eval-operator

This package contains a Hermes skill plus its companion routing and behavior policy files.

## Files

- `SKILL.md` — the passive skill prompt loaded by Hermes via `skill_view(...)`; also contains embedded fallback policies for non-policy-aware environments.
- `ROUTING.md` — agent-side routing policy. This tells Hex when to load the skill.
- `NUDGE.md` — skill execution behavior policy. This tells Hex how to behave after the skill is loaded.
- `metadata.yaml` — provider-agnostic metadata describing package roles, portability modes, load order, and Brainstack hints.

## Installation

1. Copy this directory into the Hermes skills directory.
2. Replace the existing skill file with `SKILL.md` if the skill already exists.
3. Import or save `ROUTING.md` into Brainstack as an `agent_routing_policy` memory.
4. Import or save `NUDGE.md` into Brainstack as a `skill_behavior_policy` memory.
5. Do not store `SKILL.md` as an ordinary Brainstack memory unless Hermes requires that fallback.

## Brainstack handling

Brainstack should treat `ROUTING.md` and `NUDGE.md` as policy-like memory entries, not as user preference memories.

Recommended labels:

```text
TYPE: agent_routing_policy
SCOPE: skill_activation
SKILL: system-eval-operator
```

```text
TYPE: skill_behavior_policy
SCOPE: only_when_skill_loaded
SKILL: system-eval-operator
```

## Runtime flow

```text
User message
-> Hex checks Brainstack routing policies
-> matching routing policy found
-> Hex calls skill_view(name="system-eval-operator")
-> Hex loads SKILL.md
-> Hex applies NUDGE.md only for this skill invocation
-> Hex follows the skill protocol
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
