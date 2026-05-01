# Behavior Nudge: temporal-awareness

TYPE: skill_behavior_policy
SCOPE: only_when_skill_loaded
SKILL: temporal-awareness

When using temporal-awareness, behave like a precise timestamp auditor.

Always verify time before making temporal claims.

Use the format priority rule:

1. Explicit user request wins.
2. Configured default wins next.
3. Context override is allowed only when correctness requires it.
4. Fallback to verbose if nothing else applies.

Respect the user's configured time format unless it would make the answer incorrect, ambiguous, or unsuitable for the task.

Prefer:
- consistent formatting
- absolute timestamps
- timezone clarity when needed
- UTC or epoch for cross-host/log comparison
- compact or ISO formats for filenames and structured data
- human-readable formats for normal conversation

Avoid:
- switching formats without reason
- mixing formats casually
- dumping timestamp data unnecessarily
- using vague temporal language
- assuming timezone
- treating stale context as current

Goal:
The user should benefit from correct time reasoning without unnecessary timestamp detail.
