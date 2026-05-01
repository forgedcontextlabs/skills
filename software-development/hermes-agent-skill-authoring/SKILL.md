---
name: hermes-agent-skill-authoring
description: "Author in-repo SKILL.md: frontmatter, validator, structure."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [skills, authoring, hermes-agent, conventions, skill-md]
    related_skills: [writing-plans, requesting-code-review]
---

# Authoring Hermes-Agent Skills (in-repo)

## Overview

There are two places a SKILL.md can live:

1. **User-local:** `~/.hermes/skills/<maybe-category>/<name>/SKILL.md` — personal, not shared. Created via `skill_manage(action='create')`.
2. **In-repo (this skill is about this case):** `/home/bb/hermes-agent/skills/<category>/<name>/SKILL.md` — committed, shipped with the package. Use `write_file` + `git add`. `skill_manage(action='create')` does NOT target this tree.

## When to Use

- User asks you to add a skill "in this branch / repo / commit"
- You're committing a reusable workflow that should ship with hermes-agent
- You're editing an existing skill under `/home/bb/hermes-agent/skills/` (use `patch` for small edits, `write_file` for rewrites; `skill_manage` still works for patch on in-repo skills, but not for `create`)

## Required Frontmatter

Source of truth: `tools/skill_manager_tool.py::_validate_frontmatter`. Hard requirements:

- Starts with `---` as the first bytes (no leading blank line).
- Closes with `\n---\n` before the body.
- Parses as a YAML mapping.
- `name` field present.
- `description` field present, ≤ **1024 chars** (`MAX_DESCRIPTION_LENGTH`).
- Non-empty body after the closing `---`.

Peer-matched shape used by every skill under `skills/software-development/`:

```yaml
---
name: my-skill-name               # lowercase, hyphens, ≤64 chars (MAX_NAME_LENGTH)
description: Use when <trigger>. <one-line behavior>.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [short, descriptive, tags]
    related_skills: [other-skill, another-skill]
---
```

`version` / `author` / `license` / `metadata` are NOT enforced by the validator, but every peer has them — omit and your skill sticks out.

## Size Limits

- Description: ≤ 1024 chars (enforced).
- Full SKILL.md: ≤ 100,000 chars (enforced as `MAX_SKILL_CONTENT_CHARS`, ~36k tokens).
- Peer skills in `software-development/` sit at **8-14k chars**. Aim for that range. If you're pushing past 20k, split into `references/*.md` and reference them from SKILL.md.

## Peer-Matched Structure

Every in-repo skill follows roughly:

```
# <Title>

## Overview
One or two paragraphs: what and why.

## When to Use
- Bulleted triggers
- "Don't use for:" counter-triggers

## <Topic sections specific to the skill>
- Quick-reference tables are common
- Code blocks with exact commands
- Hermes-specific recipes (tests via scripts/run_tests.sh, ui-tui paths, etc.)

## Common Pitfalls
Numbered list of mistakes and their fixes.

## Verification Checklist
- [ ] Checkbox list of post-action verifications

## One-Shot Recipes (optional)
Named scenarios → concrete command sequences.
```

Not every section is mandatory, but `Overview` + `When to Use` + actionable body + pitfalls are the minimum for the skill to feel like a peer.

## Directory Placement

```
skills/<category>/<skill-name>/SKILL.md
```

Categories currently in repo (confirm with `ls skills/`): `autonomous-ai-agents`, `creative`, `data-science`, `devops`, `dogfood`, `email`, `gaming`, `github`, `leisure`, `mcp`, `media`, `mlops/*`, `note-taking`, `productivity`, `red-teaming`, `research`, `smart-home`, `social-media`, `software-development`.

Pick the closest existing category. Don't invent new top-level categories casually.

## Workflow

1. **Survey peers** in the target category:
   ```
   ls skills/<category>/
   ```
   Read 2-3 peer SKILL.md files to match tone and structure.
2. **Check validator constraints** in `tools/skill_manager_tool.py` if unsure.
3. **Draft** with `write_file` to `skills/<category>/<name>/SKILL.md`.
4. **Validate locally**:
   ```python
   import yaml, re, pathlib
   content = pathlib.Path("skills/<category>/<name>/SKILL.md").read_text()
   assert content.startswith("---")
   m = re.search(r'\n---\s*\n', content[3:])
   fm = yaml.safe_load(content[3:m.start()+3])
   assert "name" in fm and "description" in fm
   assert len(fm["description"]) <= 1024
   assert len(content) <= 100_000
   ```
