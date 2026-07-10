# 13 · Real-World CI/CD Pipeline (Capstone)

> **Everything combined: a production-grade pipeline with lint → test → build → deploy.**

---

## 🔍 The Complete Pipeline

```mermaid
graph TD
    TRIGGER["🔀 Push to main<br/>or Manual trigger"] --> LINT

    subgraph CI["🔵 CI — Continuous Integration"]
        LINT["🔍 Lint<br/>Code quality check"]
        TEST["🧪 Test<br/>Unit + Integration"]
        SECURITY["🔒 Security Scan<br/>Vulnerability check"]
        BUILD["📦 Build<br/>Compile + Package"]
    end

    subgraph CD["🟢 CD — Continuous Deployment"]
        STAGING["🟡 Deploy Staging<br/>environment: staging"]
        PROD["🟢 Deploy Production<br/>environment: production"]
    end

    LINT --> TEST
    LINT --> SECURITY
    TEST --> BUILD
    SECURITY --> BUILD
    BUILD --> STAGING
    STAGING --> PROD

    style TRIGGER fill:#e8eaf6,stroke:#283593
    style CI fill:#e3f2fd,stroke:#1565c0
    style CD fill:#e8f5e9,stroke:#2e7d32
    style LINT fill:#fff3e0,stroke:#e65100
    style TEST fill:#e3f2fd,stroke:#1565c0
    style SECURITY fill:#fce4ec,stroke:#c62828
    style BUILD fill:#e8eaf6,stroke:#283593
    style STAGING fill:#fff9c4,stroke:#f9a825
    style PROD fill:#c8e6c9,stroke:#2e7d32
```

---

## 📊 CI vs CD — What's the Difference?

```
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  CI (Continuous Integration)                                 │
│  ─────────────────────────                                   │
│  "Does the code work?"                                       │
│                                                              │
│  ┌──────┐  ┌──────┐  ┌──────────┐  ┌───────┐               │
│  │ Lint │→ │ Test │→ │ Security │→ │ Build │               │
│  └──────┘  └──────┘  └──────────┘  └───────┘               │
│                                                              │
│  • Runs on EVERY push / PR                                   │
│  • Fast feedback (< 10 min)                                  │
│  • Catches bugs before merge                                 │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CD (Continuous Deployment)                                  │
│  ──────────────────────────                                  │
│  "Ship it!"                                                  │
│                                                              │
│  ┌─────────────┐  ┌──────────────────┐                      │
│  │ Deploy      │→ │ Deploy           │                      │
│  │ Staging     │  │ Production       │                      │
│  │ (auto)      │  │ (manual approve) │                      │
│  └─────────────┘  └──────────────────┘                      │
│                                                              │
│  • Runs only on main branch (after CI passes)                │
│  • Staging = automatic                                       │
│  • Production = requires approval                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔄 Execution Timeline

```mermaid
sequenceDiagram
    participant DEV as 👤 Developer
    participant GH as 🐙 GitHub
    participant CI_R as 🔵 CI Runners
    participant STG as 🟡 Staging
    participant PRD as 🟢 Production

    DEV->>GH: git push main
    GH->>CI_R: Trigger workflow

    par Lint + Security (parallel)
        CI_R->>CI_R: 🔍 Lint check
        CI_R->>CI_R: 🔒 Security scan
    end

    CI_R->>CI_R: 🧪 Run tests
    CI_R->>CI_R: 📦 Build artifact
    CI_R->>GH: Upload artifact

    CI_R->>STG: 🟡 Deploy to staging
    STG-->>GH: ✅ Staging healthy

    Note over PRD: ⏸️ Waiting for approval
    DEV->>GH: ✅ Approve production deploy
    GH->>CI_R: Continue
    CI_R->>PRD: 🟢 Deploy to production
    PRD-->>DEV: ✅ Live!
```

---

## 📝 Pipeline Anatomy — Concept Mapping

Every concept you learned maps to a part of this pipeline:

| Module | Where it appears |
|--------|-----------------|
| 01 - Workflow/Job/Step | The entire YAML structure |
| 02 - Step Types | `run:` commands + `uses:` actions |
| 03 - DAG | `needs:` between lint → test → build → deploy |
| 04 - Triggers | `push` to main + `workflow_dispatch` |
| 05 - Env Variables | `env:` at workflow and job levels |
| 06 - Passing Variables | `outputs` from build → deploy |
| 07 - Secrets | `${{ secrets.DEPLOY_TOKEN }}` |
| 08 - Contexts | `${{ github.sha }}` for versioning |
| 09 - Runners | `runs-on: ubuntu-latest` |
| 10 - Artifacts | Build artifact uploaded → downloaded for deploy |
| 11 - Matrix | Test across multiple Node versions |
| 12 - Permissions | `contents: read`, `packages: write` |

---

## 🧪 Demo Workflow

📄 **File:** [`.github/workflows/full-pipeline.yml`](./.github/workflows/full-pipeline.yml)

### How to test:
1. Copy to your repo's `.github/workflows/`
2. Go to **Actions tab** → **"13 - Full CI/CD Pipeline"** → **"Run workflow"**
3. Watch the DAG execute: lint → test → build → staging → production
4. Every stage prints detailed output

---

## 🏗️ Extending This Pipeline

### Add Docker build:
```yaml
- name: Build Docker image
  run: docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .

- name: Push to registry
  run: docker push ghcr.io/${{ github.repository }}:${{ github.sha }}
```

### Add Kubernetes deploy:
```yaml
- name: Deploy to K8s
  run: |
    kubectl set image deployment/app \
      app=ghcr.io/${{ github.repository }}:${{ github.sha }}
```

### Add Slack notification:
```yaml
- name: Notify Slack
  if: always()
  uses: slackapi/slack-github-action@v1
  with:
    channel-id: 'deployments'
    slack-message: "Deploy ${{ job.status }}: ${{ github.repository }}@${{ github.sha }}"
```

---

## 🎓 What's Next?

```mermaid
graph LR
    NOW["✅ You completed<br/>this course!"] --> NEXT1["📦 Reusable Workflows<br/>(workflow_call)"]
    NOW --> NEXT2["🐳 Docker CI/CD<br/>(build + push images)"]
    NOW --> NEXT3["☸️ K8s Deployments<br/>(Helm + ArgoCD)"]
    NOW --> NEXT4["🔀 Branch Protection<br/>(required checks)"]

    style NOW fill:#e8f5e9,stroke:#2e7d32
```

---

[⬅️ Permissions & Auth](../12-permissions-and-auth/) · [🏠 Back to Roadmap](../README.md)
