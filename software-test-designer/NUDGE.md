# Behavior Nudge: software-test-designer

TYPE: skill_behavior_policy
SCOPE: only_when_skill_loaded
SKILL: software-test-designer

When using software-test-designer, prioritize disciplined test design over fast output.

Before generating tests, prove that you understand:
- what kind of system is being tested
- what can fail
- which failures matter most
- what level of thoroughness is appropriate
- what assumptions you are making

Choose test depth deliberately.

Use the thoroughness priority rule:

1. Explicit user request wins.
2. Known risk increases depth.
3. Change scope determines baseline depth.
4. Known failures require regression coverage.
5. If uncertain, propose L1-L3 and state assumptions.

Prefer:
- shallow tests for low-risk changes
- deeper tests for persistent, stateful, agentic, security-sensitive, or user-facing systems
- regression tests when prior failures exist
- concrete, falsifiable pass criteria

Avoid:
- generating every test level just because levels exist
- vague tests like "make sure it works"
- vague tests like "check the UI"
- vague tests like "test memory"
- over-testing trivial changes
- under-testing risky stateful systems

Every test should specify:
- exact input
- expected behavior
- forbidden behavior, when relevant
- observable failure signals
- artifacts to capture
- cleanup steps

Every selected level should have a reason.
Every skipped level should be justifiable.

If the user's request is underspecified, make reasonable assumptions, state them clearly, and stop for confirmation before generating tests.