5. **Git add + commit** on the active branch.
6. **Note:** the CURRENT session's skill loader is cached — `skill_view` / `skills_list` will not see the new skill until a new session. This is expected, not a bug.

## Cross-Referencing Other Skills

`metadata.hermes.related_skills` unions both trees (`skills/` in-repo and `~/.hermes/skills/`) at load time. You CAN reference a user-local skill from an in-repo skill, but it won't resolve for other users who clone the repo fresh. Prefer referencing only in-repo skills from in-repo skills. If a frequently-referenced skill lives only in `~/.hermes/skills/`, consider promoting it to the repo.

## Memory-Provider-Agnostic Skill Distribution

For skills intended for GitHub submission or cross-instance portability, embed behavioral policies directly in the SKILL.md file rather than relying solely on Brainstack (or other memory provider) policies.

### Pattern: Dual-Commitment Workflow

When creating a skill with auto-load behavior:

1. **Embed policies in SKILL.md** (for portability):
   ```markdown
   ## Auto-Load Triggers
   
   Load this skill when user input contains:
   - Strong triggers: [...]
   - Contextual triggers: [...]
   
   ## Usage Policy
   
   When this skill is loaded:
   - Behavioral standards
   - Format preferences
   - Confirmation gates
   ```

2. **Commit Brainstack policies** (for local enforcement):
   - `skill_auto_load:<skill-name>` — trigger conditions
   - `skill_usage:<skill-name>` — behavioral standards
   
   This ensures the current instance enforces the policy even before the skill file is consulted.

3. **GitHub submission**: The SKILL.md file is self-contained and memory-provider-agnostic. Brainstack policies are instance-specific runtime enhancements.

### Why This Matters

- **Portability**: SKILL.md ships with the repo and works regardless of memory backend (Brainstack, mem0, supermemory, none)
- **Self-documenting**: Any agent or human reading the skill knows when it should load and how it should behave
- **Separation of concerns**: Brainstack = local enforcement; SKILL.md = canonical source of truth

### Example Skills Using This Pattern

- `software-test-designer` — Phase 1 understanding proof + Thoroughness Priority Rule
- `system-eval-operator` — Phase 1 understanding proof + failure analyst mindset
- `temporal-awareness` — Format Priority Rule + Background vs Foreground disclosure modes

---

## Portable Skill Package Structure (5-File Bundle)

For maximum portability across memory providers, structure skills as a 5-file package:

```
<skill-name>/
├── SKILL.md          # Passive skill prompt (includes embedded fallback policies)
├── ROUTING.md        # Agent-side routing policy (when to load)
├── NUDGE.md          # Skill execution behavior policy (how to behave)
├── metadata.yaml     # Provider-agnostic metadata and adapter hints
└── README.md         # Installation instructions and portability guide
```

### File Roles

| File | Purpose | Portable? | Required? |
|------|---------|-----------|-----------|
| `SKILL.md` | Skill body with embedded fallback policies | ✅ Yes | ✅ Yes |
| `ROUTING.md` | External routing policy (for policy-aware agents) | ✅ Yes | ⚠️ Optional (fallback in SKILL.md) |
| `NUDGE.md` | External behavior policy (for policy-aware agents) | ✅ Yes | ⚠️ Optional (fallback in SKILL.md) |
| `metadata.yaml` | Package metadata, provider adapter hints | ✅ Yes | ✅ Yes |
| `README.md` | Installation + portability documentation | ✅ Yes | ✅ Yes |

### Two Installation Modes

**Mode 1: Optimized Policy Mode** (recommended for Hermes + Brainstack)
- Install `SKILL.md` as the skill body
- Store `ROUTING.md` as a skill activation policy
- Store `NUDGE.md` as skill execution guidance
- Apply `NUDGE.md` only when this skill is loaded

**Mode 2: Embedded Fallback Mode** (for unknown/no policy support)
- Install only `SKILL.md`
- Use embedded policies in SKILL.md for routing and behavior
- Keep `ROUTING.md` and `NUDGE.md` as human-auditable references

### metadata.yaml Structure

