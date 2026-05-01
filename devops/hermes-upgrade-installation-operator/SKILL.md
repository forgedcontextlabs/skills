---
name: hermes-upgrade-installation-operator
description: Safely guide installation, upgrade, repair, rollback, and verification workflows by verifying current state, official upstream instructions, environment risk, rollback readiness, and deterministic post-change checks before producing mutating commands.
category: devops
version: 1.0.0
author: Forged Context Labs
license: MIT
---

# Hermes Upgrade & Installation Operator

## Rule

**Before every install, upgrade, repair, rollback, reinstall, migration, or system setup action** — verify the current system state, the official upstream instructions, the target version or desired state, the execution environment, the rollback path, and the verification plan.

This skill overrides general assistant behavior for installation, upgrade, repair, rollback, migration, dependency setup, service setup, and other system-mutation tasks. Helpfulness must not bypass the safety protocol.

Never generate mutating commands until Phase 1 is complete and the user explicitly confirms the proposed plan.

Read-only inspection commands are allowed during Phase 1. Mutating commands belong only in Phase 2 after confirmation.

If this skill should apply but has not been applied, the response is invalid. Redirect immediately to Phase 1 and do not produce mutating commands.

## Core Purpose

This skill controls safe installation and upgrade workflows for Hermes Agent components, related tools, services, dependencies, and local development environments.

It exists to prevent the agent from guessing install methods, mixing package managers, overwriting local state, skipping official instructions, or declaring success without verification.

## Mandatory Execution Phases

The execution phases are mandatory. They are not advisory formatting preferences. Any response that bypasses Phase 1, omits the stop point, or includes mutating commands before confirmation is non-compliant.

### Phase 1 — Understanding Proof

The agent must present a concise understanding proof before any mutating command is produced.

The proof must include:

1. **Intent classification**
   - fresh install
   - upgrade
   - repair
   - rollback
   - reinstall
   - migration
   - verification only
   - unknown

2. **Environment classification**
   - local dev
   - VM
   - container
   - server
   - production
   - unknown-sensitive
   - OS/distribution when available
   - privilege level when relevant

3. **Current state**
   - component name
   - current version, if installed
   - install path, if detectable
   - install method or package manager, if detectable
   - service/process status, if applicable
   - config/data paths, if relevant

4. **Official source basis**
   - official documentation, GitHub README, release notes, install script documentation, or package registry page only when officially linked
   - whether official install/upgrade instructions were found
   - whether the plan is official, partially official, or inferred

5. **Target state**
   - target version or desired installed state
   - reason for change, if known
   - compatibility or breaking-change concerns, if known

6. **Risk and rollback**
   - files, paths, services, packages, or configs likely affected
   - backup/snapshot/checkpoint status
   - rollback path or recovery limitation

7. **Verification plan**
   - version check
   - service status check
   - smoke test or health check
   - log sanity check where applicable

After presenting Phase 1, the agent must stop and wait for user confirmation.

### Phase 2 — Output Generation

Only after explicit user confirmation, the agent may produce commands.

Phase 2 output must include:

1. ordered execution steps
2. exact commands
3. explanation of what each step changes
4. checkpoints between risky steps
5. verification commands
6. rollback commands or rollback procedure
7. post-execution change summary template

The agent must not skip the confirmation gate.

## Response Validity Requirements

A response is invalid if any of the following occur:

- mutating commands appear before explicit user confirmation
- Phase 1 does not include intent, environment, current state, official source basis, target state, risk/rollback, verification plan, proposed plan, and STOP
- official source status is omitted for install, upgrade, rollback, repair, reinstall, or migration tasks
- current installed state is assumed instead of verified or marked unknown
- rollback or backup status is omitted before risky mutation
- destructive impact is not stated before removal, overwrite, restart, migration, or replacement
- verification checks are missing from Phase 2
- the response treats ROUTING.md or NUDGE.md as required for correctness

When a response would be invalid, stop, state the missing requirement, and return to Phase 1 with read-only inspection steps only.

## Quick Inspection Commands

Use commands appropriate to the operating system and tool. Prefer read-only inspection first.

**See also:** `references/brainstack-installation.md` — Brainstack memory plugin installation procedure and common failure modes.

### Environment

```bash
uname -a
cat /etc/os-release 2>/dev/null || true
whoami
id
pwd
```

### Privilege and shell context

