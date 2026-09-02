# 🌲 Holly Core

A private, self-hosted infrastructure sandpit and local AI development workstation running on **Ubuntu Linux** with a fully automated **GitHub Actions (GHA)** CI/CD pipeline. 

This repository implements a strict GitOps model. Changes pushed from a remote editor (like VS Code on Windows) automatically trigger a local self-hosted runner to validate, pull, rebuild, and prune the containerized services in place.

---

## 🏗️ Architecture Overview

The system is decoupled into three primary architectural layers managed seamlessly by [Docker Compose](https://docs.docker.com/compose/):

```
 ┌─────────────────────────┐
 │  Windows PC / VS Code   │◀─── [Local Editing & Ghost Text Autocomplete]
 └────────────┬────────────┘
              │ (git push)
              ▼
 ┌─────────────────────────┐
 │   GitHub Cloud Engine   │
 └────────────┬────────────┘
              │ (Secure Webhook Trigger via Outbound HTTPS)
              ▼
 ┌─────────────────────────────────────────────────────────────────────────┐
 │                        Ubuntu Sandpit Node                              │
 │                                                                         │
 │  ┌─────────────────┐   ┌──────────────────┐   ┌──────────────────────┐  │
 │  │  GHA Workflows  │ ═>│  Docker Daemon   │ ═>│  Prometheus/Grafana  │  │
 │  │ (Runner: giblet)│   │ (Declares State) │   │ (Streams GPU Metrics)│  │
 │  └─────────────────┘   └────────┬─────────┘   └──────────────────────┘  │
 │                                 │                                       │
 │                                 ▼                                       │
 │                    ┌──────────────────────────┐                         │
 │                    │   RTX 4070 Ti Super GPU  │                         │
 │                    │ (Local 14B/1.5B Inference)                         │
 │                    └──────────────────────────┘                         │
 └─────────────────────────────────────────────────────────────────────────┘
```

1. **Automation & Delivery Layer:** A secure, unprivileged system service worker running directly under the host user profile context (`giblet`). It listens for repository push signals via outbound-only HTTPS polling (requiring no inbound firewall holes or open ports).
2. **Local Inference & Chat Layer:** 
   * **Ollama Backend:** Harnesses an NVIDIA GeForce RTX 4070 Ti Super (16GB VRAM) to execute localized LLM orchestration weights completely network-isolated.
   * **Open WebUI:** Provides a sleek, browser-based chat environment mapped internally over Docker bridge links to interface with models.
3. **Observability & Metrics Layer:** Prometheus target integrations tracking physical hardware metrics, power consumption spikes, and compute engine thermals.

---

## 🧠 Local Intelligence Matrix

The AI layer leverages split-concurrency configurations (`OLLAMA_MAX_LOADED_MODELS=2` and `OLLAMA_NUM_PARALLEL=4`) inside VRAM memory lanes to power two distinct tasks simultaneously:

| Model Variant | Role inside VS Code / WebUI | Purpose | Resource Footprint |
| :--- | :--- | :--- | :--- |
| **Qwen 2.5 Coder 14B** | `chat`, `edit`, `apply` | Deep technical reasoning, multi-file code generation, refactoring, and general chat interface inside Open WebUI. | ~9.3 GB VRAM |
| **Qwen 2.5 Coder 1.5B** | `autocomplete` | Near-zero-latency ghost-text completion predictions tracking cursor actions directly inside the IDE. | ~1.2 GB VRAM |

---

## 🛠️ Configuration Blueprints

### VS Code Extension Integration (`config.yaml`)
To connect your local Windows VS Code workspace directly to your hardware engine, the native [Continue.dev](https://docs.continue.dev/) extension configuration is formatted as follows:

```yaml
name: Main Config
version: 1.0.0
schema: v1

models:
  - name: Qwen 2.5 Coder 14B (Chat)
    provider: ollama
    model: qwen2.5-coder:14b
    apiBase: http://10.1.1.5:11434/
    roles: [chat, edit, apply]

  - name: Qwen 2.5 Coder 1.5B (Autocomplete)
    provider: ollama
    model: qwen2.5-coder:1.5b
    apiBase: http://10.1.1.5:11434/
    roles: [autocomplete]
```

### Continuous Integration Pipeline (`.github/workflows/deploy.yml`)
The workflow utilizes two serial execution blocks to enforce state safety:
* **Lint & Validate:** Fires `docker compose config` syntax evaluations on *any* branch push to prevent typo regressions from reaching system runtimes.
* **Build & Deploy:** Restrained exclusively to the `main` branch timeline. Runs sequential `pull`, `up -d --build`, and `image prune -f` commands to seamlessly swap runtimes and reclaim storage space dynamically.

---

## 🛡️ Sandbox Security Baseline

* **Network Footprint:** Public exposures are entirely contained within the internal local area network (LAN). Runtimes are not exposed to the open internet.
* **Secrets Policy:** Private production API credentials, security tokens, or real databases should never be committed into source history. Utilize local-only `.env` files (safely ignored via `.gitignore`) or explicit secret stores.

---

## 🚀 Future Sprint Backlog

- [ ] **Story 1:** Migrate all remaining plaintext environment credentials to **GitHub Repository Secrets**.
- [ ] **Story 2:** Establish branch-based target routing to deploy experimental pull requests to a sandbox **Staging Port** (e.g., `:8080`) before merging to `main`.
- [ ] **Story 3:** Deploy an **Nginx Proxy Manager** container to achieve local DNS hostname routing (e.g., `http://holly.local`) instead of using clunky raw port strings.

***

*Disclaimer: This is for informational purposes only. For medical advice or diagnosis, consult a professional. AI responses may include mistakes.*
