# 03 · Workflow Structure — DAG

> **Jobs run in parallel by default. Use `needs:` to create dependencies — forming a DAG.**

---

## 🔍 What is a DAG?

**DAG = Directed Acyclic Graph** — a flow with a direction and no loops.

```mermaid
%%{init: {"theme": "dark"}}%%
graph LR
    subgraph DEFAULT["❌ Without 'needs' — Everything Parallel"]
        A1["Job A"] ~~~ B1["Job B"] ~~~ C1["Job C"]
    end

```

```mermaid
%%{init: {"theme": "dark"}}%%
graph LR
    subgraph CONTROLLED["✅ With 'needs' — You Control the Order"]
        A2["Job A"] --> B2["Job B"]
        A2 --> C2["Job C"]
        B2 --> D2["Job D"]
        C2 --> D2
    end

```

---

## 🧬 How `needs` Works

```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    LINT["🔍 Lint"] --> TEST["🧪 Test"]
    LINT --> SECURITY["🔒 Security Scan"]
    TEST --> BUILD["📦 Build"]
    SECURITY --> BUILD
    BUILD --> DEPLOY["🚀 Deploy"]

```

```yaml
jobs:
  lint:                    # ← No 'needs' → runs FIRST
    runs-on: ubuntu-latest
    steps: [...]

  test:
    needs: lint             # ← Waits for lint ✅
    runs-on: ubuntu-latest
    steps: [...]

  security:
    needs: lint             # ← Also waits for lint ✅
    runs-on: ubuntu-latest  #    Runs PARALLEL with test
    steps: [...]

  build:
    needs: [test, security] # ← Waits for BOTH ✅
    runs-on: ubuntu-latest
    steps: [...]

  deploy:
    needs: build            # ← Waits for build ✅
    runs-on: ubuntu-latest
    steps: [...]
```

---

## 📊 Parallel vs Sequential — Visual

```
Timeline →

WITHOUT needs (all parallel):
  ├── Job A ████████
  ├── Job B ████████
  └── Job C ████████
  Total: ~1 unit

WITH needs (sequential chain):
  ├── Job A ████████
  │                 └── Job B ████████
  │                                   └── Job C ████████
  Total: ~3 units

WITH needs (diamond pattern):
  ├── Job A ████████
  │         ├────── Job B ████████
  │         └────── Job C ████████    (B & C run parallel)
  │                          └── Job D ████████
  Total: ~3 units (B and C overlap)
```

---

## 🧩 Common DAG Patterns

### 🔹 Linear Pipeline
```mermaid
%%{init: {"theme": "dark"}}%%
graph LR
    A["Build"] --> B["Test"] --> C["Deploy"]
```
```yaml
needs: build    # test waits for build
needs: test     # deploy waits for test
```

### 🔹 Fan-Out / Fan-In (Diamond)
```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    A["Build"] --> B["Unit Tests"]
    A --> C["Integration Tests"]
    A --> D["Security Scan"]
    B --> E["Deploy"]
    C --> E
    D --> E
```
```yaml
needs: build                        # all 3 wait for build
needs: [unit-tests, int-tests, security]  # deploy waits for ALL 3
```

### 🔹 Independent Chains
```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    A["Build Frontend"] --> B["Deploy Frontend"]
    C["Build Backend"] --> D["Deploy Backend"]
```
```yaml
# No needs between chains — fully parallel
```

---

## 🧪 Demo Workflow

📄 **File:** [`.github/workflows/dag-demo.yml`](./.github/workflows/dag-demo.yml)

This demo creates a **diamond DAG** with 4 jobs and prints timing so you can see the parallel execution.

---

## ⚠️ Common Pitfalls

| Mistake | What happens |
|---------|-------------|
| Circular dependency (`A needs B`, `B needs A`) | ❌ GitHub rejects the workflow |
| Typo in `needs` job name | ❌ Workflow fails to parse |
| Forgetting `needs` | Jobs run in parallel (may not be what you want) |

---

[⬅️ Step Types](../02-step-types/) · [Next: Triggering Events ➡️](../04-triggering-events/)
