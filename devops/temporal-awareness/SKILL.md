---
name: temporal-awareness
description: Verify current time, date, and timezone before any temporal claim. Convert relative time to absolute timestamps. Handle cross-host timezone coordination and log timestamp parsing.
category: devops
---

# Temporal Awareness

## Rule
**Before every temporal statement** — "now", "recently", "this morning", "in 2 hours", "yesterday", "due soon", "expired", "last week" — you MUST verify the actual system time.

Context windows span sessions. A turn written at 03:00 and read at 15:00 is 12 hours stale. Relative terms rot immediately.

## Quick Check
```bash
date +"%Y-%m-%d %H:%M:%S %Z (%:z)"
```
Example output: `2026-04-23 15:32:18 EDT (-04:00)`

Use `timedatectl` when you also need NTP sync status or DST details:
```bash
timedatectl status
```

## Timezone: Never Assume

### Detect Local Timezone
```bash
# System timezone
timedatectl show --property=Timezone --value
# or
cat /etc/timezone 2>/dev/null || ls -l /etc/localtime

# For the user, also consider:
date +%Z   # Short code (EDT, PST, UTC)
date +%:z  # Numeric offset (-04:00)
```

### Common Timezones
| Code | Offset | Location |
|------|--------|----------|
| UTC | +00:00 | Baseline |
| EST | -05:00 | US East (winter) |
| EDT | -04:00 | US East (summer) |
| CST/CDT | -06:00/-05:00 | US Central |
| MST/MDT | -07:00/-06:00 | US Mountain |
| PST/PDT | -08:00/-07:00 | US Pacific |
| GMT/BST | +00:00/+01:00 | UK |
| CET/CEST | +01:00/+02:00 | Central Europe |
| JST | +09:00 | Japan |
| AEDT/AEST | +11:00/+10:00 | Australia East |

**Always note whether DST is active** when specifying a timezone.

## Date Math

```bash
# Relative to absolute
now="$(date +"%Y-%m-%d %H:%M:%S %Z")"
future="$(date -d "+2 hours" +"%Y-%m-%d %H:%M:%S %Z")"
past="$(date -d "-1 day" +"%Y-%m-%d %H:%M:%S %Z")"

# Convert between timezones
date -d "2026-04-23 15:00 EDT" +"%Y-%m-%d %H:%M:%S %Z" --utc
TZ="America/Los_Angeles" date -d "2026-04-23 15:00 EDT"

# Epoch timestamps for precise comparison
date +%s              # now as epoch
date -d "2026-04-23 15:00:00 EDT" +%s
```

## Convert Relatives to Absolutes

| Relative | Absolute Form |
|----------|---------------|
| "now" | "15:32:18 EDT on 2026-04-23" |
| "this morning" | "09:00–12:00 EDT on 2026-04-23" |
| "soon" | "within 30 minutes of 15:32 EDT" |
| "recently" | "in the last 4 hours (since 11:32 EDT)" |
| "yesterday" | "2026-04-22" |
| "last week" | "2026-04-16 through 2026-04-22" |
| "in 2 hours" | "17:32 EDT on 2026-04-23" |
| "the other day" | **Banned.** Pick a date. |

## Multi-Host Time Coordination

When managing multiple servers, compare clocks:

```bash
# On each host, run:
date +"%Y-%m-%d %H:%M:%S %Z (%:z) — %s"

# If SSH is available:
for host in server1 server2 server3; do
  echo -n "$host: "
  ssh "$host" 'date +"%Y-%m-%d %H:%M:%S %Z (%:z) — %s"' 2>/dev/null || echo "unreachable"
done
```

**All hosts should agree within a few seconds.** If drift exceeds 30 seconds, investigate NTP.

## Cron & Job Scheduling

When evaluating schedules:
```bash
# List active timers
systemctl list-timers --all

# View a specific cron
crontab -l
sudo cat /etc/cron.d/*

# Next run time of a timer
systemctl status <timer-name>
```

Always verify: **Does the cron use system time or a hardcoded timezone?**

## Log Timestamp Parsing

Logs may use UTC, local time, or no timezone at all.

```bash
# journalctl with explicit range (local time assumed)
sudo journalctl --since "2026-04-23 11:00" --until "2026-04-23 15:00"

# If the log is in UTC, convert:
date -d "2026-04-23 19:00 UTC" +"%Y-%m-%d %H:%M:%S %Z"

# Sort logs from multiple hosts by UTC epoch
cat host1.log host2.log | awk '{print $1, $2, $3}' | sort
```

**Rule of thumb:** When comparing events across systems, convert everything to UTC or epoch.

## NTP & Time Sync

If system time seems wrong or multi-host comparison shows drift:
```bash
# Check sync status
timedatectl status | grep "NTP\|synchronized"

# If chronyd is running:
chronyc tracking
chronyc sources

# If systemd-timesyncd is running:
timedatectl show-timesync --all

# Force immediate sync (if configured):
sudo chronyc makestep
```

**Drift tolerance for distributed systems:** < 1 second. For casual logging: < 5 minutes.

## Checklist Before Temporal Claims

- [ ] Run `date` and note the exact output
- [ ] Note the timezone and whether DST is active
- [ ] If referring to a past event, calculate the exact elapsed time
- [ ] If scheduling a future event, calculate the exact target timestamp
- [ ] If comparing logs or events across hosts, convert to UTC/epoch
- [ ] Never use relative terms without an absolute anchor

## Examples

| Wrong | Right |
|-------|-------|
| "I just did that." | "Completed at 15:28 EDT (2 minutes ago)." |
| "The backup ran this morning." | "The backup ran at 06:00 EDT today (2026-04-23)." |
| "It will expire soon." | "It expires at 16:00 EDT (in 28 minutes)." |
| "The meeting is tomorrow." | "The meeting is 2026-04-24 at 10:00 EDT." |
| "Server 2 is behind." | "Server 2 clock: 15:30:05 EDT. Server 1 clock: 15:32:18 EDT. Drift: 2m 13s." |
| "Logs show an error around then." | "Error at 2026-04-23 15:15:32 EDT (epoch 1745440532)." |