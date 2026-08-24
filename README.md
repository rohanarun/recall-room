# Recall Room

Privacy-aware meeting evidence, decisions, owners, commitments, and reviewed follow-up proposals.

Recall Room is a focused, public MIT distribution for the `meetings` module in [managed-oss-cloud](https://github.com/rohanarun/managed-oss-cloud). It includes a product web UI, a product-scoped HTTP client, the `recall-room` CLI, and a stdio MCP server exposing only this product's 9 typed actions.

## Current boundary

This repository is runnable, but it is intentionally not a second database server. Authentication, workspace isolation, shared PostgreSQL storage, plan enforcement, AI execution, and audit records remain behind the managed-oss-cloud API. This product receives a scoped API token and cannot receive database credentials or run database migrations.

- Hosted backend: `https://cloud.getsupers.com`
- Self-hosted backend: any compatible managed-oss-cloud v0.4.2 deployment
- Hosted minimum plan: `scale`
- Resource class: `high`
- Pinned backend source: [v0.4.2](https://github.com/rohanarun/managed-oss-cloud/tree/v0.4.2) at `20c4a704c77cbbbff1da995e1d91b937625a8aa4`

## AI-native by construction

- Transcript-cited proposals
- Human-owned decisions
- Approval-gated redaction

AI actions use their own `ai` token scope, preserve the typed action contract, and return durable backend job evidence. They do not grant the model database credentials or bypass approval, plan, tenant, or action boundaries.

## Run the CLI

Node.js 20 or newer is the only local dependency.

```bash
npm install
npm link
export RECALL_ROOM_TOKEN="a-scoped-workspace-token"
export RECALL_ROOM_URL="https://cloud.getsupers.com"
recall-room actions
recall-room workspace
recall-room action meeting-create '{"title":"Launch review","purpose":"Choose a safe launch window","startsAt":"2026-08-25T15:00:00.000Z","privacy":"confidential","idempotencyKey":"meetings.meeting-create.example-0001"}'
```

The generic `SUPERSUITE_TOKEN` and `SUPERSUITE_URL` variables are supported as fallbacks. Create a token in the workspace dashboard with only the `read`, `write`, and/or `ai` scopes the client needs.

## Run the web UI

The UI proxies requests through the local Node server so the workspace API token is never sent to the browser. Browser access is protected by a separate key of at least 24 characters.

```bash
export RECALL_ROOM_TOKEN="a-scoped-workspace-token"
export RECALL_ROOM_URL="https://cloud.getsupers.com"
export RECALL_ROOM_WEB_KEY="a-separate-random-browser-key"
npm start
```

Open `http://127.0.0.1:4173`. Put the service behind TLS and an authenticated reverse proxy before exposing it to a network.

Docker runs the same server:

```bash
docker build -t recall-room:0.1.0 .
docker run --rm -p 4173:4173 \
  -e RECALL_ROOM_TOKEN \
  -e RECALL_ROOM_URL \
  -e RECALL_ROOM_WEB_KEY \
  recall-room:0.1.0
```

## Connect the MCP server

The MCP server uses newline-delimited JSON-RPC over stdio and implements `initialize`, `ping`, `tools/list`, and `tools/call`. It advertises four product utilities plus the 9 product action tools with their pinned JSON input schemas.

```json
{
  "mcpServers": {
    "recall-room": {
      "command": "recall-room-mcp",
      "env": {
        "RECALL_ROOM_TOKEN": "a-scoped-workspace-token",
        "RECALL_ROOM_URL": "https://cloud.getsupers.com"
      }
    }
  }
}
```

## Self-host the backend

```bash
git clone https://github.com/rohanarun/managed-oss-cloud.git
cd managed-oss-cloud
git checkout v0.4.2
# Follow that repository's PostgreSQL, migration, TLS, and runtime instructions.
```

Then point `RECALL_ROOM_URL` at the self-hosted control-plane origin. All products may share the same backend and PostgreSQL cluster while the backend preserves workspace and module boundaries.

## Typed action catalogue

| Action ID | Capability | Token scope | Operation |
|---|---|---|---|
| `meeting-create` | Create meeting ledger | `write` | `command` |
| `participant-add` | Add attributed participant | `write` | `command` |
| `transcript-append` | Append transcript evidence | `write` | `command` |
| `decision-record` | Record human decision | `write` | `command` |
| `action-item-create` | Create owned action item | `write` | `command` |
| `summary-propose` | Queue cited meeting summary | `ai` | `ai` |
| `followup-propose` | Queue follow-up proposal | `ai` | `ai` |
| `transcript-redact` | Redact transcript segment | `write` | `command` |
| `meeting-export` | Freeze private meeting export | `write` | `command` |

The complete machine-readable module definition, JSON input schemas, MCP tool names, risk metadata, examples, and release provenance are pinned in [product-manifest.json](./product-manifest.json).

## Clean-room statement

Original clean-room implementation of the meeting decisions and commitments software category, designed and written independently. Public category behavior informed the requirements, but the product name, implementation, UI, CLI, MCP surface, tests, and documentation in this repository are original. No third-party product source code, assets, copied interface, trademarks, or branding are included.

## Security

- Use a distinct, least-privilege workspace API token per deployment.
- Never place the API token in browser code, Git history, container images, or logs.
- Keep the web server on loopback unless it is behind TLS and authentication.
- Rotate a token immediately if it is exposed.
- Treat AI output as a proposal unless the action contract explicitly records approval and execution boundaries.

See [SECURITY.md](./SECURITY.md) for vulnerability reporting and the trust boundary.

## Development

```bash
npm test
npm run verify
npm pack --dry-run
```

The tests run against a fake API and prove bearer authentication, fixed module routing, input validation, CLI execution, stdio MCP discovery/calls, web-key protection, and server-side token handling.

## License

[MIT](./LICENSE)
