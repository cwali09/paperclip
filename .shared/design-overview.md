# Paperclip - Design Overview

Open-source orchestration platform for AI agent companies. Node.js server + React UI that coordinates teams of AI agents with org charts, budgets, governance, and goal alignment.

## Entry Points

- **Web UI:** React SPA (`ui/`) served via Vite, connects to API server
- **API Server:** Node.js (`server/src/`) at `http://localhost:3100`
- **CLI:** `npx paperclipai onboard` (`cli/`) for setup and management
- **Agent heartbeats:** Agents connect via adapters and receive scheduled wake-up calls

## Data Flow

```
User (browser/mobile)
    │
    ▼
React UI (ui/) ──HTTP──> API Server (server/)
                              │
                ┌─────────────┼─────────────────┐
                ▼             ▼                  ▼
         Ticket System   Governance       Heartbeat Scheduler
         (task checkout,  (approvals,      (agent wake cycles,
          delegation)     budgets)          event triggers)
                │             │                  │
                └──────┬──────┘                  │
                       ▼                         ▼
                  PostgreSQL              Agent Adapters (packages/adapters/)
                  (embedded                      │
                   or external)     ┌────────────┼────────────┐
                                    ▼            ▼            ▼
                              OpenClaw     Claude Code    Codex/Cursor/Bash/HTTP
```

## Key Abstractions

| Concept | Description |
|---------|-------------|
| **Company** | Top-level entity with isolated data, goals, org chart, and budget |
| **Org Chart** | Hierarchical agent structure with roles, reporting lines, titles |
| **Agent** | Any runtime (OpenClaw, Claude, Codex, etc.) with a job description and budget |
| **Heartbeat** | Scheduled wake-up cycle — agent checks for work, acts, reports back |
| **Ticket** | Unit of work with full conversation tracing and tool-call audit log |
| **Goal** | Hierarchical objectives — every task traces back to company mission |
| **Governance** | Approval gates, budget enforcement, config versioning with rollback |
| **Adapter** | Runtime-specific integration (packages/adapters/) for each agent type |
| **Plugin** | Third-party extension with manifest, worker, optional UI — sandboxed execution |
| **Plugin SDK** | `@paperclipai/plugin-sdk` — define manifests, tools, UI components, event handlers |
| **Document** | Rich content attached to issues with revision history |

## Monorepo Structure

| Package | Path | Purpose |
|---------|------|---------|
| Server | `server/` | API routes, heartbeat scheduler, governance, plugin runtime |
| UI | `ui/` | React dashboard (Vite, shadcn, plugin slots) |
| CLI | `cli/` | `paperclipai` CLI for onboarding and management |
| DB | `packages/db/` | Drizzle ORM schema, migrations, PostgreSQL |
| Shared | `packages/shared/` | Shared types, utilities, and validators |
| Adapters | `packages/adapters/` | Agent runtime adapters |
| Adapter Utils | `packages/adapter-utils/` | Shared adapter helpers |
| Plugin SDK | `packages/plugins/sdk/` | Plugin authoring SDK (manifest, worker, UI) |
| Plugin Scaffolder | `packages/plugins/create-paperclip-plugin/` | `npx create-paperclip-plugin` |
| Plugin Examples | `packages/plugins/examples/` | Hello world, file browser, kitchen sink |

## Dependencies

- **Runtime:** Node.js 20+, pnpm 9.15+
- **Database:** PostgreSQL (embedded for dev, external for production)
- **ORM:** Drizzle
- **UI:** React, Vite, shadcn/ui
- **Testing:** Vitest, Playwright (e2e)
- **Releases:** Changesets

## Exit Points

- **API responses** — JSON over HTTP to UI and agents
- **Agent commands** — Dispatched via adapters to external agent runtimes
- **Database writes** — PostgreSQL for all persistent state
- **Audit log** — Immutable record of all agent actions and decisions