```yaml
name: <skill-name>
version: <semver>
status: stable | draft | deprecated
package_type: portable_hermes_skill_bundle
portability_model:
  optimized_policy_mode: Use SKILL.md plus external ROUTING.md and NUDGE.md policy records.
  embedded_fallback_mode: Use SKILL.md only; embedded policies provide routing and behavior guidance.
  recommended_for_hermes_brainstack: optimized_policy_mode
  recommended_for_unknown_provider: embedded_fallback_mode
roles:
  skill:
    file: SKILL.md
    purpose: Passive skill prompt loaded by an agent when invoked.
    portable: true
    fallback_contains_embedded_policies: true
  routing:
    file: ROUTING.md
    type: agent_routing_policy
    scope: skill_activation
    purpose: Describes when this skill should be activated.
    portable_content: true
    provider_specific_storage: true
  nudge:
    file: NUDGE.md
    type: skill_behavior_policy
    scope: only_when_skill_loaded
    purpose: Describes how the agent should behave after this skill is activated.
    portable_content: true
    provider_specific_storage: true
provider_agnostic_policy_mapping:
  routing:
    meaning: skill activation policy
    not_memory_type: user preference
    adapter_hint: Index or store under routing/control/policy namespace.
  nudge:
    meaning: skill execution guidance
    not_memory_type: user preference
    adapter_hint: Inject or retrieve only after this skill is selected.
brainstack_policy_hints:
  shelf: operating
  record_type: canonical_policy
  do_not_summarize_as_user_preference: true
  keep_routing_and_nudge_separate: true
```

### Provider Adapter Mapping

| Provider Style | Routing Storage | Nudge Storage |
|----------------|-----------------|---------------|
| File-based | `ROUTING.md` | `NUDGE.md` |
| Vector DB | policy document tagged `skill_activation` | policy document tagged `skill_execution` |
| Key-value / SQL | record type `skill_routing_policy` | record type `skill_behavior_policy` |
| Prompt-only | embedded fallback section in `SKILL.md` | embedded fallback section in `SKILL.md` |
| Brainstack | `operating/canonical_policy` | `operating/canonical_policy` |

### Embedded Fallback Section Template (for SKILL.md)

Add this section at the end of SKILL.md for fallback mode:

```markdown
---

## Embedded Policies (Fallback Mode)

Use this section when external policy storage is unavailable.

### Auto-Load Triggers

Load this skill when user input contains:
- Strong triggers: [...]
- Contextual triggers: [...]

Do not auto-load for:
- [...]

### Usage Behavior

When this skill is active:
- [...]

Avoid:
- [...]

### External Policy Equivalence

`ROUTING.md` contains the same activation intent in external-policy form.
`NUDGE.md` contains the same usage guidance in external-policy form.
```

### Creating a Portable Package

1. **Write SKILL.md** with embedded fallback section at the end
2. **Extract routing policy** to `ROUTING.md` (same content as embedded section)
3. **Extract behavior policy** to `NUDGE.md` (same content as embedded section)
4. **Write metadata.yaml** with package structure and provider hints
5. **Write README.md** with installation modes and runtime flow
6. **Zip the package** for distribution: `zip -r <skill-name>.zip <skill-name>/`
7. **Push to skills repo** (not hermes-agent repo) — skills live in a dedicated `skills` repository for distribution, separate from the core hermes-agent codebase

**Session-Learned: Skills Repo vs. Hermes-Agent Repo**
- The `skills` repo (`forgedcontextlabs/skills`) is for distributing portable skill packages
- The `hermes-agent` repo (`NousResearch/hermes-agent`) is for core agent code
- Do not push skills to `hermes-agent/skills/` unless contributing upstream via PR
- For personal/instance skills, push to your `skills` fork instead
- Pattern: `https://github.com/<your-org>/skills.git` not `https://github.com/<upstream>/hermes-agent.git`

### Example: temporal-awareness Package

The `temporal-awareness` skill (v1.2.0) demonstrates this pattern:
- 388 lines in SKILL.md (includes lines 340-388 as embedded fallback)
- ROUTING.md with trigger conditions
- NUDGE.md with behavior guidance
- metadata.yaml with provider adapter hints
- README.md with dual-mode installation guide

