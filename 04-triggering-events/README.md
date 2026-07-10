# 04 · Triggering Events

> **The `on:` keyword defines WHEN your workflow runs.**

---

## 🔍 All Trigger Types at a Glance

```mermaid
graph TD
    ON["on:"] --> MANUAL["🖱️ Manual<br/>workflow_dispatch"]
    ON --> PUSH["⬆️ Push<br/>push"]
    ON --> PR["🔀 Pull Request<br/>pull_request"]
    ON --> CRON["⏰ Schedule<br/>schedule"]
    ON --> OTHER["📦 Others<br/>release, issues, etc."]

    style ON fill:#1565c0,stroke:#1565c0,color:#fff
    style MANUAL fill:#e8f5e9,stroke:#2e7d32
    style PUSH fill:#e3f2fd,stroke:#1565c0
    style PR fill:#fff3e0,stroke:#e65100
    style CRON fill:#fce4ec,stroke:#c62828
    style OTHER fill:#f5f5f5,stroke:#9e9e9e
```

---

## 📊 Quick Reference

| Trigger | When it fires | Use case |
|---------|--------------|----------|
| `workflow_dispatch` | Manual click in UI | Testing, ad-hoc deploys |
| `push` | Code pushed to branch | CI — lint, test, build |
| `pull_request` | PR opened/updated/merged | Code review checks |
| `schedule` (cron) | On a time schedule | Nightly builds, cleanups |
| `release` | Release published | Publishing packages |
| `workflow_call` | Called by another workflow | Reusable workflows |

---

## 1️⃣ Manual Trigger (`workflow_dispatch`)

```mermaid
sequenceDiagram
    participant U as 👤 You
    participant GH as 🐙 GitHub UI

    U->>GH: Click "Run workflow"
    U->>GH: Fill in inputs (optional)
    GH->>GH: Start workflow
    GH-->>U: ✅ Done
```

```yaml
on:
  workflow_dispatch:
    inputs:                        # 👈 Optional user inputs
      environment:
        description: 'Deploy to which env?'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      debug:
        description: 'Enable debug?'
        type: boolean
        default: false
```

> Access inputs via: `${{ github.event.inputs.environment }}`

---

## 2️⃣ Push Trigger

```mermaid
graph LR
    DEV["👤 Developer"] -->|git push| REPO["📁 GitHub Repo"]
    REPO -->|matches branch/path filter?| CHECK{Filter}
    CHECK -->|✅ Yes| RUN["▶️ Run Workflow"]
    CHECK -->|❌ No| SKIP["⏭️ Skip"]

    style RUN fill:#e8f5e9,stroke:#2e7d32
    style SKIP fill:#ffebee,stroke:#c62828
```

```yaml
on:
  push:
    branches:
      - main                      # Only on main
      - 'release/**'              # Any release/* branch
    paths:
      - 'src/**'                  # Only if files in src/ changed
      - '!src/**/*.md'            # Ignore markdown changes
    tags:
      - 'v*'                      # On version tags (v1.0, v2.3)
```

### Filter Logic:

```
branches + paths = AND logic
  ├── Branch matches? ──── AND ──── Path matches?
  │        ✅                          ✅          → ▶️ Run
  │        ✅                          ❌          → ⏭️ Skip
  │        ❌                          ✅          → ⏭️ Skip
  └────────❌──────────────────────────❌──────────→ ⏭️ Skip
```

---

## 3️⃣ Pull Request Trigger

```mermaid
graph TD
    PR["🔀 Pull Request Event"] --> TYPES["Activity Types"]
    TYPES --> OPENED["opened"]
    TYPES --> SYNC["synchronize<br/>(new commits pushed)"]
    TYPES --> CLOSED["closed"]
    TYPES --> REVIEW["review_requested"]

    style PR fill:#fff3e0,stroke:#e65100
```

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]  # 👈 Which PR activities
    branches:
      - main                                 # Only PRs targeting main
    paths:
      - 'src/**'                             # Only if src/ changed
```

> **Default types** (if you don't specify): `opened`, `synchronize`, `reopened`

---

## 4️⃣ Schedule (Cron)

```mermaid
graph LR
    CRON["⏰ Cron Expression"] --> GH["GitHub checks every ~minute"]
    GH --> MATCH{Time matches?}
    MATCH -->|Yes| RUN["▶️ Run Workflow"]
    MATCH -->|No| WAIT["💤 Wait"]

    style CRON fill:#fce4ec,stroke:#c62828
    style RUN fill:#e8f5e9,stroke:#2e7d32
```

```yaml
on:
  schedule:
    - cron: '30 5 * * 1-5'    # At 5:30 UTC, Monday through Friday
```

### Cron Cheat Sheet:

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-6, Sun=0)
│ │ │ │ │
* * * * *
```

| Expression | Meaning |
|------------|---------|
| `0 0 * * *` | Midnight UTC daily |
| `30 5 * * 1-5` | 5:30 AM UTC weekdays |
| `0 */6 * * *` | Every 6 hours |
| `0 9 1 * *` | 9 AM on the 1st of each month |

---

## 🔗 Combining Multiple Triggers

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:            # Also allow manual trigger
  schedule:
    - cron: '0 0 * * 0'        # Weekly on Sunday midnight
```

```mermaid
graph TD
    W["Workflow"] --> T1["push to main"]
    W --> T2["PR to main"]
    W --> T3["Manual trigger"]
    W --> T4["Every Sunday midnight"]

    T1 --> RUN["▶️ Same workflow runs"]
    T2 --> RUN
    T3 --> RUN
    T4 --> RUN

    style RUN fill:#e8f5e9,stroke:#2e7d32
```

---

## 🧪 Demo Workflows

| File | Trigger type |
|------|-------------|
| [`manual-trigger.yml`](./.github/workflows/manual-trigger.yml) | `workflow_dispatch` with inputs |
| [`push-trigger.yml`](./.github/workflows/push-trigger.yml) | `push` with branch/path filters |
| [`pr-trigger.yml`](./.github/workflows/pr-trigger.yml) | `pull_request` events |
| [`cron-trigger.yml`](./.github/workflows/cron-trigger.yml) | `schedule` with cron |

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Cron runs on wrong timezone | Cron is **always UTC** |
| Cron schedule delays | GitHub doesn't guarantee exact timing (can be up to 15 min late) |
| PR workflows on forks can't access secrets | Use `pull_request_target` (⚠️ carefully) |

---

[⬅️ Workflow Structure (DAG)](../03-workflow-structure-dag/) · [Next: Environment Variables ➡️](../05-environment-variables/)
