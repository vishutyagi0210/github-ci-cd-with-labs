# 09 · Runners

> **Runners are the machines that execute your jobs. 3 types: GitHub-hosted, self-hosted, 3rd-party.**

---

## 🔍 The 3 Types

```mermaid
graph TD
    RUNNER["🖥️ Runner"] --> GH["☁️ GitHub-Hosted"]
    RUNNER --> SELF["🏠 Self-Hosted"]
    RUNNER --> THIRD["🏢 3rd-Party Hosted"]

    GH --> GH1["✅ Zero setup"]
    GH --> GH2["✅ Fresh VM every run"]
    GH --> GH3["⚠️ Limited specs"]
    GH --> GH4["⚠️ Usage limits (free tier)"]

    SELF --> SH1["✅ Custom hardware/specs"]
    SELF --> SH2["✅ Access to private networks"]
    SELF --> SH3["⚠️ You manage it"]
    SELF --> SH4["⚠️ Not auto-cleaned"]

    THIRD --> TH1["✅ Managed for you"]
    THIRD --> TH2["✅ Custom specs"]
    THIRD --> TH3["⚠️ Extra cost"]

    style GH fill:#e3f2fd,stroke:#1565c0
    style SELF fill:#fff3e0,stroke:#e65100
    style THIRD fill:#e8f5e9,stroke:#2e7d32
```

---

## 📊 Comparison

| | GitHub-Hosted | Self-Hosted | 3rd-Party |
|---|---|---|---|
| **Setup** | None | Install agent on your server | Sign up + configure |
| **Cost** | Free tier + paid | Your infra cost | Vendor pricing |
| **Specs** | 2 vCPU, 7GB RAM | Whatever you have | Configurable |
| **Clean slate** | ✅ Fresh VM each time | ❌ Persistent (you clean) | Varies |
| **Private network** | ❌ Public internet only | ✅ Your network | Varies |
| **`runs-on`** | `ubuntu-latest` | `self-hosted` | Vendor-specific |
| **Examples** | — | Your own server | BuildJet, Namespace, Actuated |

---

## ☁️ GitHub-Hosted Runners

### Available Machines:

```mermaid
graph LR
    subgraph LINUX["🐧 Linux"]
        L1["ubuntu-latest<br/>(Ubuntu 24.04)"]
        L2["ubuntu-22.04"]
        L3["ubuntu-20.04"]
    end

    subgraph MAC["🍎 macOS"]
        M1["macos-latest<br/>(macOS 14)"]
        M2["macos-13"]
    end

    subgraph WIN["🪟 Windows"]
        W1["windows-latest<br/>(Windows Server 2022)"]
        W2["windows-2019"]
    end

    style LINUX fill:#e3f2fd,stroke:#1565c0
    style MAC fill:#f5f5f5,stroke:#9e9e9e
    style WIN fill:#e8eaf6,stroke:#283593
```

```yaml
jobs:
  linux-job:
    runs-on: ubuntu-latest      # Most common

  mac-job:
    runs-on: macos-latest       # For iOS/macOS builds

  windows-job:
    runs-on: windows-latest     # For .NET/Windows builds
```

---

## 🏠 Self-Hosted Runners

### Setup Flow:

```mermaid
sequenceDiagram
    participant YOU as 👤 You
    participant GH as 🐙 GitHub
    participant SRV as 🖥️ Your Server

    YOU->>GH: Settings → Actions → Runners → New
    GH-->>YOU: Download link + token
    YOU->>SRV: Download & install runner agent
    SRV->>GH: Register with token
    GH-->>SRV: ✅ Connected

    Note over SRV: Runner is now listening<br/>for workflow jobs

    SRV->>SRV: Executes jobs when assigned
    SRV-->>GH: Report results
```

```yaml
jobs:
  on_premises:
    runs-on: self-hosted        # 👈 Basic self-hosted

  with_labels:
    runs-on: [self-hosted, linux, gpu]   # 👈 With labels
```

---

## 🧪 Demo Workflow

📄 **File:** [`.github/workflows/multi-runner.yml`](./.github/workflows/multi-runner.yml)

Runs the **same job** on all 3 OS runners so you can compare their output.

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Using `macos` for simple tasks | macOS runners cost **10x** more minutes — use Linux |
| Self-hosted runner not secured | Add labels, restrict to private repos, use ephemeral runners |
| Hardcoding Linux paths on multi-OS | Use `${{ runner.temp }}` instead of `/tmp` |

---

[⬅️ GitHub Contexts](../08-github-contexts/) · [Next: Artifacts & Cache ➡️](../10-artifacts-and-cache/)