This package can be installed on:
- Hermes + Brainstack (uses external policies)
- Hermes + mem0/supermemory (uses provider-specific adapters)
- Any agent with skill filesystem (uses embedded fallback)

## GitHub Submission Workflow

When preparing skills for GitHub PR:

1. **Embed all policies** in SKILL.md (auto-load triggers, usage policies, behavioral constraints)
2. **Add README.md** if submitting multiple skills with a shared protocol (e.g., Phase 1 → confirmation → Phase 2)
3. **Verify peer structure** matches existing skills in the target category
4. **Commit locally** with clear version bump and changelog in commit message
5. **Confirm with user before committing** — never commit skill changes without explicit approval
6. **Push and create PR** via `gh` CLI or web UI (requires `gh` auth setup)

**Git Remote Reality Check:**
- Verify the remote URL matches the target repository before pushing
- If the PAT is for a different account than the repo owner, push to a fork first
- Pattern: `https://github.com/<PAT-owner>/<repo>.git` not `https://github.com/<upstream>/<repo>.git`
- **403 Forbidden after successful auth means wrong repo, not wrong credentials** — the PAT authenticated successfully but the account lacks write access to the target repo
- **Troubleshooting sequence:**
  1. Check who the PAT authenticated as: `git remote -v` shows the URL, but the auth identity comes from the credential
  2. Verify repo ownership: `curl -s https://api.github.com/repos/<owner>/<repo> | jq '.owner.login'`
  3. If PAT owner ≠ repo owner, either: (a) use a PAT from an account with write access, (b) push to a fork and PR, or (c) switch to SSH if you have key-based access
  4. Common mistake: using a personal PAT to push to an org repo without confirming org membership/permissions

**Note**: `gh` CLI must be installed and authenticated separately. Use the `github-auth` skill for standard setup.

## Editing Existing In-Repo Skills

- **Small fix (typo, added pitfall, tightened trigger):** `skill_manage(action='patch', name=..., old_string=..., new_string=...)` works fine on in-repo skills.
- **Major rewrite:** `write_file` the whole SKILL.md. `skill_manage(action='edit')` also works but requires supplying the full new content.
- **Adding supporting files:** `write_file` to `skills/<category>/<name>/references/<file>.md`, `templates/<file>`, or `scripts/<file>`. `skill_manage(action='write_file')` also works and enforces the references/templates/scripts/assets subdir allowlist.
- **Always commit** the edit — in-repo skills are source, not runtime state.

## Common Pitfalls

1. **Using `skill_manage(action='create')` for an in-repo skill.** It writes to `~/.hermes/skills/`, not the repo tree. Use `write_file` for in-repo creation.

2. **Leading whitespace before `---`.** The validator checks `content.startswith("---")`; any leading blank line or BOM fails validation.

3. **Description too generic.** Peer descriptions start with "Use when ..." and describe the *trigger class*, not the one task. "Use when debugging X" > "Debug X".

4. **Forgetting the author/license/metadata block.** Not validator-enforced, but every peer has it; omitting makes the skill look half-finished.

5. **Writing a skill that duplicates a peer.** Before creating, `ls skills/<category>/` and open 2-3 peers. Prefer extending an existing skill to creating a narrow sibling.

6. **Expecting the current session to see the new skill.** It won't. The skill loader is initialized at session start. Verify in a fresh session or via `skill_view` using the exact path.

7. **Linking to skills that don't exist in-repo.** `related_skills: [some-user-local-skill]` works for you but breaks for other clones. Prefer only in-repo links.

## Verification Checklist

- [ ] File is at `skills/<category>/<name>/SKILL.md` (not in `~/.hermes/skills/`)
- [ ] Frontmatter starts at byte 0 with `---`, closes with `\n---\n`
- [ ] `name`, `description`, `version`, `author`, `license`, `metadata.hermes.{tags, related_skills}` all present
- [ ] Name ≤ 64 chars, lowercase + hyphens
- [ ] Description ≤ 1024 chars and starts with "Use when ..."
- [ ] Total file ≤ 100,000 chars (aim for 8-15k)
- [ ] Structure: `# Title` → `## Overview` → `## When to Use` → body → `## Common Pitfalls` → `## Verification Checklist`
- [ ] `related_skills` references resolve in-repo (or are explicitly OK to be user-local)
- [ ] `git add skills/<category>/<name>/ && git commit` completed on the intended branch
