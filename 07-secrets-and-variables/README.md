# 07 · Secrets and Variables

> **Secrets = encrypted, masked in logs. Variables = plain config values.**

---

## 🔍 Secrets vs Variables

```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    subgraph SEC["🔒 Secrets"]
        S1["Encrypted at rest"]
        S2["Masked in logs (****)"]
        S3["Cannot be read back from UI"]
        S4["e.g., API keys, tokens, passwords"]
    end

    subgraph VAR["📋 Variables"]
        V1["Stored in plain text"]
        V2["Visible in logs"]
        V3["Can be read from UI"]
        V4["e.g., app name, region, flags"]
    end

```

---

## 🏗️ Where They Live — 3 Levels

```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    ORG["🏢 Organization Level"] --> REPO["📁 Repository Level"]
    REPO --> ENV["🌍 Environment Level<br/>(staging, production)"]

    ORG_S["Secrets + Variables<br/>Shared across all repos"] -.-> ORG
    REPO_S["Secrets + Variables<br/>This repo only"] -.-> REPO
    ENV_S["Secrets + Variables<br/>Only for this environment<br/>+ approval gates"] -.-> ENV

```

### Precedence (narrower wins):

```
Environment secret  >  Repository secret  >  Organization secret

┌─────────────────────────────────────────────┐
│ Org:  API_KEY = "org-default-key"           │
│  ┌───────────────────────────────────────┐  │
│  │ Repo: API_KEY = "repo-specific-key"  │  │
│  │  ┌────────────────────────────────┐   │  │
│  │  │ Env (prod): API_KEY = "prod-x" │  │  │
│  │  │ 👉 This one wins in prod env   │  │  │
│  │  └────────────────────────────────┘   │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

---

## 📝 How to Set Them

### Via GitHub UI:

```
Repository → Settings → Secrets and variables → Actions
  ├── Secrets tab → "New repository secret"
  └── Variables tab → "New repository variable"
```

### Via CLI:

```bash
# Set a secret
gh secret set API_KEY --body "my-secret-value"

# Set a variable
gh variable set APP_REGION --body "us-east-1"

# Set environment-specific secret
gh secret set DB_PASSWORD --env production --body "p@ssw0rd"
```

---

## 📝 How to Use Them

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production              # 👈 Activates env-level secrets
    steps:
      - name: Use a secret
        env:
          API_KEY: ${{ secrets.API_KEY }}  # 👈 secrets context
        run: |
          echo "Key length: ${#API_KEY}"
          # echo "$API_KEY"   ← This would show **** in logs

      - name: Use a variable
        run: |
          echo "Region: ${{ vars.APP_REGION }}"   # 👈 vars context
          echo "App: ${{ vars.APP_NAME }}"
```

---

## 🔒 Security Flow

```mermaid
%%{init: {"theme": "dark"}}%%
sequenceDiagram
    participant DEV as 👤 Developer
    participant UI as 🖥️ GitHub Settings
    participant VAULT as 🔐 Encrypted Storage
    participant WF as ▶️ Workflow Run

    DEV->>UI: Set secret "API_KEY = abc123"
    UI->>VAULT: Encrypt and store
    Note over VAULT: Encrypted with Libsodium<br/>sealed box

    DEV->>WF: Trigger workflow
    WF->>VAULT: Request ${{ secrets.API_KEY }}
    VAULT-->>WF: Decrypt → "abc123"
    Note over WF: If echoed in logs → "****"
    WF-->>DEV: Logs show **** (masked)
```

---

## 🧪 Demo Workflow

📄 **File:** [`.github/workflows/secrets-demo.yml`](./.github/workflows/secrets-demo.yml)

> ⚠️ **Before running:** Create these in your repo settings:
> - Secret: `DEMO_SECRET` = any value
> - Variable: `DEMO_VAR` = any value

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Echoing secrets in logs | GitHub masks them, but avoid `echo $SECRET` anyway |
| Concatenating secrets to bypass masking | `echo "${SECRET}x"` can leak — be careful |
| Using secrets in `if:` conditions | Secrets can't be directly used in `if:` — set as env first |
| PRs from forks can't access secrets | By design — use `pull_request_target` carefully |

---

[⬅️ Passing Variables](../06-passing-variables/) · [Next: GitHub Contexts ➡️](../08-github-contexts/)
