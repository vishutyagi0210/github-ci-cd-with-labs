# 14 · Composite Actions — The Industry Standard

> **A composite action bundles multiple steps into one reusable `uses:` block. No Node.js, no Docker — just YAML.**

---

## 🔍 What is a Composite Action?

```mermaid
%%{init: {"theme": "dark"}}%%
graph LR
    subgraph BEFORE["❌ Without Composite Actions"]
        direction TB
        W1["Workflow A"] --> S1["Step 1: Checkout"]
        W1 --> S2["Step 2: Setup JDK"]
        W1 --> S3["Step 3: Maven Build"]
        W1 --> S4["Step 4: Run Tests"]

        W2["Workflow B"] --> S5["Step 1: Checkout"]
        W2 --> S6["Step 2: Setup JDK"]
        W2 --> S7["Step 3: Maven Build"]
        W2 --> S8["Step 4: Run Tests"]
    end

    subgraph AFTER["✅ With Composite Actions"]
        direction TB
        W3["Workflow A"] --> CA1["uses: setup-java-build"]
        W4["Workflow B"] --> CA2["uses: setup-java-build"]

        CA1 --> INSIDE["Internally runs all 4 steps"]
        CA2 --> INSIDE
    end
```

> **One action.yml → multiple workflows reuse it → DRY pipelines**

---

## 🔍 Composite vs Other Action Types

```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    ACTION["🔷 Custom Action"] --> COMP["📦 Composite"]
    ACTION --> JS["⚡ JavaScript"]
    ACTION --> DOCKER["🐳 Container"]

    COMP --> C1["✅ Pure YAML — no build step"]
    COMP --> C2["✅ No runtime dependency"]
    COMP --> C3["✅ Can use ANY marketplace action inside"]
    COMP --> C4["✅ Fastest to create and maintain"]
    COMP --> C5["⚠️ Limited — no pre/post hooks"]

    JS --> J1["✅ Full programmatic logic"]
    JS --> J2["⚠️ Needs Node.js runtime"]
    JS --> J3["⚠️ Must compile/bundle"]

    DOCKER --> D1["✅ Any language/tool"]
    DOCKER --> D2["⚠️ Slow — container pull"]
    DOCKER --> D3["⚠️ Linux runners only"]
```

---

## 🔍 The action.yml Anatomy

Every composite action lives in its own folder and has **one required file: `action.yml`**

```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    subgraph FOLDER[".github/actions/my-action/"]
        FILE["action.yml"]
    end

    FILE --> NAME["name: My Action"]
    FILE --> DESC["description: What it does"]
    FILE --> INPUTS["inputs: parameters it accepts"]
    FILE --> OUTPUTS["outputs: values it returns"]
    FILE --> RUNS["runs: using: composite"]

    RUNS --> STEPS["steps: the actual work"]
    STEPS --> S1["- uses: actions/checkout@v4"]
    STEPS --> S2["- run: echo hello"]
    STEPS --> S3["- uses: another/action@v1"]
```

### Syntax Breakdown

```yaml
# .github/actions/my-action/action.yml

name: "My Custom Action"              # 👈 Display name
description: "Does X, Y, Z"           # 👈 What it does

# ─── INPUTS (parameters the caller passes in) ───
inputs:
  java-version:
    description: "JDK version"
    required: true                     # 👈 Caller MUST provide
    default: "17"                      # 👈 Optional default
  build-args:
    description: "Extra Maven args"
    required: false

# ─── OUTPUTS (values returned to the caller) ───
outputs:
  artifact-path:
    description: "Path to built JAR"
    value: ${{ steps.build.outputs.jar-path }}  # 👈 From a step

# ─── THE COMPOSITE RUNTIME ───
runs:
  using: "composite"                   # 👈 THIS makes it composite
  steps:
    - name: Checkout
      uses: actions/checkout@v4        # 👈 Can use marketplace actions

    - name: Setup JDK
      uses: actions/setup-java@v4
      with:
        java-version: ${{ inputs.java-version }}  # 👈 Access inputs

    - name: Build
      id: build
      shell: bash                      # 👈 MUST specify shell in composite
      run: |
        mvn clean package ${{ inputs.build-args }}
        echo "jar-path=target/*.jar" >> $GITHUB_OUTPUT
```

> **🔑 Key Rules:**
> - `runs.using` MUST be `"composite"`
> - Every `run:` step MUST have `shell:` specified (bash, pwsh, etc.)
> - Access inputs with `${{ inputs.xxx }}` (not `env`)
> - Can mix `run:` and `uses:` steps freely

---

## 🔍 How to Reference Composite Actions

```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    subgraph LOCAL["📁 Same Repo (Local)"]
        L1["uses: ./.github/actions/my-action"]
    end

    subgraph REMOTE["🌐 Another Repo (Remote)"]
        R1["uses: org/repo/.github/actions/my-action@main"]
    end

    subgraph MARKET["🏪 Published to Marketplace"]
        M1["uses: my-org/my-action@v1"]
    end

    LOCAL --> BEST["✅ Most common in production"]
    REMOTE --> GOOD["✅ Good for org-wide sharing"]
    MARKET --> PUB["✅ Public/open-source distribution"]
```

