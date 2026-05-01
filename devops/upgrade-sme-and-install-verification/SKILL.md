---
name: upgrade-sme-and-install-verification
description: Pre-install verification, high-risk detection, post-upgrade SME analysis, and snapshot discipline for software upgrades and installs
category: devops
version: 1.0
author: Hex
created: 2026-04-30
trigger: Any software install, upgrade, or version change on infrastructure I administer
---

## Purpose

This skill prevents silent breakage during software upgrades by enforcing verification before installation, detecting high-risk patterns that require escalation, performing systematic post-upgrade analysis, and maintaining snapshot discipline. It exists because upgrades can silently break things when local changes are restashed on top of upstream commits, when patch seams conflict between components, or when install methods are inferred rather than verified.

## Part 1 — Pre-Install / Pre-Upgrade Verification

Before installing or upgrading any software, complete these steps in order:

### Step 1.1 — Locate Official Source
- Find the official project repository (GitHub, GitLab, or project website)
- Verify it is the canonical source (check stars, forks, official documentation links)
- Record the repository URL

### Step 1.2 — Read Official Documentation
- Read the README in full
- Read any INSTALL, SETUP, or DEPLOY documentation
- Read release notes or changelog for the target version
- Identify the developer-recommended install method explicitly

### Step 1.3 — Verify Install Method Authority
- Confirm the install method is documented by the project developers
- Do NOT accept install methods from:
  - Stack Overflow answers
  - Forum posts
  - Third-party blogs
  - Prior personal experience without current documentation
- If no official install method exists, or documentation is ambiguous/incomplete/contradictory: **STOP and report to Jack**

### Step 1.4 — Document Planned Approach
Record before proceeding:
- Software name and target version
- Source repository URL
- Install method to be used
- Developer documentation URL that authorizes the method
- Any planned deviations and justification

## Part 2 — High-Risk Install Detection

The following triggers require **mandatory stop-and-report** before proceeding. If any are present, halt and present findings to Jack with a recommended path forward.

### Trigger 2.1 — Core Application File Modification
- Installer modifies core application files (`gateway/run.py`, `run_agent.py`, main entry points)
- Installer touches files that other components patch or depend on

### Trigger 2.2 — Stash/Restore Operations
- Install involves stashing local changes
- Install restores local changes on top of upstream commits
- Any git operation that blends local work with upstream updates

### Trigger 2.3 — Missing or Ambiguous Documentation
- No official install documentation exists
- Documentation is incomplete, contradictory, or outdated
- Install steps are inferred rather than stated

### Trigger 2.4 — Undocumented Steps Required
- Install method requires steps not documented by the developer
- Workarounds, patches, or manual interventions are needed
- Configuration must be guessed or reverse-engineered

### Trigger 2.5 — Large Commit Delta
- Upgrade involves more than 20 commits since last known-good state
- Changelog indicates breaking changes, migrations, or structural changes

### Trigger 2.6 — Multi-Component File Conflicts
- Install affects files shared between two or more components
- Example: Brainstack and Hermes both touching `gateway/run.py`
- Any file that is a patch seam between systems

**When any trigger fires:**
1. Stop immediately
2. Document which trigger(s) fired and why
3. Present to Jack with recommended path forward
4. Do not proceed until explicit approval is received
5. If Jack does not respond, wait. Do not proceed due to time pressure.

## Part 3 — Post-Upgrade SME Analysis

After any authorized software upgrade, complete these steps before declaring success:

### Step 3.1 — Read Changelog Fully
- Read the complete changelog or release notes
- Identify breaking changes, deprecations, migrations
- Note any files or APIs that changed

### Step 3.2 — Identify Modified Files
- List every file modified by the upgrade that you interact with directly
- For git-based installs: `git diff --name-only <old-commit> <new-commit>`
- For package installs: check package manifest for installed paths

### Step 3.3 — Check Patch Seams
- For any file also patched by another component, verify patch still applies cleanly
- Example: if Brainstack patches `gateway/run.py`, check for conflicts
- Run patch dry-run if applicable: `patch --dry-run`
- Look for:
  - Rejected hunks
  - Offset warnings
  - Context mismatches

### Step 3.4 — Run Health Checks
- Run available doctor/health tools:
  - `hermes doctor` (if available)
  - `brainstack_doctor.py` or `brainstack stats`
  - Application-specific health endpoints
- Check service status and connectivity

### Step 3.5 — Verify Dependencies
- Confirm all dependent components still wire correctly
- Check imports, function signatures, API contracts
- Look for undefined variables, broken imports, changed signatures

### Step 3.6 — Produce Findings Report
Create a brief report with:
- What changed (version, commit range, key modifications)
- What was checked (files, patches, dependencies, health checks)
- What passed
- What failed or needs attention
- Recommended next steps

### Step 3.7 — Present Before Restart
- Present findings report to Jack
- Do not restart services or declare upgrade complete until reviewed
- If issues found, propose remediation plan

## Part 4 — Snapshot Discipline

### Before Upgrade
- Confirm a pre-upgrade VM snapshot exists
- If no snapshot exists, request one from Jack before proceeding
- Record snapshot name/timestamp

### After Successful Upgrade
- After clean post-upgrade report and Jack's approval
- Request a new snapshot to establish new baseline
- Record snapshot name/timestamp

## Part 5 — Install Method Documentation

After every install or upgrade, create a structured note for future reference. When LLMWiki is operational, this goes to `raw/inbox/` first (airlock rule). Until then, save to `~/.hermes/docs/installs/`.

### Required Fields
```markdown
## Software: <name>
- Version: <version>
- Date: <YYYY-MM-DD>
- Source: <repository URL>
- Install Method: <method name/description>
- Documentation: <URL to official docs>
- Deviations: <any deviations from documented method and why>
- Verification: <post-install verification results>
- Snapshot Before: <snapshot name/timestamp>
- Snapshot After: <snapshot name/timestamp>
```

## Pitfalls

- **Silent restash breakage**: Git stashes applied on top of upstream commits can break without obvious errors. Always check patch seams.
- **Assumed authority**: Install methods from blogs, Stack Overflow, or memory are not authoritative. Only developer documentation counts.
- **Time pressure escalation**: If Jack does not respond, wait. Do not proceed with high-risk operations because time has passed.
- **Partial verification**: Running health checks but not checking patch seams (or vice versa) leaves gaps. Complete all steps.
- **Shared file blindness**: Files touched by multiple components (Hermes + Brainstack, etc.) are conflict magnets. Always check seams.

## Verification Checklist

Before declaring an upgrade complete, confirm:
- [ ] Official install method identified and documented
- [ ] No high-risk triggers present (or Jack approved with triggers present)
- [ ] Pre-upgrade snapshot confirmed
- [ ] Changelog read in full
- [ ] Modified files listed
- [ ] Patch seams checked for conflicts
- [ ] Health checks passed
- [ ] Dependencies verified
- [ ] Findings report produced
- [ ] Jack reviewed findings
- [ ] Post-upgrade snapshot requested
- [ ] Install documentation written