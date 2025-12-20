# 009-AT-ARCH-shared-toolbox — Shared Toolbox Architecture Notes

**Purpose:** Capture the requirements and rollout plan for the Firebase + Cloud Run “shared toolbox” pattern supporting Vertex AI Agent Engine agents (as outlined in the Google memory/tooling tutorials).

## Target Architecture

```
User
  |
  v
Vertex AI Agent Engine (primary agent)
  |
  |--> HTTPS call to single toolbox URL (Firebase Hosting)
        |
        +--> /api/mcp/orders/**  -> Cloud Run: mcp-orders-server (DB access)
        +--> /api/mcp/email/**   -> Cloud Run: mcp-email-server (SendGrid)
        +--> /api/mcp/search/**  -> Cloud Run: mcp-search-server (Google Search)
```

**Key components**

- **Agent Engine (ADK orchestrator):** Delegates tool calls via Model Context Protocol (MCP) endpoints.
- **Firebase Hosting:** Single public entry point; `firebase.json` rewrites multiplex calls to the right Cloud Run services.
- **Cloud Run MCP services:** One container per tool, each with tailored IAM and dependencies; starts on demand and returns MCP-compliant JSON responses.
- **IAM / Security:** Principle of least privilege per service (e.g., orders tool only reaches databases; email tool wired to SendGrid).

## Implementation Checklist

1. **Tool Definition**
   - [ ] Enumerate required MCP tools (`orders`, `email`, `search`, etc.).
   - [ ] Write MCP-compliant servers (Python/Node) per tool with standardized schemas.

2. **Cloud Run Deployment**
   - [ ] Containerize each tool with minimum base image.
   - [ ] Grant per-service IAM (Cloud SQL, Secret Manager, third-party APIs).
   - [ ] Configure scale-to-zero + concurrency as needed.

3. **Firebase Front Door**
   - [ ] Create Firebase Hosting site (e.g., `my-agent-tools`).
   - [ ] Author `firebase.json` rewrites for `/api/mcp/<tool>/**` → Cloud Run service (`serviceId`, region).
   - [ ] Enable HTTPS and enforce authentication policy (e.g., signed requests, custom headers).

4. **Agent Configuration**
   - [ ] Register toolbox base URL in ADK tool configuration.
   - [ ] Ensure tools surface through `AgentCard` / A2A metadata.
   - [ ] Document retry/offline handling (e.g., fallback message when tool unreachable).

5. **Observability**
   - [ ] Centralize logging / metrics (Cloud Logging, Error Reporting per Cloud Run service).
   - [ ] Add structured logs for tool invocations and durations (trace correlation).

## Outstanding Questions (`TODO(ask)`)

- Clarify authentication strategy between Agent Engine and Firebase (signed JWT vs. API key).
- Confirm if Firebase Hosting can reside in same project/region for latency guarantees.
- Determine standardized MCP response schema for each tool family (orders/email/search).
- Align Terraform/Deployment scripts to provision Firebase + Cloud Run alongside agent packaging.

Use this document as the reference when we begin implementing the shared toolbox stack. Sync with `STATUS.md` as items graduate from planning to execution.
