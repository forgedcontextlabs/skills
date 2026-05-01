# Routing Policy: hermes-upgrade-installation-operator

TYPE: agent_routing_policy
SCOPE: skill_activation
SKILL: hermes-upgrade-installation-operator

Auto-load this skill when the user request involves installation, upgrade, repair, rollback, reinstall, migration, dependency setup, service setup, package manager use, official install instructions, or verification of installed system components.

Activation is mandatory for matching tasks. If a response would generate mutating commands for a matching task without this skill loaded and applied, the response is invalid and must redirect to Phase 1 of this skill.

## Strong triggers

- install
- installation
- setup
- upgrade
- update
- reinstall
- repair
- fix install
- rollback
- downgrade
- migrate
- migration
- dependency
- package manager
- apt
- dnf
- yum
- pacman
- brew
- pip
- pipx
- npm
- pnpm
- yarn
- docker
- docker compose
- systemd
- service file
- daemon
- version mismatch
- latest version
- release notes
- changelog
- official docs
- GitHub README
- install instructions
- upgrade instructions
- broken install
- verify install
- smoke test

## Contextual triggers

- user is setting up Hermes, Hex, a Hermes skill, or a related local agent component
- user asks whether an installed component matches official documentation
- user asks what is already installed before changing it
- user asks to compare current version to latest stable
- user asks to repair a toolchain, service, VM, container, or development environment
- user is about to run commands that may mutate files, packages, services, configs, or containers
- user references a developer site, GitHub repository, package registry, release note, or install script

## Do not trigger for

- general conceptual explanation of software installation with no action path
- software recommendations without setup, install, or upgrade intent
- code review unrelated to deployment, package management, or runtime setup
- pure architecture discussion unless it includes installation, migration, or repair steps
- fictional or creative writing involving software setup

## Action

If triggered, call:

```text
skill_view(name="hermes-upgrade-installation-operator")
```

Then follow the skill protocol.

If the skill cannot be loaded but the request matches this routing policy, enforce the embedded fallback policy in SKILL.md.

## Important

Before generating mutating commands, verify current state, official upstream instructions, target version or desired state, environment classification, rollback path, and verification plan.

Read-only inspection may happen in Phase 1.

Mutating commands must wait until after the user confirms the Phase 1 understanding proof.

First response after activation must include the Phase 1 sections: Intent, Environment, Current State, Official Source Basis, Target State, Risk / Rollback, Verification Plan, Proposed Plan, and STOP.
