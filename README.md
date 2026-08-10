# Agent Py

Agent Py is a local-first AIOps workspace for authenticated chat, knowledge-base RAG, intelligent incident diagnosis, and MCP tool management. It combines a Vue 3 frontend, a FastAPI backend, SQLite persistence, Milvus vector search, LangChain/LangGraph orchestration, and the official Tencent Cloud CLS MCP Server for real log access.

The project is designed for local development and controlled operations: user-owned data stays scoped to the authenticated user, credentials live only in ignored local JSON files, and diagnostic workflows are backed by durable jobs and auditable evidence.

## Features

### Workspace, Auth, and Isolation

- User registration, login, logout, session recovery, and current-user lookup.
- Argon2 password hashing with no plaintext password storage.
- User and tenant isolation across chats, messages, prompts, skills, knowledge bases, documents, vectors, indexing jobs, MCP connections, AIOps tasks, evidence, reports, feedback, and tool audits.
- Protected Vue workspace routes for chat, knowledge management, intelligent diagnosis, and MCP management.
- Responsive Chinese-language UI for desktop and narrow screens.
- Global success, info, and error feedback with manual dismissal and auto-hide behavior.

### Streaming Chat and Agent Runtime

- Persistent chat sessions stored in SQLite, with creation, switching, reverse chronological listing, generated titles, clearing, and deletion.
- SSE chat streaming powered by `langchain` `create_agent` and an OpenAI-compatible Qwen/Bailian provider.
- Agent-driven tool use for knowledge retrieval, current time, and user-enabled MCP tools.
- User-managed system prompts for each session.
- Progressive `SKILL.md` support: only skill name and description are injected initially, and full skill content is loaded on demand.
- Per-session memory modes: compress every 30 rounds, compress automatically at 70% context usage, or compress manually.
- Collapsible reasoning context, tool-call status, and tool result summaries.
- Audited tool calls stored in SQLite.
- Feedback on assistant answers and individual knowledge citations.

### Knowledge Base and RAG

- Markdown and PDF uploads with filename, size, MIME type, SHA-256, upload time, and indexing status.
- Duplicate file detection, explicit overwrite, and vector cleanup on deletion.
- Chunking strategies for fixed-size text, Markdown headings, and paragraphs.
- Bounded chunk preview before upload.
- Durable document indexing jobs with queued, running, succeeded, failed, and cancelled states.
- Embeddings written to a Milvus HNSW/COSINE collection with owner, tenant, knowledge base, document, source, and chunking metadata.
- Hybrid retrieval with Milvus vector search, in-memory BM25L keyword search, RRF fusion, and Qwen reranking.
- Explainable citations that include vector rank/similarity, BM25 rank/score, RRF score, rerank rank/score, document source, chunk summary, and metadata.
- A LangChain knowledge retrieval tool that always applies the authenticated user's access scope and returns empty results instead of fabricating citations.

### AIOps Diagnosis

- LangGraph Plan-Execute-Replan workflow: Planner retrieves SOPs, Executor calls real tools, Replanner decides whether to continue, adjust, or generate a report.
- Active alert aggregation from Prometheus v1 and Alertmanager v2.
- Alert-to-diagnosis entry point from the frontend.
- Durable SQLite-backed diagnosis jobs with queueing, execution, cancellation, failure, completion, leases, retries, and recovery.
- Diagnosis SSE events for plans, steps, tool calls, evidence, replanning, reports, completion, and errors.
- Persistent raw inputs, plans, execution steps, tool calls, logs, metrics, alerts, knowledge citations, checkpoints, and Markdown reports.
- Diagnosis history with full evidence chains.
- Successful diagnoses can be promoted into user-owned knowledge-base incident cases.
- Structured feedback for diagnosis steps and final reports.

### MCP and External Systems

- Local execution of the official Tencent Cloud `cls-mcp-server`.
- Real CLS log, alert, metric, and helper tools through SSE; no mock log profile is used.
- User-level MCP connection management for SSE and Streamable HTTP servers.
- Connection checks and real tool discovery from the frontend.
- Shared MCP configuration for chat and AIOps.
- Timeout, retry, duplicate tool-name protection, explicit failure handling, and per-call auditing.

### Platform Foundation

- Durable background job runtime with attempts, leases, heartbeats, retry backoff, timeouts, and cooperative cancellation.
- SQLAlchemy models and repository boundaries around SQLite.
- Alembic-managed database migrations.
- Shared TypeScript API/SSE contracts in `packages/api-contracts`.
- `/health`, `/ready`, `/config/check`, and `/metrics` endpoints.
- Structured request logs with request IDs, paths, status codes, duration, and sensitive-field redaction.
- Local demo tooling for uploading Java ecommerce incident logs, publishing Alertmanager alerts, indexing SOPs, and running the complete alert-to-report-to-case flow.

## Application Routes

