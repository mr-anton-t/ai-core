# Architecture: MCP Gateway + OpenCode Self-Hosted

Two integration options for leadership review. No VPN, no tokens in chat UI, all secrets stay in-house.

---

## Option 1: MCP Gateway with OAuth (recommended)

```mermaid
flowchart LR
    subgraph Internet
        ChatGPT["ChatGPT<br/>Custom MCP Server"]
    end

    subgraph "Company Infrastructure"
        GW["MCP Gateway<br/>OAuth + AES-256 Vault<br/>Audit Log"]
        N8N["n8n<br/>Orchestrator"]
        Agent["GitHub AI Agent<br/>OpenCode"]
        GH["GitHub<br/>Issues / PRs / Board"]
        Slack["Slack<br/>Notifications"]
        Jenkins["Jenkins<br/>Deploy Only"]
    end

    ChatGPT -->|"HTTPS, OAuth flow<br/>no token in UI"| GW
    N8N -->|"Service account"| GW
    Agent -->|"Local keys"| GW
    GW --> N8N
    GW --> Agent
    N8N --> GH
    N8N --> Slack
    Agent --> GH
    GH -->|"After merge"| Jenkins
```

**How it works:**
- ChatGPT connects to the gateway via OAuth in the browser — zero tokens typed in the chat UI.
- Gateway holds all secrets in an AES-256 vault, injects them only at call time.
- Every tool call is audited on your side.
- n8n and the GitHub agent also route through the same gateway.
- Jenkins stays out of the loop except post-merge deploy.

---

## Option 2: OpenCode Self-Hosted

```mermaid
flowchart LR
    subgraph Internet
        User["User / Browser"]
    end

    subgraph "Company VPS"
        OC["OpenCode serve<br/>HTTP API"]
        Keys["Keys file<br/>mode 600"]
        OC2["AI Agent<br/>orch-system-prompt.txt"]
    end

    subgraph "Company Infrastructure"
        GH2["GitHub"]
        Jenkins2["Jenkins<br/>Deploy Only"]
    end

    User -->|"HTTPS"| OC
    OC --> OC2
    OC2 --> Keys
    OC2 --> GH2
    GH2 -->|"After merge"| Jenkins2
```

**How it works:**
- OpenCode runs on your VPS, exposed as an HTTP server.
- API keys live in a local file with mode 600 — never leave the box.
- Full control: what the agent can do, which keys, which budget.
- Needs public HTTPS or a tunnel for external access.

---

## Recommendation

**Option 1** for the n8n + GitHub board + Slack pipeline: one gateway, one audit trail, OAuth for ChatGPT.
**Option 2** if you want maximum isolation and already run OpenCode.
