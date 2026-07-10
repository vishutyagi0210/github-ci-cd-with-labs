# 🚀 GitHub CI/CD — Beginner to Pro

> Learn GitHub Actions from zero to production-ready pipelines.
> Every module = **visual flow** + **runnable demo workflow**.

---

## 🗺️ Learning Roadmap

```mermaid
graph TD
    START([🟢 START HERE]) --> M01

    subgraph CORE["📦 Core Features"]
        M01["01 · Workflow Fundamentals"]
        M02["02 · Step Types"]
        M03["03 · Workflow Structure — DAG"]
        M04["04 · Triggering Events"]
        M05["05 · Environment Variables"]
        M06["06 · Passing Variables"]
        M07["07 · Secrets & Variables"]
        M08["08 · GitHub Contexts"]
    end

    subgraph ADV["⚡ Advanced Features"]
        M09["09 · Runners"]
        M10["10 · Artifacts & Cache"]
        M11["11 · Matrix & Conditionals"]
        M12["12 · Permissions & Auth"]
    end

    M13["🏆 13 · Real-World CI/CD Pipeline"]

    M01 --> M02 --> M03 --> M04
    M04 --> M05 --> M06 --> M07 --> M08
    M08 --> M09 --> M10 --> M11 --> M12
    M12 --> M13

    style START fill:#00c853,stroke:#00c853,color:#fff
    style M13 fill:#ff6d00,stroke:#ff6d00,color:#fff
    style CORE fill:#e3f2fd,stroke:#1565c0
    style ADV fill:#fce4ec,stroke:#c62828
```

---

## 📂 Module Index

| # | Module | What You'll Learn | Demo Workflows |
|---|--------|-------------------|----------------|
| 01 | [Workflow Fundamentals](./01-workflow-fundamentals/) | workflow → job → step hierarchy | `hello-world.yml` |
| 02 | [Step Types](./02-step-types/) | Shell, scripts, marketplace actions | 3 workflows |
| 03 | [Workflow Structure — DAG](./03-workflow-structure-dag/) | Parallel jobs, `needs` keyword | `dag-demo.yml` |
| 04 | [Triggering Events](./04-triggering-events/) | Manual, push, PR, cron triggers | 4 workflows |
| 05 | [Environment Variables](./05-environment-variables/) | Scope: workflow / job / step | `env-scopes.yml` |
| 06 | [Passing Variables](./06-passing-variables/) | `GITHUB_OUTPUT`, `GITHUB_ENV`, inter-job | 3 workflows |
| 07 | [Secrets & Variables](./07-secrets-and-variables/) | Repo/org secrets, config variables | `secrets-demo.yml` |
| 08 | [GitHub Contexts](./08-github-contexts/) | `github.*`, `runner.*`, `matrix.*` | `contexts-explorer.yml` |
| 09 | [Runners](./09-runners/) | GitHub-hosted, self-hosted, 3rd-party | `multi-runner.yml` |
| 10 | [Artifacts & Cache](./10-artifacts-and-cache/) | Persist data, speed up builds | 2 workflows |
| 11 | [Matrix & Conditionals](./11-matrix-and-conditionals/) | Multi-OS × multi-version, `if`, concurrency | `matrix-demo.yml` |
| 12 | [Permissions & Auth](./12-permissions-and-auth/) | `GITHUB_TOKEN`, OIDC, external auth | `permissions-demo.yml` |
| 13 | [Real-World CI/CD](./13-real-world-ci-cd/) | Full pipeline: lint → test → build → deploy | `full-pipeline.yml` |

---

## 🧪 How to Use the Demo Workflows

```mermaid
graph LR
    A["📋 Copy .yml file"] --> B["📁 Paste to your repo's<br/>.github/workflows/"]
    B --> C["⬆️ Push to GitHub"]
    C --> D["🖱️ Go to Actions tab"]
    D --> E["▶️ Run workflow"]
    E --> F["📊 See logs & output"]

    style A fill:#e8eaf6,stroke:#283593
    style F fill:#e8f5e9,stroke:#2e7d32
```

> **All demos use `workflow_dispatch`** — you can trigger them manually from the GitHub Actions tab. No code changes needed!

---

## 📋 Prerequisites

- A GitHub account (free tier works)
- A GitHub repository (public or private)
- Basic YAML knowledge (indentation = structure)

---

*Built with ❤️ for DevOps engineers who learn by doing.*
