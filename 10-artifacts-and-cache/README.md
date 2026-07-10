# 10 · Artifacts and Cache

> **Jobs run on ephemeral VMs — when the job ends, everything is gone. Artifacts and cache persist data.**

---

## 🔍 Artifacts vs Cache

```mermaid
graph TD
    subgraph ART["📦 Artifacts"]
        A1["Upload files FROM a job"]
        A2["Download files IN another job"]
        A3["Download from GitHub UI"]
        A4["Retained for 90 days"]
        A5["Use: build outputs, test reports, logs"]
    end

    subgraph CACHE["⚡ Cache"]
        C1["Speed up repeated installs"]
        C2["Key-based lookup"]
        C3["10 GB limit per repo"]
        C4["LRU eviction"]
        C5["Use: node_modules, pip, Maven deps"]
    end

    style ART fill:#e3f2fd,stroke:#1565c0
    style CACHE fill:#e8f5e9,stroke:#2e7d32
```

---

## 📊 Quick Comparison

| | Artifacts | Cache |
|---|---|---|
| **Purpose** | Share files between jobs / download later | Speed up dependency installs |
| **Lifetime** | 90 days (configurable) | LRU evicted, 10 GB limit |
| **Scope** | Workflow run (cross-job) | Repository (cross-run) |
| **Action** | `upload-artifact` / `download-artifact` | `actions/cache` |
| **Accessible from UI?** | ✅ Yes (download ZIP) | ❌ No |

---

## 📦 Artifacts Flow

```mermaid
sequenceDiagram
    participant JA as 🟦 Job A (Build)
    participant STORE as ☁️ Artifact Storage
    participant JB as 🟦 Job B (Deploy)
    participant UI as 👤 GitHub UI

    JA->>JA: Build → creates dist/
    JA->>STORE: upload-artifact: dist/
    STORE-->>JA: ✅ Uploaded

    JB->>STORE: download-artifact
    STORE-->>JB: dist/ (restored)
    JB->>JB: Deploy using dist/

    UI->>STORE: Download ZIP from Actions page
    STORE-->>UI: 📥 dist.zip
```

### Upload:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output          # Artifact name
    path: dist/                 # What to upload
    retention-days: 30          # How long to keep (default: 90)
```

### Download:

```yaml
- uses: actions/download-artifact@v4
  with:
    name: build-output          # Must match upload name
    path: ./downloaded/         # Where to put it
```

---

## ⚡ Cache Flow

```mermaid
sequenceDiagram
    participant WF as ▶️ Workflow Run
    participant CACHE as ☁️ Cache Store
    
    WF->>CACHE: Check cache key "npm-abc123"
    
    alt Cache HIT ✅
        CACHE-->>WF: Restore node_modules/
        WF->>WF: Skip npm install (fast!)
    else Cache MISS ❌
        WF->>WF: npm install (slow)
        WF->>CACHE: Save node_modules/ with key "npm-abc123"
    end
```

### Usage:

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm                                    # What to cache
    key: npm-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |                                 # Fallback keys
      npm-${{ runner.os }}-
```

### Cache Key Strategy:

```
key: npm-Linux-abc123def   ← Exact match (ideal)
                ↓ miss
restore-keys:
  npm-Linux-              ← Prefix match (close enough)
                ↓ miss
  npm-                    ← Broader prefix
                ↓ miss
  (cache miss — full install)
```

---

## 🧪 Demo Workflows

| File | What it demonstrates |
|------|---------------------|
| [`upload-download.yml`](./.github/workflows/upload-download.yml) | Artifact upload in Job A → download in Job B |
| [`cache-demo.yml`](./.github/workflows/cache-demo.yml) | Dependency caching with `actions/cache` |

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Artifact name collision across matrix jobs | Include `${{ matrix.os }}` in artifact name |
| Cache key too generic | Use `hashFiles()` to include lock file hash |
| Caching `node_modules/` directly | Cache `~/.npm` instead — more reliable |
| Cache > 10 GB | Least recently used caches get evicted |

---

[⬅️ Runners](../09-runners/) · [Next: Matrix & Conditionals ➡️](../11-matrix-and-conditionals/)