---

## 🔍 Input/Output Data Flow

```mermaid
%%{init: {"theme": "dark"}}%%
sequenceDiagram
    participant WF as 🔄 Workflow
    participant CA as 📦 Composite Action
    participant S1 as Step 1 (inside)
    participant S2 as Step 2 (inside)

    WF->>CA: calls with inputs (java-version: 17)
    CA->>S1: ${{ inputs.java-version }} → 17
    S1->>S1: Setup JDK 17
    S1->>S2: GITHUB_OUTPUT → jar-path
    S2->>S2: Build JAR
    S2->>CA: step output → jar-path
    CA->>WF: output.artifact-path → target/app.jar
```

---

## 🏗️ Real Project Structure — Java CI/CD with Composites

This module includes a **complete working example**:

```
14-composite-actions/
├── README.md                              ← You are here
├── app/                                   ← Sample Java app
│   ├── src/main/java/com/demo/App.java
│   ├── src/main/resources/application.properties
│   ├── pom.xml
│   └── Dockerfile
├── .github/
│   ├── actions/                           ← 🔑 Composite actions live here
│   │   ├── setup-java-build/
│   │   │   └── action.yml                 ← CI: checkout + JDK + Maven build
│   │   ├── docker-build-push/
│   │   │   └── action.yml                 ← CD: Docker build + tag + push
│   │   └── deploy/
│   │       └── action.yml                 ← CD: Pull + run on server
│   └── workflows/
│       └── ci-cd-pipeline.yml             ← Main pipeline using composites
```

---

## 🔍 Pipeline Flow

```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    PUSH["⬆️ Push to main"] --> WF["🔄 ci-cd-pipeline.yml"]

    WF --> CI["🟦 CI Job<br/>runs-on: ubuntu-latest"]
    CI --> CA1["📦 uses: setup-java-build<br/>inputs: java-version: 17"]
    CA1 --> BUILD_DONE["✅ JAR built + tested"]

    BUILD_DONE --> CD["🟦 CD Job<br/>runs-on: self-hosted"]
    CD --> CA2["📦 uses: docker-build-push<br/>inputs: image-name, tag"]
    CA2 --> IMAGE_DONE["✅ Docker image pushed"]

    IMAGE_DONE --> DEPLOY["🟦 Deploy Job<br/>runs-on: self-hosted"]
    DEPLOY --> CA3["📦 uses: deploy<br/>inputs: image, port"]
    CA3 --> LIVE["🟢 App running on server"]
```

---

## 🔍 Why Composite Actions Win in Production

```mermaid
%%{init: {"theme": "dark"}}%%
graph TD
    subgraph TEAM["👥 Engineering Team"]
        DEV1["Dev Team A"]
        DEV2["Dev Team B"]
        DEV3["Dev Team C"]
    end

    subgraph PLATFORM["🏗️ Platform Team maintains"]
        CA1["📦 setup-java-build"]
        CA2["📦 docker-build-push"]
        CA3["📦 deploy"]
        CA4["📦 notify-slack"]
        CA5["📦 run-sonar-scan"]
    end

    DEV1 --> CA1
    DEV1 --> CA2
    DEV2 --> CA1
    DEV2 --> CA3
    DEV3 --> CA1
    DEV3 --> CA5

    CA1 --> UPDATE["Platform team updates action.yml<br/>→ ALL teams get the fix automatically"]
```

> **This is exactly how it works at scale:**
> - Platform/DevOps team owns the composite actions
> - Dev teams just `uses:` them with inputs
> - One fix in `action.yml` → every pipeline is updated
> - No copy-paste, no drift, no "works on my pipeline"

---

## 📝 Quick Reference

| What | How |
|------|-----|
| Create a composite action | `action.yml` with `runs.using: composite` |
| Reference locally | `uses: ./.github/actions/action-name` |
| Reference from another repo | `uses: org/repo/.github/actions/action-name@ref` |
| Pass inputs | `with: { key: value }` in the caller |
| Read inputs inside action | `${{ inputs.key }}` |
| Return outputs | `echo "key=value" >> $GITHUB_OUTPUT` + declare in `outputs:` |
| Shell requirement | Every `run:` MUST have `shell: bash` (or pwsh, sh, etc.) |
| Can use marketplace actions? | ✅ Yes — mix `run:` and `uses:` freely |
| Pre/post hooks? | ❌ No — only JS/Docker actions support this |

---

## 🔗 Files in This Module

| File | What it does |
|------|-------------|
| [App.java](app/src/main/java/com/demo/App.java) | Simple Spring Boot REST API |
| [pom.xml](app/pom.xml) | Maven build config |
| [Dockerfile](app/Dockerfile) | Multi-stage Docker build |
| [setup-java-build/action.yml](.github/actions/setup-java-build/action.yml) | **Composite** — checkout + JDK + Maven build |
| [docker-build-push/action.yml](.github/actions/docker-build-push/action.yml) | **Composite** — Docker build + tag + push |
| [deploy/action.yml](.github/actions/deploy/action.yml) | **Composite** — Pull image + run container |
| [ci-cd-pipeline.yml](.github/workflows/ci-cd-pipeline.yml) | Main pipeline using all 3 composites |
