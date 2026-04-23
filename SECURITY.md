# Security & Trust

## What a Skill Is

Hermes Agent skills are **plain markdown files** with YAML frontmatter. They contain:
- Procedural knowledge (when to use, what to check)
- Bash command examples (for the agent to run, not for you to run blindly)
- Reference tables and checklists

**A skill is NOT:**
- An executable binary
- A script that runs on install
- A package manager dependency

## What a Skill Can Do

When loaded into context, the agent may choose to execute bash commands shown in the skill. This is by design — the skill teaches the agent how to perform a task. This means:

- **Inspect before installing.** Read the `SKILL.md` first.
- **Trust the author.** Skills from unknown repos should be treated like shell scripts from the internet.

## How We Verify This Repo

Every skill in this repo is checked before commit:

1. **Lint** — YAML frontmatter validated, markdown structure checked
2. **Personal info scrub** — No hostnames, usernames, hardcoded paths, or API keys
3. **Command validation** — Every bash command tested on AlmaLinux 10.1
4. **Dependency check** — All required tools verified in PATH
5. **No network exfiltration** — Commands only use local system tools (`date`, `timedatectl`, etc.)

## How to Audit Before Installing

```bash
# Preview the skill without installing
hermes skills inspect github/forgedcontextlabs/skills <skill-name>

# Or read the raw file directly
curl -s https://raw.githubusercontent.com/forgedcontextlabs/skills/main/<category>/<skill-name>/SKILL.md
```

## Reporting Issues

If you find a command that is unsafe, incorrect, or contains personal information, open an issue or PR.
