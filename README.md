# Hermes Agent Skills

Reusable procedural memory for [Hermes Agent](https://hermes-agent.nousresearch.com/). Each skill is a markdown file loaded into context when the agent needs it — no dependencies, no install scripts, just structured knowledge.

## Install

```bash
# Add this repo as a skill source (instant, local)
hermes skills tap add forgedcontextlabs/skills

# Install any skill
hermes skills install github/forgedcontextlabs/skills <skill-name>

# Or preview before installing
hermes skills inspect github/forgedcontextlabs/skills <skill-name>
```

> **Safety:** Skills are markdown, not binaries, but the agent may execute commands inside them. Always inspect before installing. See [SECURITY.md](SECURITY.md) for our verification process.

> **Hub indexing:** This repo is available immediately via `tap add`. Global search (`hermes skills search`) indexes periodically — typically within 24–48 hours.

## Available Skills

### DevOps / Infrastructure

| Skill | Purpose | Dependencies |
|-------|---------|-------------|
| **temporal-awareness** | Verify system time before any temporal claim. Date math, timezone conversion, log timestamp parsing, multi-host clock sync. | `date`, `timedatectl` |

## Featured Skill: Temporal Awareness

**Problem:** AI agents hallucinate time. "I just did that" means nothing when the context window spans 12 hours. Relative terms rot immediately.

**Solution:** A hard rule — verify `date` before every temporal statement.

### What It Covers

- **Quick time checks** — `date +"%Y-%m-%d %H:%M:%S %Z (%:z)"`
- **Timezone auto-detection** — never assume UTC, never hardcode a zone
- **Date math** — convert "+2 hours", "-1 day", relative terms to absolute timestamps
- **Cross-timezone conversion** — `TZ="America/Los_Angeles" date ...`
- **Multi-host clock comparison** — detect drift across servers
- **Log timestamp parsing** — journalctl ranges, UTC vs local, epoch conversion
- **NTP/sync troubleshooting** — `chronyc tracking`, `timedatectl show-timesync`

### Example: Before and After

| Without Skill | With Skill |
|---------------|------------|
| "I just did that." | "Completed at 15:28 EDT (2 minutes ago)." |
| "The backup ran this morning." | "The backup ran at 06:00 EDT today (2026-04-23)." |
| "It will expire soon." | "It expires at 16:00 EDT (in 28 minutes)." |
| "Server 2 is behind." | "Server 2 clock: 15:30:05 EDT. Server 1 clock: 15:32:18 EDT. Drift: 2m 13s." |

### Why This Matters

- **Distributed systems** drift. Two servers 5 minutes apart cause log ordering nightmares.
- **DST transitions** break cron jobs and scheduled tasks twice a year.
- **Log correlation** across timezones is impossible without explicit conversion.
- **Agent credibility** depends on precision. "Soon" is not a timestamp.

## Contributing

1. Write a `SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: your-skill-name
   description: What it does and when to use it.
   category: devops
   ---
   ```
2. Place it under `<category>/<skill-name>/SKILL.md`
3. Keep it generic — no personal hostnames, usernames, or hardcoded paths
4. Test every bash command before submitting
5. Open a PR

## Structure

```
.
├── README.md
├── devops/
│   └── temporal-awareness/
│       └── SKILL.md
└── <category>/
    └── <skill-name>/
        └── SKILL.md
```

Skills may include optional subdirectories:
- `references/` — external docs, API specs
- `templates/` — reusable configs, YAML, JSON
- `scripts/` — helper scripts (must be executable and documented)
- `assets/` — images, diagrams

## License

MIT