```bash
command -v sudo >/dev/null 2>&1 && sudo -n true 2>/dev/null && echo "sudo: available" || echo "sudo: unavailable or requires password"
echo "$SHELL"
echo "$PATH"
```

### Common install location discovery

```bash
command -v hermes 2>/dev/null || true
command -v hex 2>/dev/null || true
which -a hermes 2>/dev/null || true
which -a hex 2>/dev/null || true
```

### Version discovery examples

```bash
hermes --version 2>/dev/null || true
hex --version 2>/dev/null || true
python --version 2>/dev/null || true
python3 --version 2>/dev/null || true
node --version 2>/dev/null || true
npm --version 2>/dev/null || true
git --version 2>/dev/null || true
```

### Package manager discovery

```bash
command -v apt 2>/dev/null || true
command -v dnf 2>/dev/null || true
command -v yum 2>/dev/null || true
command -v pacman 2>/dev/null || true
command -v brew 2>/dev/null || true
command -v pip 2>/dev/null || true
command -v pipx 2>/dev/null || true
command -v npm 2>/dev/null || true
command -v pnpm 2>/dev/null || true
command -v yarn 2>/dev/null || true
command -v docker 2>/dev/null || true
```

### Git repository state

Run only inside the relevant repository:

```bash
git status --short
git branch --show-current
git remote -v
git log -1 --oneline
```

### Python package state

```bash
python3 -m pip show PACKAGE_NAME 2>/dev/null || true
pipx list 2>/dev/null || true
```

### Node package state

```bash
npm ls -g --depth=0 2>/dev/null || true
npm ls --depth=0 2>/dev/null || true
```

### Service status

```bash
systemctl status SERVICE_NAME --no-pager 2>/dev/null || true
systemctl is-enabled SERVICE_NAME 2>/dev/null || true
journalctl -u SERVICE_NAME -n 80 --no-pager 2>/dev/null || true
```

### Container state

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}' 2>/dev/null || true
docker compose ps 2>/dev/null || true
```

### Hermes skill installation state

```bash
# Check if skill directory exists
ls -la ~/.hermes/skills/SKILL_NAME/ 2>/dev/null || true

# Check skill files present
ls ~/.hermes/skills/SKILL_NAME/{SKILL.md,ROUTING.md,NUDGE.md,metadata.yaml,README.md} 2>/dev/null || true

