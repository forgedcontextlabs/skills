# Skill Package Installation Procedure

This reference documents the complete procedure for installing Hermes skill packages from 5-file bundles.

## Package Structure

A portable skill package contains:

```
<skill-name>/
├── SKILL.md          # Passive skill prompt (includes embedded fallback policies)
├── ROUTING.md        # Agent-side routing policy (when to load)
├── NUDGE.md          # Skill execution behavior policy (how to behave)
├── metadata.yaml     # Provider-agnostic metadata and adapter hints
└── README.md         # Installation instructions and portability guide
```

## Installation Steps

### Step 1: Extract Files

```bash
unzip -o <skill-package>.zip -d ~/.hermes/skills/
```

Verify:
```bash
ls -la ~/.hermes/skills/<skill-name>/
# Should show: SKILL.md, ROUTING.md, NUDGE.md, metadata.yaml, README.md
```

### Step 2: Import ROUTING.md as Brainstack Policy

```python
# Use brainstack_remember tool with:
shelf = "operating"
record_type = "canonical_policy"
stable_key = f"skill_routing_policy:{skill_name}"
authority_class = "canonical_policy"
category = "skill_activation"
source_role = "user"
content = <contents of ROUTING.md>
```

### Step 3: Import NUDGE.md as Brainstack Policy

```python
# Use brainstack_remember tool with:
shelf = "operating"
record_type = "canonical_policy"
stable_key = f"skill_behavior_policy:{skill_name}"
authority_class = "canonical_policy"
category = "skill_execution"
source_role = "user"
content = <contents of NUDGE.md>
```

### Step 4: Verify Brainstack Records

```python
# Use brainstack_recall to verify both policies are present:
query = f"skill_routing_policy:{skill_name}"
query = f"skill_behavior_policy:{skill_name}"
```

Expected: Both queries return committed records with matching stable_keys.

### Step 5: Test Skill Loading

```python
# Use skill_view to verify the skill loads:
skill_view(name="<skill-name>")
```

Expected: Skill content returns successfully with `readiness_status: available`.

## Common Failure Modes

### Policy Import Skipped

**Symptom:** Skill file exists but doesn't auto-load on trigger phrases.

**Cause:** ROUTING.md was not imported as a Brainstack canonical_policy.

**Fix:** Complete Step 2 above.

### Wrong Stable Key Format

**Symptom:** Policy exists but routing doesn't fire.

**Cause:** Stable key doesn't match the expected pattern `skill_routing_policy:<skill-name>`.

**Fix:** Re-import with correct stable_key format.

### Wrong Shelf

**Symptom:** Policy exists but isn't consulted during routing.

**Cause:** Policy was saved to `memory` or `user` shelf instead of `operating`.

**Fix:** Re-import with `shelf="operating"`.

### Wrong Record Type

**Symptom:** Policy exists but isn't treated as authoritative.

**Cause:** Record type wasn't `canonical_policy`.

**Fix:** Re-import with `record_type="canonical_policy"`.

## GitHub Submission Notes

When submitting skills to `forgedcontextlabs/skills`:

1. **Directory structure:** Skills go at repo root, NOT in category subdirs.
   - ✓ Correct: `system-eval-operator/SKILL.md`
   - ❌ Wrong: `software-development/system-eval-operator/SKILL.md`

2. **Git identity:** Use org identity for all commits.
   - `git config user.name "forgedcontextlabs"`
   - `git config user.email "forgedcontextlabs@users.noreply.github.com"`
   - Never use personal names or system usernames.

3. **File permissions:** Ensure SKILL.md is readable (644).
   - `chmod 644 SKILL.md`

4. **Commit message:** Use conventional commits format.
   ```
   feat: add <skill-name> skill
   
   - Key feature 1
   - Key feature 2
   
   Author: Forged Context Labs
   ```

## Verification Checklist

- [ ] Skill directory exists at `~/.hermes/skills/<skill-name>/`
- [ ] All 5 files present (SKILL.md, ROUTING.md, NUDGE.md, metadata.yaml, README.md)
- [ ] SKILL.md permissions are 644 (readable)
- [ ] Brainstack policy `skill_routing_policy:<skill-name>` exists on `operating` shelf
- [ ] Brainstack policy `skill_behavior_policy:<skill-name>` exists on `operating` shelf
- [ ] Both policies have `record_type="canonical_policy"`
- [ ] `skill_view(name="<skill-name>")` returns successfully
- [ ] Skill auto-loads on trigger phrases (test with a sample query)
