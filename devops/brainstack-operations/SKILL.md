---
name: brainstack-operations
version: 1.0.0
category: devops
description: Diagnose and repair Brainstack memory backend issues — Chroma embedding config, Kuzu graph, semantic index, and tier-2 worker health.
tags: [brainstack, memory, troubleshooting, chroma, kuzu, embedding, backend-health]
triggers:
  - "brainstack"
  - "memory backend"
  - "corpus degraded"
  - "embedding config"
  - "brainstack stats"
  - "brainstack doctor"
  - "chroma backend"
  - "graph backend"
  - "semantic index"
  - "tier-2 extraction"
  - "brainstack troubleshooting"
  - "brainstack repair"
  - "brainstack health"
auto_load_policy: |
  Load this skill when the user asks about Brainstack health, status, troubleshooting, or backend configuration. Also load when brainstack_stats, brainstack_doctor, or brainstack_inspect outputs show degraded/unavailable backends.
usage_policy: |
  - Always run brainstack_stats(strict=true) first to get full backend health report
  - Identify the specific degraded backend (corpus, graph, semantic_index, tier2)
  - Check environment variables before assuming code/config issues
  - Dry-run test fixes before declaring success
  - Brainstack reads env vars at initialization — fixes take effect next session
  - Prefer env var fixes over config file changes for Brainstack
  - Verify with direct Python import test when possible
---

# Brainstack Operations & Troubleshooting

## Purpose

Diagnose and repair Brainstack memory backend issues. Brainstack is the durable memory substrate for Hermes Agent, providing graph storage (Kuzu), vector search (Chroma), and semantic indexing.

## Backend Architecture

| Backend | Purpose | Env Var Config |
|---------|---------|----------------|
| **Graph (Kuzu)** | Structured entity/relation storage | `BRAINSTACK_GRAPH_BACKEND=kuzu` |
| **Corpus (Chroma)** | Vector similarity search for documents | `BRAINSTACK_EMBEDDINGS_URL`, `BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING` |
| **Semantic Index** | Lexical + semantic evidence retrieval | Auto-managed |
| **Tier-2 Worker** | Async extraction from transcripts | Config-disabled by default |

## Diagnosis Protocol

### Phase 1: Health Assessment

1. **Run stats with strict mode:**
   ```bash
   brainstack_stats(strict=true)
   ```

2. **Identify degraded backend:**
   - Check `backend_health.backends.*.status`
   - Note `reason_code` and `error_class`
   - Check `fallback_channels` for active fallbacks

3. **Common degradation patterns:**

| Symptom | Reason Code | Fix |
|---------|-------------|-----|
| Corpus degraded | `BACKEND_EMBEDDING_CONFIG_MISSING` | Set `BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING=true` (non-prod) or `BRAINSTACK_EMBEDDINGS_URL=<tei-endpoint>` (prod) |
| Graph unavailable | `BACKEND_DEPENDENCY_MISSING` | Install Kuzu: `pip install kuzu` |
| Semantic index degraded | `SEMANTIC_INDEX_DEGRADED` | Run `brainstack_consolidate(apply=true, maintenance_class='semantic_index')` |
| Tier-2 idle | `tier2.requested=false` | Config setting — not a故障 |

### Phase 2: Environment Variable Discovery

Brainstack configuration is **env-var driven**, not config-file driven. Check:

```bash
env | grep -i brainstack | sort
env | grep -i embed | sort
env | grep -i chroma | sort
```

Key environment variables:

| Variable | Purpose | Typical Value |
|----------|---------|---------------|
| `BRAINSTACK_EMBEDDINGS_URL` | External TEI endpoint for embeddings | `http://localhost:8080/embed` |
| `BRAINSTACK_EMBEDDINGS_PROVIDER` | Embedding provider type | `tei`, `openai` |
| `BRAINSTACK_EMBEDDINGS_MODEL` | Model name for embeddings | `bge-m3`, `text-embedding-3-small` |
| `BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING` | Allow Chroma's built-in embedding (non-prod only) | `true` |
| `BRAINSTACK_DISABLE_CHROMA_DEFAULT_EMBEDDING` | Explicitly disable default embedding | `true` |
| `BRAINSTACK_GRAPH_BACKEND` | Graph backend selection | `kuzu`, `sqlite` |