| Route | Purpose |
| --- | --- |
| `/login`, `/register` | Authentication |
| `/chat` | Streaming agent chat, prompts, skills, memory modes, citations, and feedback |
| `/knowledge` | Document upload, chunk preview, indexing, retry, details, and deletion |
| `/aiops` | Active alerts, live diagnosis, execution chain, evidence, reports, and case library |
| `/mcp` | MCP connection configuration, enable/disable controls, health checks, and tool discovery |

## Repository Layout

```text
apps/backend/           FastAPI, LangChain/LangGraph, SQLite, Alembic, uv, pytest
apps/frontend/          Vue 3, Vite, TypeScript, Pinia, Vitest
packages/api-contracts/ Shared TypeScript HTTP, error, OpenAPI, and SSE contracts
config/                 Committable templates and ignored local JSON configuration
infra/                  Compose assets for etcd, MinIO, Milvus, Attu, and Alertmanager
scripts/                macOS/Linux and Windows local startup helpers
openspec/               OpenSpec specs, active changes, and archived changes
docs/                   Installation guides, tutorials, operations docs, and OpenSpec wiki
```

Do not create parallel backend packages, frontend applications, or contract directories. Backend code belongs under `apps/backend/src/super_ai/` and should import with `from super_ai...`.

## Prerequisites

Install the platform dependencies for your operating system:

- [macOS setup guide](docs/setup/macos.md)
- [Linux setup guide](docs/setup/linux.md)
- [Windows setup guide](docs/setup/windows.md)

The guides cover Git, Docker, Node/npm, `uv`, and the official `cls-mcp-server`.

## Configuration

Agent Py reads project configuration only from local JSON files. It does not read project settings from `.env` files or machine environment variables.

Create local configuration files from the safe templates:

```bash
cp config/project.template.json config/project.json
cp config/user.project.template.json config/user.project.json
```

- `config/project.json` contains local baseline runtime configuration.
- `config/user.project.json` contains personal model and CLS configuration and overrides the baseline configuration.
- `config/project.template.json` and `config/user.project.template.json` are safe to commit.

The local configuration files are ignored by Git. Never commit API keys, tokens, passwords, CLS credentials, logset IDs, topic IDs, or demo passwords. See [operations and monitoring](docs/operations-and-monitoring.md) for configuration details.

## Local Development

Docker Compose only runs local infrastructure: etcd, MinIO, Milvus, Attu, and Alertmanager. The CLS MCP Server, backend, and frontend run directly on the host.

### One-command Startup

From the repository root on macOS or Linux:

```bash
./scripts/start-local.sh
```

On Windows Command Prompt:

```text
scripts\start-local.bat
```

The startup scripts launch infrastructure containers, prepare dependencies, run SQLite migrations, and start MCP, backend, and frontend processes. Runtime logs are written to `apps/backend/var/`.

### Manual Startup

Install frontend and backend dependencies:

```bash
npm install
cd apps/backend
uv sync
mkdir -p var
uv run alembic upgrade head
```

Start infrastructure from the repository root:

```bash
docker compose -f infra/compose.yaml up -d etcd minio milvus attu alertmanager
```

Start the official CLS MCP Server with the `clsMcpServer` settings from `config/project.json`, then start the backend:

```bash
cd apps/backend
uv run uvicorn super_ai.api.app:create_app --factory --host 127.0.0.1 --port 8000
```

In another terminal, start the frontend:

```bash
cd apps/frontend
npm run dev -- --host 127.0.0.1
```

Local endpoints:

- Frontend: `http://127.0.0.1:5173`
- Backend: `http://127.0.0.1:8000`
- Backend readiness: `http://127.0.0.1:8000/ready`
- CLS MCP SSE: `http://127.0.0.1:3000/sse`
- Alertmanager: `http://127.0.0.1:9093`
- Milvus: `http://127.0.0.1:19530`
- Attu: `http://127.0.0.1:8001`

## Real Logs and Alerts Tutorial

Uploading CLS logs, publishing local Alertmanager alerts, indexing SOPs, and running frontend AIOps diagnosis are explicit demo operations. They are not part of the normal startup path.

Follow the [real logs and alerts tutorial](docs/tutorials/real-log-and-alert.md) to run the full demo flow.

## Validation

Run OpenSpec validation from the repository root:

```bash
openspec validate --all
```

Run backend checks from `apps/backend`:

```bash
uv run ruff check .
uv run pyright
uv run pytest
```

Run frontend checks from `apps/frontend`:

```bash
npm run typecheck
npm run test
npm run build
```

Run shared contract checks from the repository root:

```bash
npm run contracts:typecheck
npm --workspace packages/api-contracts run test
```

## Security Notes

- Keep `config/project.json` and `config/user.project.json` local.
- Do not commit secrets, local databases, runtime logs, build output, caches, or dependency folders.
- Do not use client-provided owner or tenant IDs as authorization authority.
- MCP tools should call real user-enabled connections and must preserve timeout, retry, failure, and audit behavior.
- Knowledge retrieval and diagnosis evidence must come from real indexed documents, tool results, logs, metrics, or alerts.

## License

No license file is currently included. Add one before publishing the project for external reuse.
