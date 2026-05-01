# Brainstack Memory Plugin Installation

## Discovery: PyPI Package is Unrelated

The `brainstack` package on PyPI (v0.0.1) is **NOT** the Hermes memory plugin. It is a different project: "A CPU-friendly, modular mini-brain with explainable routing and tiny experts" by Abhinav Basu.

**Do NOT install via:**
```bash
pip install brainstack  # WRONG - unrelated project
```

## Correct Installation Method

The Brainstack memory plugin must be installed as a Hermes plugin from GitHub:

```bash
# Step 1: Install plugin from GitHub
hermes plugins install Yepyhun/Brainstack --enable

# Step 2: Move to correct location (if installer puts it in wrong place)
mkdir -p ~/.hermes/plugins/memory
mv ~/.hermes/plugins/Brainstack ~/.hermes/plugins/memory/brainstack

# Step 3: Run the Brainstack installer script
python3 ~/.hermes/plugins/memory/brainstack/install_into_hermes.py \
    ~/.hermes/hermes-agent \
    --enable \
    --config ~/.hermes/config.yaml \
    --embedding-runtime none
```

## Installer Options

| Flag | Purpose |
|------|---------|
| `--enable` | Patch config.yaml to enable Brainstack |
| `--config <path>` | Path to Hermes config.yaml |
| `--embedding-runtime none` | No embedding service (corpus search unavailable but core memory works) |
| `--embedding-runtime local-tei-jina` | Requires `--runtime docker` — adds TEI Jina v5 service |
| `--embedding-runtime external` | Operator-managed embedding endpoint |
| `--skip-deps` | Skip installing kuzu/chromadb |
| `--doctor` | Run diagnostics after install |

## Verification

```bash
hermes memory status
# Should show:
#   Plugin:    installed ✓
#   Status:    available ✓
#   brainstack (local) ← active
```

## Common Failures

### "FAIL target is not a Hermes checkout"
**Cause:** Wrong path to Hermes installation  
**Fix:** Use `~/.hermes/hermes-agent` as target

### "FAIL No Hermes agent config found"
**Cause:** Config file not detected  
**Fix:** Add `--config ~/.hermes/config.yaml`

### "--embedding-runtime local-tei-jina requires --runtime docker"
**Cause:** TEI Jina embedding requires Docker runtime  
**Fix:** Use `--embedding-runtime none` for local installs without Docker

### Plugin installed but `hermes memory status` shows "NOT installed"
**Cause:** Plugin in wrong directory  
**Fix:** Ensure plugin is in `~/.hermes/plugins/memory/brainstack/` (not `~/.hermes/plugins/Brainstack/`)

## Post-Install

Restart the gateway for changes to take effect:
```bash
hermes gateway restart
```

## References

- Plugin source: https://github.com/Yepyhun/Brainstack
- Hermes memory plugin discovery: `~/.hermes/hermes-agent/plugins/memory/__init__.py`
- Only ONE memory provider can be active at a time (selected via `memory.provider` in config.yaml)
