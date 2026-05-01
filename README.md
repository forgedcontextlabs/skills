# 🧠 Hermes Skills

**Deterministic skill modules for Hermes Agent**

------------------------------------------------------------------------

## ⚡ TL;DR

-   Hermes runs on **skills**, not prompts
-   Skills enforce **explicit rules and execution flow**
-   Every task follows:
    1.  **Understand (STOP)**
    2.  **Execute**
-   No assumptions
-   No silent failures

------------------------------------------------------------------------

## What This Is

This repo contains **skills used by Hermes Agent**.

A skill is a **strict control module** that defines:

-   when it activates
-   how decisions are made
-   what rules must be followed
-   how failure is handled

When a skill is active, it **overrides default behavior**.

------------------------------------------------------------------------

## How It Works

    Task
     ↓
    Skill Selected
     ↓
    SKILL.md Loaded
     ↓
    Phase 1: Understanding (STOP)
     ↓
    Phase 2: Execution
     ↓
    NUDGE (if failure)

------------------------------------------------------------------------

## Structure

    skills/
    ├── <skill-name>/
    │   ├── SKILL.md
    │   ├── ROUTING.md
    │   ├── NUDGE.md
    │   ├── metadata.yaml
    │   └── resources/

------------------------------------------------------------------------

## Execution Model

All skills must:

-   Require **Phase 1 understanding before execution**
-   Use **priority-based rules**
-   **Disallow implicit assumptions**
-   Define **failure detection and recovery**

------------------------------------------------------------------------

## Example Domains

-   Memory hygiene
-   Temporal reasoning
-   Test design

------------------------------------------------------------------------

## Creating a Skill

Required:

-   `SKILL.md`
-   `metadata.yaml`

Recommended:

-   `ROUTING.md`
-   `NUDGE.md`

Minimum structure:

``` yaml
name: example-skill
description: What it enforces
```

``` md
## Decision Rules (Priority Ordered)
1. Rule
2. Rule

## Execution
Phase 1 → Understanding (STOP)  
Phase 2 → Execution  

## Failure Handling
- Condition → Action
```

------------------------------------------------------------------------

## Notes

-   Skills are **constraints**, not suggestions
-   If a rule conflicts with default behavior, **the rule wins**
-   If understanding is unclear, **execution must not proceed**

------------------------------------------------------------------------

## License

MIT License

Copyright (c) 2026 Forged Context Labs

Permission is hereby granted, free of charge, to any person obtaining a
copy
of this software and associated documentation files (the "Software"), to
deal
in the Software without restriction, including without limitation the
rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or
sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included
in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
IN THE
SOFTWARE.
