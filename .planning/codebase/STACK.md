# Technology Stack

**Analysis Date:** 2026-01-18

## Languages

**Primary:**
- JavaScript (ES6+) - Used within n8n Code nodes for data transformation

**Secondary:**
- JSON - Workflow definitions and configuration
- YAML - Docker Compose deployment configuration

## Runtime

**Environment:**
- n8n (workflow automation platform)
- Node.js (n8n internal runtime)
- Docker container: `docker.n8n.io/n8nio/n8n`

**Package Manager:**
- Not applicable - n8n workflows are self-contained JSON files
- No external dependencies to manage

## Frameworks

**Core:**
- n8n - Low-code workflow automation platform
- Nodes used:
  - `n8n-nodes-base.scheduleTrigger` v1.2 - Cron-based triggers
  - `n8n-nodes-base.code` v2 - JavaScript execution
  - `n8n-nodes-base.httpRequest` v4.2 - REST API calls

**Testing:**
- Manual testing via n8n UI "Test Workflow" button
- No automated testing framework

**Build/Dev:**
- Docker/Docker Compose for deployment
- Dockge for Docker stack management

## Key Dependencies

**Critical:**
- n8n Docker image - Core workflow engine
- n8n static data storage - State persistence for deduplication

**Infrastructure:**
- Docker - Container runtime
- Dockge - Docker management UI (optional)

## Configuration

**Environment:**
- `compose.yaml` - Docker Compose deployment config
- Environment variables in compose file:
  - `GENERIC_TIMEZONE=Europe/Copenhagen`
  - `TZ=Europe/Copenhagen`
  - `N8N_HOST=192.168.0.25`
  - `N8N_PORT=5678`
  - `N8N_PROTOCOL=http`
  - `N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true`
  - `N8N_RUNNERS_ENABLED=true`
  - `NODE_ENV=production`
  - `N8N_SECURE_COOKIE=false`

**Build:**
- No build step - JSON workflows imported directly into n8n
- Workflow files: `shareville-workflow.json`, `portfolio-news-workflow.json`

## Platform Requirements

**Development:**
- n8n instance (local or cloud)
- Access to import JSON workflow files
- Network access to external APIs (Shareville, Yahoo Finance, Discord)

**Production:**
- Docker host (Raspberry Pi or similar)
- Port 5678 exposed for n8n UI
- Persistent volume for n8n data
- Network access to external APIs
- Static IP or DNS for N8N_HOST

## Resource Usage

- **RAM:** ~200-300MB
- **CPU:** Minimal (spikes during workflow execution)
- **Disk:** ~50MB for n8n + workflow data

## n8n Specific Features Used

**State Management:**
- `$getWorkflowStaticData('global')` - Persists seen trade/news IDs
- Automatic cleanup of entries older than 7 days

**Data Flow:**
- `$input.first().json` - Access current node input
- `$('Node Name').first().json` - Reference data from earlier nodes
- `$json` - Current item in expressions

**Execution:**
- Schedule Trigger at 5-minute intervals (trader tracker)
- Schedule Trigger at 30-minute intervals (news alerts)
- Sequential node execution with data transformation

---

*Stack analysis: 2026-01-18*