### Phase 3: Source Code Verification

If env vars don't resolve the issue, read Brainstack source:

```bash
# Location
/home/hrmsadmin/brainstack-standalone/brainstack/

# Key files
corpus_backend_chroma.py   # Chroma embedding logic, env var checks
graph_backend_kuzu.py      # Graph backend initialization
```

**Critical code path for Chroma embedding** (`corpus_backend_chroma.py:68-73`):

```python
if not _allow_chroma_default_embedding():
    raise RuntimeError(
        "Chroma default embedding is disabled. Configure local TEI via "
        "BRAINSTACK_EMBEDDINGS_URL, or explicitly set "
        "BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING=true for non-production diagnostics."
    )
```

The `_allow_chroma_default_embedding()` function checks:
1. `BRAINSTACK_DISABLE_CHROMA_DEFAULT_EMBEDDING` (default=false) — if true, block
2. `BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING` (default=false) — if true, allow

**Both default to false** — explicit opt-in required.

### Phase 4: Dry-Run Verification

Before declaring success, test the fix directly:

```bash
export BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING=true && python3 << 'EOF'
import os
os.environ['BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING'] = 'true'
from brainstack.corpus_backend_chroma import ChromaCorpusBackend

backend = ChromaCorpusBackend(db_path='/home/hrmsadmin/.hermes/brainstack/brainstack.chroma')
try:
    backend.open()
    print(f'SUCCESS: Collection count: {backend.collection.count()}')
    backend.close()
except Exception as e:
    print(f'FAILED: {e}')
    exit(1)
EOF
```

Expected output: `SUCCESS: Collection count: <n>` (count may be 0 if no corpus docs published yet)

### Phase 5: Persistence

Add env vars to `/home/hrmsadmin/.bashrc`:

```bash
echo 'export BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING=true' >> ~/.bashrc
```

**Important:** Brainstack initializes at session start. The fix takes effect on **next session**, not current session.

## Pitfalls

1. **Current session won't see the fix** — Brainstack provider is already initialized. Must restart Hermes or start new session.

2. **Don't use default embedding in production** — It's sentence-transformers based, loaded into Python process. For production, run a dedicated TEI (Text Embeddings Inference) service:
   ```bash
   docker run -d -p 8080:80 huggingface/text-embeddings-inference:cpu-1.5 --model-id BAAI/bge-m3
   export BRAINSTACK_EMBEDDINGS_URL=http://localhost:8080/embed
   ```

3. **Config.yaml doesn't control Brainstack backends** — The `plugins.brainstack` section in `~/.hermes/config.yaml` only sets paths and retrieval limits, not backend enablement. Backend enablement is env-var driven.

4. **SQLite fallback is functional** — When Chroma is degraded, Brainstack falls back to SQLite lexical search. Memory operations still work, just no vector similarity.

5. **Tier-2 worker is config-disabled** — `tier2.requested=false` is intentional. Tier-2 extraction runs on idle window triggers when enabled. Not a故障.

6. **Verify all tools before claiming absence** — Do not claim a tool is not configured until you have checked all relevant variants. Example: `gh` CLI may be missing while `git` is fully configured and authenticated. Always check:
   - `git --version` + `git remote -v` + `git config credential.helper`
   - `gh --version` + `gh auth status`
   - SSH keys: `ls ~/.ssh/*.pub`
   - `.netrc` or credential store files
   False claims about infrastructure state violate Information Integrity (Tenant #1).

## Verification Checklist

- [ ] `brainstack_stats(strict=true)` shows all backends `active`
- [ ] Direct Python import test succeeds
- [ ] Env var persisted to `.bashrc`
- [ ] User informed that fix takes effect next session
- [ ] Production vs. non-prod embedding trade-off explained

## Related Skills

- `temporal-awareness` — Time verification (unrelated but often loaded alongside)
- `software-test-designer` — Test design (Brainstack policies reference this)
- `system-eval-operator` — System evaluation (Brainstack policies reference this)

## Reference Files

- `references/chroma-embedding-config.md` — Detailed Chroma embedding configuration options
- `references/brainstack-env-vars.md` — Complete env var reference
