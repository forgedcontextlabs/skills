# Chroma Embedding Configuration Options

## Overview

Brainstack's Chroma corpus backend supports three embedding configurations:

1. **External TEI (Production)** — Dedicated Text Embeddings Inference service
2. **Chroma Default (Non-Production)** — Built-in sentence-transformers
3. **SQLite Fallback** — Lexical search only (no vectors)

---

## Option 1: External TEI Service (Recommended for Production)

### Setup

```bash
# Run TEI with a production model
docker run -d -p 8080:80 huggingface/text-embeddings-inference:cpu-1.5 \
  --model-id BAAI/bge-m3 \
  --revision refs/pr/79

# Set environment variables
export BRAINSTACK_EMBEDDINGS_URL=http://localhost:8080/embed
export BRAINSTACK_EMBEDDINGS_PROVIDER=tei
export BRAINSTACK_EMBEDDINGS_MODEL=bge-m3
```

### Supported TEI Models

| Model | Dimensions | Max Tokens | Use Case |
|-------|------------|------------|----------|
| `BAAI/bge-m3` | 1024 | 8192 | General purpose, multilingual |
| `sentence-transformers/all-MiniLM-L6-v2` | 384 | 512 | Fast, lightweight |
| `thenlper/gte-large` | 1024 | 512 | High accuracy |
| `intfloat/multilingual-e5-large` | 1024 | 512 | Multilingual |

### Environment Variables

```bash
BRAINSTACK_EMBEDDINGS_URL=http://localhost:8080/embed
BRAINSTACK_EMBEDDINGS_PROVIDER=tei
BRAINSTACK_EMBEDDINGS_MODEL=bge-m3
BRAINSTACK_EMBEDDINGS_QUERY_PREFIX=query:
BRAINSTACK_EMBEDDINGS_DOCUMENT_PREFIX=document:
BRAINSTACK_EMBEDDINGS_TIMEOUT_SECONDS=15
```

### Verification

```bash
curl -X POST http://localhost:8080/embed \
  -H "Content-Type: application/json" \
  -d '{"inputs": ["test embedding"]}' | jq
```

Expected: JSON array of floats (embedding vector)

---

## Option 2: Chroma Default Embedding (Non-Production Only)

### When to Use

- Local development
- Testing Brainstack functionality
- No Docker/container runtime available
- Quick validation before TEI setup

### Setup

```bash
export BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING=true
```

Add to `~/.bashrc` for persistence:

```bash
echo 'export BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING=true' >> ~/.bashrc
```

### Trade-offs

| Aspect | Chroma Default | External TEI |
|--------|----------------|--------------|
| **Performance** | Slow (in-process) | Fast (dedicated service) |
| **Memory** | Loads into Python process | Isolated in container |
| **Model Control** | Fixed (sentence-transformers) | Choose any TEI-supported model |
| **Production Ready** | No | Yes |
| **Setup Complexity** | Zero (env var only) | Moderate (Docker + model) |

### How It Works

Chroma's `DefaultEmbeddingFunction` uses `sentence-transformers/all-MiniLM-L6-v2` internally. The embedding is computed in the same Python process as Brainstack.

**Code path** (`corpus_backend_chroma.py:42-44`):

```python
def _build_embedding_function(self) -> Any:
    chromadb, _ = self._import_chromadb()
    return chromadb.utils.embedding_functions.DefaultEmbeddingFunction()
```

---

## Option 3: SQLite Fallback (Degraded Mode)

### When This Activates

- `BRAINSTACK_EMBEDDINGS_URL` not set
- `BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING` not set or false
- Chroma backend fails to initialize

### Behavior

- Vector similarity search: **unavailable**
- Lexical (keyword) search: **functional**
- Memory operations: **functional** (via SQLite substrate)
- Semantic recall quality: **reduced** (no vector similarity)

### Detection

```json
{
  "backend_health": {
    "corpus": {
      "status": "degraded",
      "reason_code": "BACKEND_EMBEDDING_CONFIG_MISSING",
      "sqlite_fallback_active": true
    }
  }
}
```

---

## Collection Naming with External Embeddings

When using an external embedding service, Brainstack creates collections with a fingerprint suffix to avoid cross-model contamination:

```python
def _collection_name_for_embedding(base_name: str, fingerprint: str) -> str:
    suffix = hashlib.sha256(str(fingerprint).encode("utf-8")).hexdigest()[:12]
    return f"{base_name}_{suffix}"
```

**Fingerprint includes:**
- Embedding API type (`tei`, `openai`)
- Model name
- Query/document prefixes
- URL hash

This means switching models creates a new collection — existing corpus docs won't be re-embedded automatically.

---

## Troubleshooting

### "Chroma default embedding is disabled"

**Cause:** Neither `BRAINSTACK_EMBEDDINGS_URL` nor `BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING` is set.

**Fix:** Choose Option 1 (TEI) or Option 2 (default embedding).

### "Connection refused" to TEI endpoint

**Cause:** TEI service not running or wrong port.

**Fix:**
```bash
docker ps | grep text-embeddings
curl -s http://localhost:8080/embed -d '{"inputs":["test"]}' -H "Content-Type: application/json"
```

### Slow embedding performance

**Cause:** Chroma default embedding (in-process) or undersized TEI container.

**Fix:** 
- For TEI: Increase container resources (`--cpus`, `--memory`)
- For default: Accept limitation (non-prod only) or migrate to TEI

### Collection count stays at 0

**Cause:** No corpus documents published yet. This is normal — corpus is populated as Brainstack processes sessions.

**Verify:** Check `brainstack_stats` row counts:
```json
{
  "row_counts": {
    "corpus_documents": 0,
    "corpus_sections": 0,
    "semantic_evidence_index": 97
  }
}
```

`semantic_evidence_index` being non-zero means semantic recall still works via the index even if corpus is empty.

---

## Migration Path

For production deployments:

1. Start with `BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING=true` for validation
2. Deploy TEI service with chosen model
3. Set `BRAINSTACK_EMBEDDINGS_URL` and related vars
4. Remove `BRAINSTACK_CHROMA_ALLOW_DEFAULT_EMBEDDING` from `.bashrc`
5. Restart Hermes sessions to pick up new config

**Note:** Existing corpus documents remain in the old collection (fingerprinted by embedding config). New documents go to the new collection. This is intentional — mixing embeddings from different models corrupts similarity search.
