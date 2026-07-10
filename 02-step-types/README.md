# 02 · Step Types

> **A step can be 3 things: a shell command, a script file, or a marketplace action.**

---

## 🔍 The 3 Types at a Glance

```mermaid
graph LR
    STEP["🔷 Step"] --> T1["🖥️ Shell Command"]
    STEP --> T2["📄 Script File"]
    STEP --> T3["📦 Marketplace Action"]

    T1 --> E1["run: echo 'hello'"]
    T2 --> E2["run: bash ./scripts/deploy.sh"]
    T3 --> E3["uses: actions/checkout@v4"]

    style STEP fill:#1565c0,stroke:#1565c0,color:#fff
    style T1 fill:#e8eaf6,stroke:#283593
    style T2 fill:#e8eaf6,stroke:#283593
    style T3 fill:#e8eaf6,stroke:#283593
    style E1 fill:#f5f5f5,stroke:#9e9e9e
    style E2 fill:#f5f5f5,stroke:#9e9e9e
    style E3 fill:#f5f5f5,stroke:#9e9e9e
```

---

## 📊 Comparison

| | Shell Command | Script File | Marketplace Action |
|---|---|---|---|
| **Keyword** | `run:` | `run:` | `uses:` |
| **What it does** | Executes inline command | Runs an external script | Runs pre-built logic from GitHub |
| **Best for** | Quick one-liners | Complex multi-line logic | Standard tasks (checkout, setup, deploy) |
| **Example** | `run: npm test` | `run: bash ./scripts/test.sh` | `uses: actions/setup-node@v4` |

---

## 1️⃣ Shell Command (`run:`)

```yaml
steps:
  # Single line
  - name: Print greeting
    run: echo "Hello!"

  # Multi-line (use pipe |)
  - name: Multiple commands
    run: |
      echo "Step 1: Install"
      npm install
      echo "Step 2: Test"
      npm test
```

```mermaid
graph LR
    R["run:"] --> SHELL["Default: bash"]
    R --> PS["shell: pwsh"]
    R --> PY["shell: python"]

    style R fill:#1565c0,color:#fff
```

> You can change the shell with `shell: pwsh` (PowerShell) or `shell: python`.

---

## 2️⃣ Script File (`run:` with a file path)

```
📁 your-repo/
├── .github/workflows/
│   └── build.yml
└── scripts/
    └── deploy.sh      ← 👈 your script
```

```yaml
steps:
  # ⚠️ MUST checkout first — the VM starts empty!
  - uses: actions/checkout@v4

  - name: Run deploy script
    run: bash ./scripts/deploy.sh
```

> **Why checkout first?** The runner VM starts with an empty workspace. `actions/checkout` clones your repo into it.

---

## 3️⃣ Marketplace Action (`uses:`)

```mermaid
graph TD
    A["uses: owner/action-name@version"] --> B["GitHub downloads the action"]
    B --> C["Action runs inside your job"]
    C --> D["Provides outputs / side-effects"]

    style A fill:#e8eaf6,stroke:#283593
    style D fill:#e8f5e9,stroke:#2e7d32
```

```yaml
steps:
  # Checkout your code
  - uses: actions/checkout@v4

  # Setup Node.js (from GitHub Marketplace)
  - uses: actions/setup-node@v4
    with:                          # 👈 Inputs for the action
      node-version: '20'

  # Install and test
  - run: npm install && npm test
```

### Version pinning formats:

| Format | Example | Meaning |
|--------|---------|---------|
| `@v4` | `actions/checkout@v4` | Major version (recommended) |
| `@v4.1.0` | `actions/checkout@v4.1.0` | Exact version |
| `@main` | `actions/checkout@main` | Branch (⚠️ risky) |
| `@sha` | `actions/checkout@abc123` | Commit SHA (most secure) |

---

## ▶️ Execution Flow

```mermaid
sequenceDiagram
    participant GH as 🐙 GitHub
    participant VM as 🖥️ Runner VM

    GH->>VM: Start job
    Note over VM: Step 1: uses: actions/checkout@v4
    VM->>VM: Clone repository into workspace
    Note over VM: Step 2: run: echo "hello"
    VM->>VM: Execute shell command
    Note over VM: Step 3: run: bash ./scripts/build.sh
    VM->>VM: Execute script file
    Note over VM: Step 4: uses: actions/setup-node@v4
    VM->>VM: Download & run marketplace action
    VM-->>GH: ✅ All steps complete
```

---

## 🧪 Demo Workflows

| File | What it demonstrates |
|------|---------------------|
| [`shell-commands.yml`](./.github/workflows/shell-commands.yml) | Inline commands + multi-line + different shells |
| [`run-script.yml`](./.github/workflows/run-script.yml) | Running an external `.sh` script file |
| [`third-party-action.yml`](./.github/workflows/third-party-action.yml) | Using marketplace actions with `with:` inputs |

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Using `uses:` and `run:` in the same step | ❌ Not allowed — each step is **one or the other** |
| Forgetting `actions/checkout` before running scripts | The VM starts empty — your code isn't there yet |
| Using `@main` for action versions | Pin to a version tag like `@v4` for stability |

---

[⬅️ Workflow Fundamentals](../01-workflow-fundamentals/) · [Next: Workflow Structure (DAG) ➡️](../03-workflow-structure-dag/)
