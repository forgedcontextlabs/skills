# Hermes Agent Skills

Reusable procedural memory for [Hermes Agent](https://hermes-agent.nousresearch.com/). Think of each skill as a cheat sheet the agent reads when it needs to do something specific — no code to run, no packages to install, just knowledge.

## What Even Is a Skill?

A skill is a markdown file. When the agent encounters a situation the skill covers, it loads that file into its memory and follows the instructions inside. That's it.

This repo contains community skills you can install into your agent in seconds.

## Quick Start (For the Impatient)

```bash
# 1. Add this repo as a source (one time)
hermes skills tap add forgedcontextlabs/skills

# 2. Install a skill
hermes skills install github/forgedcontextlabs/skills temporal-awareness

# 3. Done. The agent now knows how to handle time properly.
```

Want to peek before installing?
```bash
hermes skills inspect github/forgedcontextlabs/skills temporal-awareness
```

---

## Featured Skill: Temporal Awareness

**The Problem:** AI agents are bad at time. They say things like "I just did that" or "it happened recently" without actually checking what time it is. Over a long conversation, this drifts into nonsense.

**What This Skill Does:** It forces the agent to check the system clock before making any time-related statement. No more guessing.

### What You Get

- **Actual timestamps** instead of vague words like "soon" or "recently"
- **Automatic timezone detection** — the agent figures out your local time instead of assuming UTC
- **Date math** — "in 2 hours" becomes an exact timestamp
- **Log reading** — parses timestamps from system logs correctly, even across timezones
- **Clock sync checking** — detects if your servers have drifted out of sync
- **Customizable output** — pick how you want times formatted (see below)

### Example: Before vs After

| Without This Skill | With This Skill |
|---|---|
| "I just did that." | "Completed at 15:28 EDT (2 minutes ago)." |
| "The backup ran this morning." | "The backup ran at 06:00 EDT today." |
| "It will expire soon." | "It expires at 16:00 EDT (in 28 minutes)." |
| "Server 2 is behind." | "Server 2 clock: 15:30:05 EDT. Server 1 clock: 15:32:18 EDT. Drift: 2m 13s." |

### Why You Care

- **Logs from multiple servers** are impossible to compare if the clocks don't match
- **DST (daylight saving)** breaks cron jobs twice a year unless you handle it explicitly
- **"Soon" is not a timestamp** — precision matters when debugging outages

---

## Configuration (Optional)

By default, the agent uses a verbose format with full timezone info. If you prefer something else, set it once and forget it.

### The Easy Way: Environment Variable

Add this to `~/.hermes/.env`:

```bash
# Pick one: verbose, iso, compact, human12, human24, utc, epoch, rfc3339
TEMPORAL_FORMAT=iso
```

Then restart your agent. That's it.

### What Each Format Looks Like

| Format | Example Output | When to Use |
|--------|---------------|-------------|
| **verbose** | `2026-04-23 15:32:18 EDT (-04:00)` | Debugging, when you need everything |
| **iso** | `2026-04-23T15:32:18-04:00` | APIs, filenames, sorting |
| **compact** | `20260423_153218` | File backups, no spaces allowed |
| **human12** | `Apr 23 03:32 PM EDT` | Talking to humans |
| **human24** | `2026-04-23 15:32 EDT` | Talking to humans (24h clock) |
| **utc** | `2026-04-23 19:32:18 UTC` | Comparing logs from different timezones |
| **epoch** | `1776973684` | Databases, precise comparison |
| **rfc3339** | `2026-04-23T15:32:18-04:00` | Web APIs, JSON |

### The Fallback Chain

If you don't set anything, the agent tries:
1. `TEMPORAL_FORMAT` env var
2. Your `~/.hermes/memories/USER.md` preferences
3. Default to `verbose`

Full technical details are in the [skill file itself](devops/temporal-awareness/SKILL.md).

---

## Safety

Skills are markdown files, not programs. They can't install malware or modify your system. **However**, the agent may run bash commands that are *inside* the skill, so treat them like you would a shell script from the internet — read before you trust.

Every skill in this repo is:
- Linted for structure
- Scrubbed for personal info (no hardcoded hostnames, usernames, or secrets)
- Tested on AlmaLinux 10.1
- Verified to use only standard system tools

See [SECURITY.md](SECURITY.md) for the full verification process.

---

## Install Notes

- This repo is available immediately via `hermes skills tap add` (local registration)
- Global search (`hermes skills search`) indexes periodically — usually 24–48 hours for new repos

---

## Contributing

Got a skill idea? Keep it simple:

1. Write a `SKILL.md` file with this at the top:
   ```yaml
   ---
   name: your-skill-name
   description: What it does and when to use it.
   category: devops
   ---
   ```
2. Put it under `<category>/<skill-name>/SKILL.md`
3. No personal info, no hardcoded paths
4. Test every command
5. Open a PR

## Repo Structure

```
.
├── README.md           ← You are here
├── SECURITY.md         ← Safety and verification info
├── LICENSE             ← MIT
├── devops/
│   └── temporal-awareness/
│       └── SKILL.md    ← The actual skill content
└── <category>/
    └── <skill-name>/
        └── SKILL.md
```

Skills can also include optional folders:
- `references/` — docs, API specs
- `templates/` — reusable config files
- `scripts/` — helper scripts (must be documented)
- `assets/` — images, diagrams

## License

MIT