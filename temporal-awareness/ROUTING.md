# Routing Policy: temporal-awareness

TYPE: agent_routing_policy
SCOPE: skill_activation
SKILL: temporal-awareness

Auto-load this skill when the user request depends on current time, relative time, scheduling, timestamps, logs, expiration, deadlines, timezone handling, or multi-host clock coordination.

## Strong triggers

- now
- today
- yesterday
- tomorrow
- recently
- soon
- this morning
- last week
- in X minutes
- in X hours
- expired
- due
- stale
- deadline
- schedule
- cron
- timer
- logs
- timestamp
- timezone
- UTC
- epoch
- NTP
- clock drift
- when did this happen
- how long ago
- what time

## Contextual triggers

- user is debugging logs or service behavior
- user is coordinating events across hosts, VMs, servers, containers, or timezones
- user asks whether something has expired, is late, is due, or is stale
- user asks to schedule, compare, parse, or normalize times
- user references prior conversation timing that may now be stale
- user is doing DevOps work where time accuracy matters

## Do not trigger for

- purely historical dates that do not require current time
- fictional or creative writing involving time
- static date formatting examples with no real temporal claim
- conceptual explanations of timezones unless current time matters

## Action

If triggered, call:

```text
skill_view(name="temporal-awareness")
```

Then follow the skill protocol.

## Important

Before making a temporal claim, verify current system time using the appropriate command or trusted time source.
Convert relative terms to absolute timestamps.
Include timezone and UTC offset when precision matters.
Do not rely on stale conversation context for current time.
