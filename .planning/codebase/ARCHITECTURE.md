# Architecture

**Analysis Date:** 2026-01-18

## Pattern Overview

**Overall:** Event-Driven Pipeline Architecture (n8n Workflow Automation)

**Key Characteristics:**
- No custom code repository - workflows defined as JSON configurations
- n8n orchestrates data flow through sequential node execution
- External API integration pattern with data enrichment stages
- State management via n8n's built-in staticData mechanism
- Push-based notifications to Discord

## Layers

**Trigger Layer:**
- Purpose: Initiate workflow execution on schedule
- Location: Schedule Trigger nodes in workflow JSON files
- Contains: Cron-like interval configurations
- Depends on: n8n runtime scheduler
- Used by: Entire workflow pipeline

**Data Acquisition Layer:**
- Purpose: Fetch raw data from external APIs
- Location: HTTP Request nodes calling Shareville API
- Contains: Profile lookups, activity feed fetches
- Depends on: External API availability
- Used by: Processing layer

**Data Enrichment Layer:**
- Purpose: Augment trade data with market context
- Location: Yahoo Finance API calls and processing nodes
- Contains: Ticker search, financial data, news aggregation
- Depends on: Data Acquisition output
- Used by: Analysis layer

**Processing/Analysis Layer:**
- Purpose: Filter, deduplicate, and analyze data
- Location: Code nodes with embedded JavaScript
- Contains: State checks, investment analysis logic
- Depends on: Enriched data from previous layers
- Used by: Output layer

**Output Layer:**
- Purpose: Format and deliver notifications
- Location: Discord webhook nodes
- Contains: Embed formatting, HTTP POST to Discord
- Depends on: Processed and formatted data
- Used by: End users (Discord recipients)

## Data Flow

**Trader Tracking Flow:**

1. Schedule Trigger fires every 5 minutes
2. Trader List node emits 5 trader objects (slug, name, focus)
3. For each trader: Get User ID from Shareville API by slug
4. Get Activity Feed for user ID (limit 50 trades)
5. Filter to TRADE events only
6. Check state (staticData) to identify new trades
7. If new: Search Yahoo Finance for ticker
8. Get Yahoo cookie/crumb for authenticated API access
9. Fetch financial data (P/E, margins, analyst ratings)
10. Generate investment analysis (bullish/bearish signals)
11. Format Discord embed with all data
12. POST to Discord webhook

**Portfolio News Flow:**

1. Schedule Trigger fires every 30 minutes
2. Get profile by slug (yonas-v)
3. Get activity feed (limit 100)
4. Extract unique instruments from trades
5. For each stock: Search Yahoo Finance for news
6. Filter to unseen news (state check)
7. Format Discord embed with news links
8. POST to Discord webhook

**State Management:**
- Uses n8n's `$getWorkflowStaticData('global')`
- Persists seen trade IDs and news article IDs
- Auto-cleanup of entries older than 7 days
- Prevents duplicate notifications across workflow runs

## Key Abstractions

**Trader Configuration:**
- Purpose: Define monitored Shareville users
- Examples: `shareville-workflow.json` Trader List node
- Pattern: Hardcoded array in Code node JavaScript
```javascript
const traders = [
  { slug: 'bagnis', name: 'Bagnis', focus: 'Scalping/swing' },
  { slug: 'mingus', name: 'Mingus', focus: 'Nordic defense' },
  // ...
];
```

**Trade Object:**
- Purpose: Normalized trade data with enrichments
- Pattern: Accumulated via spread operators through pipeline
- Contains: Shareville trade data + Yahoo financial data + analysis

**Discord Embed:**
- Purpose: Formatted notification payload
- Pattern: JSON object matching Discord API embed schema
- Contains: Title, description, fields, color, thumbnail, footer

## Entry Points

**Shareville Workflow:**
- Location: `shareville-workflow.json`
- Triggers: Schedule (every 5 minutes)
- Responsibilities: Track trader activity, send trade alerts

**Portfolio News Workflow:**
- Location: `portfolio-news-workflow.json`
- Triggers: Schedule (every 30 minutes)
- Responsibilities: Monitor news for owned stocks

**Docker Deployment:**
- Location: `compose.yaml`
- Triggers: Manual deployment via Dockge
- Responsibilities: Run n8n container with persistent storage

## Error Handling

**Strategy:** Fail-safe with `neverError: true`

**Patterns:**
- HTTP nodes configured with `neverError: true` to continue on API failures
- Null-coalescing throughout Code nodes (`|| null`, `|| []`)
- Graceful degradation - missing Yahoo data does not block trade alerts
- Empty array returns in Code nodes to skip processing when no data

## Cross-Cutting Concerns

**Logging:** n8n execution history provides built-in logging; no custom logging
**Validation:** Minimal - trusts API responses, uses optional chaining
**Authentication:** Yahoo Finance requires cookie/crumb flow; Shareville API is public
**Rate Limiting:** Implicit via schedule intervals (5 min / 30 min)

---

*Architecture analysis: 2026-01-18*
