# Routing Policy: system-eval-operator

TYPE: agent_routing_policy
SCOPE: skill_activation
SKILL: system-eval-operator

Auto-load this skill when the user is asking to evaluate, stress-test, validate, audit, review, or challenge a system, component, workflow, testing framework, generated test suite, or software artifact.

## Strong triggers

- evaluate this
- stress test this
- validate this
- audit this
- review the test suite
- check the testing framework
- does this test enough
- what are the gaps
- find weaknesses
- break this
- red team this
- false positives
- false negatives
- coverage gaps
- regression risk
- acceptance criteria
- failure signals
- adversarial eval
- agent eval
- LLM eval
- prompt injection
- memory poisoning
- instruction hierarchy
- context drift

## Contextual triggers

- user has already created a testing framework and now wants to assess its quality
- user wants to evaluate generated tests rather than design new ones
- user wants to pressure-test a system after implementation
- user asks whether a test plan, eval suite, UI, API, workflow, or agent behavior is sufficient
- user asks what could still go wrong after tests pass
- user asks to compare expected behavior against observed results

## Do not trigger for

- initial test design requests, unless the user is evaluating a proposed test design
- direct implementation requests with no evaluation intent
- ordinary debugging unless the user asks for systemic failure analysis
- creative writing
- non-software planning unless the user explicitly asks for evaluation

## Action

If a trigger is present, call:

```text
skill_view(name="system-eval-operator")
```

Then follow the skill protocol exactly.

## Important

The skill must complete Phase 1 Understanding Proof before generating evaluations.
Do not skip the confirmation step.
Do not generate evaluations prematurely.
Do not assume the target is Hermes unless the user explicitly says so.