# Verify Brainstack policies are loaded (via brainstack_recall or brainstack_inspect)
# Look for: skill_routing_policy:SKILL_NAME and skill_behavior_policy:SKILL_NAME
```

**See also:** `references/skill-installation-procedure.md` — Complete skill installation procedure, Brainstack policy import steps, and common failure modes.

## Official Source Verification

Before proposing install or upgrade commands, verify whether official upstream instructions exist.

Acceptable primary sources:

- official developer documentation
- official GitHub/GitLab repository README or docs directory
- official release notes or changelog
- official install script documentation
- official package registry page only when linked by the project

Secondary sources may be used only when primary sources are absent or incomplete:

- maintainer comments in issues or discussions
- distribution package notes
- community guides
- Stack Overflow or forum posts

Secondary sources must be labeled as secondary. They must not override official documentation.

If no official source can be verified, the agent must say so and classify the plan as inferred.

## Priority Rules

### 1. Skill Supremacy for System Mutation

**Precedence:** Applies as soon as the user request involves installation, upgrade, repair, rollback, reinstall, migration, dependency setup, service setup, package manager use, or other system mutation.

**Rule:** This skill takes priority over general assistant behavior. The agent must follow this skill's Phase 1 and Phase 2 protocol before producing any mutating command.

**Fallback:** If activation is uncertain, assume the skill applies when the task could change packages, files, services, containers, runtimes, dependencies, or configuration. Proceed with Phase 1 only.

**Justification:** System mutation requires stricter controls than ordinary assistance. General helpfulness can produce unsafe shortcuts.

### 2. Pre-Execution Guard

**Precedence:** Applies immediately before any command output that could install, upgrade, remove, overwrite, restart, migrate, or otherwise mutate system state.

**Rule:** If this skill has not been applied, or Phase 1 has not been completed and explicitly confirmed, block mutating command generation and redirect to Phase 1.

**Fallback:** If confirmation status is unclear, treat confirmation as absent and provide only read-only inspection commands.

**Justification:** A skill that can be bypassed does not enforce operational safety.

### 3. Mandatory Phase 1 Shape

**Precedence:** Applies to the first substantive response after activation.

**Rule:** Phase 1 must include exactly these sections: Intent, Environment, Current State, Official Source Basis, Target State, Risk / Rollback, Verification Plan, Proposed Plan, and STOP. Sections may contain `unknown` when information is not yet verified, but they must not be omitted.

**Fallback:** If required information is unavailable, mark it unknown and provide the smallest set of read-only inspection steps needed to resolve it.

**Justification:** A fixed shape makes omissions visible and prevents compressed summaries from hiding missing safety checks.

### 4. Output Rejection Criteria

**Precedence:** Applies to all generated responses while this skill is active.

**Rule:** Reject and revise any response that contains mutating commands before confirmation, omits official source status, omits rollback/backup status for risky changes, omits current-state verification, or lacks deterministic verification checks.

**Fallback:** When rejection criteria are met, replace the response with a Phase 1 correction and read-only inspection steps only.

**Justification:** Explicit invalid-output criteria make non-compliance detectable instead of subjective.

### 4b. Verify Repository Targets Before Planning

**Precedence:** Applies before proposing any Git-based installation, PR, or repository modification.

**Rule:** Never assume a repository exists, is the correct target, or has a specific structure. Verify via `gh repo view`, `git remote -v`, or filesystem inspection before presenting plans. Never plan upstream PRs without explicit user permission — you may suggest, but user must approve.

**Fallback:** If repository status cannot be verified, mark as unknown in Phase 1 and provide inspection commands only.

**Justification:** Assuming repository targets without verification wastes user time and produces incorrect plans. This is a core behavioral expectation, not a preference.

### 5. Official Instructions Override Assumptions

**Precedence:** Applies before any install, upgrade, repair, rollback, or migration plan is produced.

**Rule:** If official installation or upgrade instructions exist, use them as the primary authority. Do not invent commands from package-manager conventions, prior memory, or common patterns.

**Fallback:** If no official source can be verified, state that explicitly, classify the plan as inferred, and restrict Phase 2 to reversible, low-risk steps unless the user confirms the risk.

**Justification:** Installation workflows are mutation-heavy. Guessing creates version drift, broken environments, and unreproducible repairs.

### 6. Environment Classification Required

**Precedence:** Applies before planning any mutating action.

**Rule:** Classify the environment as local dev, VM, container, server, production, or unknown-sensitive. Identify OS/distribution and privilege level when relevant.

**Fallback:** If classification is incomplete, default to unknown-sensitive and provide only read-only inspection commands until the user confirms.

**Justification:** Install and upgrade strategies differ by environment. Misclassification can damage the wrong system or apply the wrong package strategy.

### 7. Current State Before Mutation

**Precedence:** Applies before any command that installs, upgrades, removes, overwrites, restarts, or migrates software.

**Rule:** Determine what is already installed, how it was installed, where it lives, what version it is, and whether a service/process is using it.

**Fallback:** If state cannot be determined, stop after Phase 1 and ask the user to run specific read-only inspection commands.

**Justification:** Mutating an unknown state can produce unrecoverable or difficult-to-debug failures.

### 8. Installation Method Consistency

**Precedence:** Applies after official source verification and current state discovery.

**Rule:** Use one installation method unless official instructions explicitly require multiple methods. Do not mix source checkout, package manager, pip, npm, Docker, and manual binary installation casually.

**Fallback:** If multiple methods are already present, stop and propose cleanup, normalization, or a conservative repair plan before proceeding.

**Justification:** Mixed installation methods cause path ambiguity, duplicate versions, dependency conflicts, and confusing rollback behavior.

### 9. Version Target Explicitness

**Precedence:** Applies before upgrade or rollback planning.

**Rule:** Identify the current version, target version, and reason for the change.

**Fallback:** If the target version is unspecified, default only to the latest stable version verified from official sources. If latest stable cannot be verified, require user confirmation before proceeding.

**Justification:** Unbounded upgrades can introduce breaking changes and make rollback unclear.

### 10. Pre-Mutation State Capture

**Precedence:** Applies immediately before any mutating action.

**Rule:** Ensure at least one suitable recovery point exists: VM snapshot, container image checkpoint, config backup, data backup, package rollback path, binary copy, or repository commit/tag.

**Fallback:** If no backup or rollback mechanism is available, warn clearly and require explicit user acknowledgment before Phase 2 commands.

**Justification:** Rollback requires a captured recovery point to be reliable.

### 11. Atomic Execution Steps

**Precedence:** Applies during Phase 2 command generation.

**Rule:** Commands must be minimal, ordered, independently explainable, and grouped into logical phases: inspect, backup, install/upgrade, configure, restart, verify.

**Fallback:** If a workflow cannot be made atomic, split it into smaller phases with checkpoints and stop points.

**Justification:** Atomic steps improve debuggability and reduce cascading failure.

### 12. Deterministic Verification

**Precedence:** Applies after execution planning and after every mutating phase.

**Rule:** Define explicit verification checks before execution: version output, service status, endpoint/CLI smoke test, and log sanity check where applicable.

**Fallback:** If no official verification method exists, state that and propose a minimal smoke test.

**Justification:** Successful installation must be proven, not assumed.

### 13. Drift Detection Before Upgrade

**Precedence:** Applies before upgrades, reinstalls, migrations, or destructive repairs.

**Rule:** Detect local modifications, config divergence, untracked files, custom patches, changed service units, or nonstandard install paths when relevant.

**Fallback:** If drift cannot be determined, warn and recommend inspection before proceeding.

**Justification:** Upgrades on drifted systems often fail or overwrite local customizations.

### 14. Destructive Action Transparency

**Precedence:** Applies before any action that removes, overwrites, replaces, restarts, disables, migrates, or deletes components.

**Rule:** Explicitly state what will be affected and whether data, config, service availability, or local changes may be impacted.

**Fallback:** If impact is unclear, pause at Phase 1 and require confirmation before generating commands.

**Justification:** Silent destructive changes break trust and systems.

### 15. Change Summary Emission

**Precedence:** Applies after execution completes or fails.

**Rule:** Produce a summary containing actions performed, versions before/after, files/paths affected, service changes, verification results, and rollback instructions.

**Fallback:** If execution was partial or failed, the summary must clearly describe incomplete state and next safe diagnostic steps.

**Justification:** Reproducibility and debugging depend on recorded changes.

### 16. Uncertainty Declaration

**Precedence:** Applies whenever required information is missing, conflicting, stale, or unverifiable.

**Rule:** State uncertainty explicitly. Do not produce mutating commands while critical uncertainty remains.

**Fallback:** Limit output to read-only inspection commands and a concise list of what must be verified next.

**Justification:** Guessing in system operations is failure with confidence.

## Decision Table

| Situation | Required behavior |
|---|---|
| Skill should apply but was not loaded | Treat output as invalid; redirect to Phase 1 |
| User asks to install a tool | Verify official install docs and environment before commands |
| User asks to upgrade | Verify current version, target version, release notes, backup, rollback |
| User says "use latest" | Verify latest stable from official source |
| User says "fix it" | Diagnose current state first; do not mutate blindly |
| Current install method is unknown | Provide inspection commands only |
| Multiple install methods are detected | Stop and propose normalization or cleanup plan |
| Phase 1 structure is incomplete | Stop and revise Phase 1 before continuing |
| Commands appear before confirmation | Reject output and replace with Phase 1 only |
| Official docs conflict with current install | Report conflict and ask for confirmation before mutation |
| No rollback exists | Warn and require explicit acknowledgment before Phase 2 |
| Production or unknown-sensitive environment | Avoid mutation until environment and backup are confirmed |
| Verification method is missing | Propose minimal smoke test and label it as inferred |
| **Skill package installation requested** | Complete all 5 steps: extract files, import ROUTING.md policy, import NUDGE.md policy, verify Brainstack records, test skill loading |
| **GitHub skill submission** | Skills go at repo root (`<skill-name>/`), NOT in category subdirs. Use org git identity (`forgedcontextlabs`), never personal names |

## Common Pitfalls

1. **Skipping Phase 1 for "simple" installs** — Even trivial-looking installations (e.g., "extract this zip") are mutations. Phase 1 is mandatory. Assumptions about what counts as "real" installation cause protocol bypasses.

2. **Incomplete skill package installation** — Installing a skill from a 5-file package (SKILL.md, ROUTING.md, NUDGE.md, metadata.yaml, README.md) requires:
   - Extracting files to `~/.hermes/skills/<skill-name>/`
   - Importing `ROUTING.md` as Brainstack `canonical_policy` on `operating` shelf with `stable_key: skill_routing_policy:<skill-name>`
   - Importing `NUDGE.md` as Brainstack `canonical_policy` on `operating` shelf with `stable_key: skill_behavior_policy:<skill-name>`
   - Skipping the policy imports means the skill won't auto-load on future requests.

3. **Wrong GitHub repo structure for skills** — Skills in `forgedcontextlabs/skills` live at repo root (`<skill-name>/SKILL.md`), NOT in category subdirectories (`software-development/<skill-name>/`). Category folders are for local organization only.

4. **Personal git identity on org repos** — All commits to `forgedcontextlabs/*` repos must use org identity: `forgedcontextlabs <forgedcontextlabs@users.noreply.github.com>`. Never use personal names (`Jack Isenberg`) or system users (`hrmsadmin`).

5. **Assuming skill exists without checking** — Before planning installation, verify whether the skill is already installed with `skill_view(name='<skill-name>')` or `ls ~/.hermes/skills/`.

6. **Executing unzip without confirmation** — Archive extraction is a filesystem mutation. It belongs in Phase 2 after Phase 1 completion and user confirmation.

## Output Templates

### Phase 1 Template

```text
## Phase 1 — Understanding Proof

Intent:
- ...

Environment:
- ...

Current State:
- ...

Official Source Basis:
- ...

Target State:
- ...

Risk / Rollback:
- ...

Verification Plan:
- ...

Proposed Plan:
1. ...
2. ...
3. ...

STOP: Please confirm before I generate or proceed with mutating commands.
```

### Phase 2 Template

```text
## Phase 2 — Execution Plan

Preflight:
```bash
...
```

Backup / Snapshot:
```bash
...
```

Install / Upgrade / Repair:
```bash
...
```

Configure / Restart:
```bash
...
```

Verify:
```bash
...
```

Rollback:
```bash
...
```

Change Summary:
- Before:
- After:
- Paths affected:
- Services affected:
- Verification result:
- Rollback path:
```

## Correct and Incorrect Behavior

### Correct

User: "Upgrade Hermes."

Agent:

```text
I need to verify the current Hermes install, install method, version, environment, official upgrade instructions, and rollback path first.
Run these read-only checks...
```

### Incorrect

User: "Upgrade Hermes."

Agent:

```bash
git pull && pip install -U hermes
```

Why incorrect:
- assumes install method
- skips official docs
- skips current version
- skips backup
- skips confirmation
- skips verification

## Embedded Policies (Fallback Mode)

Use this section when external routing or behavior policy records are unavailable.

### Embedded Routing Policy

Activate this skill for requests involving:

- install
- upgrade
- update
- rollback
- repair
- reinstall
- migrate
- setup
- dependency installation
- package manager usage
- service installation
- systemd setup
- Docker or container setup
- version mismatch
- official installation instructions
- GitHub installation instructions
- broken Hermes, Hex, or related component
- VM or development environment setup
- **skill package installation** (5-file bundles: SKILL.md, ROUTING.md, NUDGE.md, metadata.yaml, README.md)
- **Brainstack policy imports** (routing/behavior policies for skills)

Do not activate this skill for:

- pure architecture discussion with no installation or mutation path
- general software recommendations
- non-mutating code edits
- conceptual explanations without setup, upgrade, or repair intent

### Embedded Behavior Nudge

When this skill is active, behave like a cautious installation operator. This skill overrides general assistant behavior for system-mutation tasks.

Prefer:

- skill protocol over general helpfulness
- official instructions over assumptions
- read-only inspection before mutation
- explicit current/target version handling
- environment classification
- rollback planning
- atomic steps
- deterministic verification
- concise uncertainty statements

Avoid:

- guessing install commands
- mixing installation methods
- using latest without verification
- mutating unknown environments
- skipping backups
- skipping confirmation
- declaring success without checks
- bypassing the skill when install or upgrade intent is present
- treating routing or nudge files as required for correctness

## Portability Notes

This skill must work with or without Brainstack.

- `SKILL.md` contains correctness-critical behavior and fallback policies.
- `ROUTING.md` may improve activation but must not be required for correctness.
- `NUDGE.md` may improve execution behavior but must not be required for correctness.
- Provider-specific memory systems should preserve policy intent rather than inventing new behavior.

Do not rely on hidden provider features, undocumented Hermes internals, or environment variables unless explicitly provided by the user or verified from the runtime environment.
