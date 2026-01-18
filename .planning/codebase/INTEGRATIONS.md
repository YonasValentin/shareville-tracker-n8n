# External Integrations

**Analysis Date:** 2026-01-18

## APIs & External Services

**Shareville (Nordnet):**
- Base URL: `https://api.prod.nntech.io/shareville/v3`
- Purpose: Fetch trader profiles and activity feeds
- Auth: None (public API)
- Endpoints used:
  - `GET /profiles/slug/{slug}` - Get user profile by slug
  - `GET /profiles/{id}/activity-feed?language=da&limit={n}` - Get trading activity
- Response data: Profile IDs, trade events, instrument details, timestamps

**Yahoo Finance:**
- Purpose: Stock ticker search, financial data, news
- Auth: Cookie + crumb authentication (handled automatically)
- Endpoints used:
  - `GET query1.finance.yahoo.com/v1/finance/search?q={name}` - Search ticker symbols
  - `GET fc.yahoo.com` - Obtain session cookie
  - `GET query2.finance.yahoo.com/v1/test/getcrumb` - Get API crumb token
  - `GET query2.finance.yahoo.com/v10/finance/quoteSummary/{symbol}` - Get financial data
- Required headers: `User-Agent: Mozilla/5.0`
- Data retrieved:
  - Ticker symbols, exchange info, sector/industry
  - P/E ratios, market cap, profit margins
  - ROE, revenue growth, debt ratios
  - Analyst recommendations, price targets
  - 52-week high/low, beta
  - Related news articles

**Discord:**
- Purpose: Send trade and news alerts
- Auth: Webhook URL (includes token)
- Endpoint: `POST https://discord.com/api/webhooks/{id}/{token}`
- Payload format: JSON with `embeds` array
- Embed features used: title, description, color, fields, thumbnail, footer, timestamp

## Data Storage

**Databases:**
- None - No external database

**Internal Storage:**
- n8n workflow static data (persisted in Docker volume)
- Location: `/home/node/.n8n` (inside container)
- Volume: `n8n_data` (Docker named volume)
- Used for: Deduplication state (seen trade IDs, news article IDs)

**File Storage:**
- Local filesystem only (JSON workflow files)

**Caching:**
- n8n workflow static data serves as cache for seen items
- 7-day TTL on cached entries (auto-cleanup)

## Authentication & Identity

**Auth Provider:**
- None required for external APIs
- Discord webhook includes embedded token
- Yahoo Finance uses cookie-based session

**Implementation:**
- Shareville API: Public, no auth
- Yahoo Finance: Cookie/crumb flow within workflow
- Discord: Webhook URL contains authentication

## Monitoring & Observability

**Error Tracking:**
- None configured
- n8n execution history in UI

**Logs:**
- Docker container logs: `docker logs n8n`
- n8n execution history accessible via UI

**Health Check:**
- Docker healthcheck: `wget -q --spider http://localhost:5678/healthz`
- Interval: 30s, Timeout: 10s, Retries: 3

## CI/CD & Deployment

**Hosting:**
- Self-hosted Docker container
- Target: Local network (192.168.0.25)
- Managed via Dockge at `http://192.168.0.25:5001`

**CI Pipeline:**
- None - Manual deployment
- Workflow import via n8n UI

**Deployment Steps:**
1. Create Docker Compose stack in Dockge
2. Deploy container
3. Import workflow JSON files via n8n UI
4. Activate workflows

## Environment Configuration

**Required env vars (in compose.yaml):**
- `GENERIC_TIMEZONE` - Timezone for schedule triggers
- `TZ` - Container timezone
- `N8N_HOST` - Host IP/domain
- `N8N_PORT` - Port number (5678)
- `N8N_PROTOCOL` - http/https

**Hardcoded in workflows:**
- Discord webhook URL (in workflow JSON)
- Shareville profile slug (in workflow JSON)
- Tracked trader list (in workflow JSON)

**Secrets location:**
- Discord webhook URL embedded in workflow files
- No external secrets management

## Webhooks & Callbacks

**Incoming:**
- None - Workflows use schedule triggers, not webhooks

**Outgoing:**
- Discord webhook: `https://discord.com/api/webhooks/1462271025029972116/_pntfPiYeHxMXhiNGPIow50Q2XvT5CpA2d2o8OSCm64Cu4nLt4SYKU43-Z5F3woQFfkA`

## API Rate Limits & Considerations

**Shareville API:**
- No documented rate limits
- Currently fetching every 5 minutes

**Yahoo Finance API:**
- Unofficial API, may have undocumented limits
- Cookie/crumb mechanism required for financial data
- User-Agent header required to avoid blocks

**Discord Webhooks:**
- Rate limit: 30 requests/minute per webhook
- Current usage: ~5 alerts per 5-minute cycle max

## Data Flow Summary

```
Schedule Trigger (5 min)
        |
        v
Shareville API (profiles, trades)
        |
        v
Yahoo Finance API (search -> cookie -> crumb -> financial data)
        |
        v
n8n Static Data (deduplication check)
        |
        v
Discord Webhook (rich embed alerts)
```

---

*Integration audit: 2026-01-18*
