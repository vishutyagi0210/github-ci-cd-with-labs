# 12 · Permissions and Auth

> **`GITHUB_TOKEN` is auto-generated per run. You can scope its permissions and use OIDC for external services.**

---

## 🔍 Authentication Layers

```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    AUTH["🔐 Authentication"] --> TOKEN["🔑 GITHUB_TOKEN<br/>(built-in, auto-generated)"]
    AUTH --> PAT["🎫 Personal Access Token<br/>(stored as secret)"]
    AUTH --> OIDC["🪪 OIDC<br/>(keyless auth to cloud)"]

    TOKEN --> T1["Scoped to THIS repo"]
    TOKEN --> T2["Expires when run ends"]
    TOKEN --> T3["Permissions configurable"]

    PAT --> P1["Created by a user"]
    PAT --> P2["Stored as repo secret"]
    PAT --> P3["Cross-repo access"]

    OIDC --> O1["No long-lived secrets"]
    OIDC --> O2["AWS, GCP, Azure support"]
    OIDC --> O3["Short-lived tokens"]

```

---

## 🔑 GITHUB_TOKEN Permissions

### Default vs Explicit:

```mermaid
%%{init: {"theme": "dark"}}%%
graph LR
    subgraph DEFAULT["Default (permissive)"]
        D1["contents: read ✅"]
        D2["packages: read ✅"]
        D3["issues: write ✅"]
        D4["pull-requests: write ✅"]
        D5["...many more write ✅"]
    end

    subgraph EXPLICIT["Explicit (least privilege)"]
        E1["contents: read ✅"]
        E2["packages: read ✅"]
        E3["Everything else: none ❌"]
    end

    DEFAULT -->|"Best practice:<br/>use explicit"| EXPLICIT

```

### Setting Permissions:

```yaml
# ── Workflow-level (applies to all jobs) ──
permissions:
  contents: read
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest

    # ── Job-level (overrides workflow-level) ──
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4        # Needs contents: read
```

### Permission Reference:

| Scope | `read` | `write` | Common Use |
|-------|--------|---------|------------|
| `contents` | Checkout code | Push commits, create releases |
| `packages` | Pull images | Push images to GHCR |
| `pull-requests` | Read PR info | Comment on PRs |
| `issues` | Read issues | Create/update issues |
| `id-token` | — | OIDC authentication |
| `actions` | Read workflow | Cancel/rerun workflows |

---

## 🪪 OIDC — Keyless Cloud Auth

```mermaid
%%{init: {"theme": "dark"}}%%
sequenceDiagram
    participant WF as ▶️ GitHub Workflow
    participant GH as 🐙 GitHub OIDC Provider
    participant CLOUD as ☁️ Cloud Provider (AWS/GCP)

    WF->>GH: Request OIDC token
    GH-->>WF: JWT token (short-lived)
    WF->>CLOUD: Exchange JWT for cloud credentials
    CLOUD->>CLOUD: Verify JWT (trust GitHub OIDC)
    CLOUD-->>WF: Temporary credentials (15 min)
    WF->>CLOUD: Use credentials to deploy
    Note over WF: No long-lived secrets needed! 🎉
```

### Example — AWS OIDC:

```yaml
permissions:
  id-token: write              # 👈 Required for OIDC
  contents: read

steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789:role/github-actions
      aws-region: us-east-1
      # No AWS_ACCESS_KEY needed! 🎉

  - run: aws s3 ls               # Now authenticated via OIDC
```

---

## 🧪 Demo Workflow

📄 **File:** [`.github/workflows/permissions-demo.yml`](./.github/workflows/permissions-demo.yml)

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Using default (permissive) permissions | Always set explicit `permissions:` |
| Storing cloud keys as secrets | Use OIDC instead (no long-lived secrets) |
| `GITHUB_TOKEN` can't access other repos | Use a PAT secret or GitHub App token |
| Fork PRs have read-only token | By design — prevents secret exfiltration |

---

[⬅️ Matrix & Conditionals](../11-matrix-and-conditionals/) · [Next: Real-World CI/CD ➡️](../13-real-world-ci-cd/)
