# Routing Policy: software-test-designer

TYPE: agent_routing_policy
SCOPE: skill_activation
SKILL: software-test-designer

Auto-load this skill when the user is asking for software testing, validation, QA planning, or failure-mode analysis.

## Strong triggers

- design tests
- write tests
- test strategy
- test plan
- QA
- validation plan
- verify this
- regression
- test coverage
- edge cases
- failure modes
- risk surface
- what could break
- pytest
- Playwright
- e2e test
- integration test
- smoke test
- adversarial test

## Contextual triggers

- user is building or modifying software and asks how to ensure correctness
- user is designing a system and asks about reliability or safety
- user is working on Hermes Agent, memory, tools, web UI, API, CLI, orchestration, or deployment and asks about validation
- user wants multiple levels of test thoroughness

## Do not trigger for

- ordinary debugging unless the user asks how to prevent recurrence
- direct implementation requests with no testing or validation angle
- purely conceptual brainstorming
- creative writing
- non-software planning
- requests where "test" means an exam, quiz, or personality test

## Action

If a strong or contextual trigger is present, call:

```text
skill_view(name="software-test-designer")
```

Then follow the skill protocol exactly.

## Important

The skill must first produce its Phase 1 Understanding Proof and stop for user confirmation before generating tests.

Do not silently skip the skill's confirmation gate.
Do not generate tests before the user confirms.
