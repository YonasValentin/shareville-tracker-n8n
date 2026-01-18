# Coding Conventions

**Analysis Date:** 2026-01-18

## Project Type

**n8n Workflow Automation** - This is NOT a traditional codebase. It consists of:
- JSON workflow definition files for n8n
- Embedded JavaScript in n8n Code nodes
- Docker Compose configuration for deployment

## Workflow File Structure

**File Format:** JSON (n8n workflow export format)
**Key Files:**
- `shareville-workflow.json` - Main trader tracking workflow
- `portfolio-news-workflow.json` - Portfolio news alerting workflow

**Workflow JSON Schema:**
```json
{
  "name": "Workflow Name",
  "nodes": [...],
  "connections": {...},
  "settings": { "executionOrder": "v1" },
  "staticData": null,
  "meta": { "instanceId": "..." },
  "pinData": {},
  "versionId": "N",
  "triggerCount": 0
}
```

## Node Naming Conventions

**Pattern:** Title Case with descriptive action
- Trigger nodes: Describe interval (e.g., `Every 5 Minutes`, `Every 30 Minutes`)
- HTTP Request nodes: Verb + Subject (e.g., `Get User ID`, `Get Activity Feed`)
- Code nodes: Action-oriented (e.g., `Filter Trades`, `Check If New`, `Process Yahoo Search`)

**Node ID Pattern:** lowercase-kebab-case
- `schedule-trigger`, `trader-list`, `get-user-id`, `yahoo-search`

**Examples from codebase:**
| Node ID | Display Name |
|---------|--------------|
| `schedule-trigger` | `Every 5 Minutes` |
| `get-user-id` | `Get User ID` |
| `filter-trades` | `Filter Trades` |
| `check-state` | `Check If New` |
| `format-message` | `Format Discord Message` |

## Embedded JavaScript Conventions

**Location:** Inside `jsCode` parameter of Code nodes
**Runtime:** n8n Code node environment (Node.js-based)

### Variable Naming

**Pattern:** camelCase for all variables
```javascript
// Good
const traderInfo = $('Trader List').first().json;
const traderName = traderInfo.name;
const yahooData = $input.first().json;

// Constants
const nordicExchanges = ['STO', 'CPH', 'OSL', 'HEL'];
const oneWeekAgo = Date.now() - (7 * 24 * 60 * 60 * 1000);
```

### Data Access Patterns

**Input access:**
```javascript
// Current node input
const data = $input.first().json;

// Reference specific node by name
const traderInfo = $('Trader List').first().json;
const prevData = $('Process Yahoo Search').first().json;

// Access item in loop
const stockInfo = $('Extract Unique Stocks').item;
```

**Static data (state persistence):**
```javascript
const staticData = $getWorkflowStaticData('global');
const lastSeen = staticData.lastSeen || {};
// ... modify lastSeen ...
staticData.lastSeen = lastSeen;
```

### Return Patterns

**Single item output:**
```javascript
return [{ json: { ...data, newField: value } }];
```

**Multiple items output:**
```javascript
return items.map(item => ({ json: item }));
```

**Filter/skip item:**
```javascript
// Return empty array to stop propagation
if (alreadySeen) {
  return [];
}
```

### Function Definition Style

**Pattern:** Function declarations inside Code nodes
```javascript
function generateAnalysis(data) {
  const signals = [];
  let bullish = 0;
  let bearish = 0;
  // ... logic ...
  return { signals, verdict, verdictColor, bullish, bearish };
}
```

## Error Handling

**HTTP Requests:**
- Use `neverError: true` option to continue workflow on API failures
- Check for null/undefined in processing nodes

```json
"options": {
  "response": {
    "response": {
      "neverError": true
    }
  }
}
```

**Defensive coding in JavaScript:**
```javascript
const feed = $input.first().json.feed || [];
const quotes = yahooData.quotes || [];
const bestMatch = quotes[0];  // May be undefined - handled downstream
```

**Null coalescing for optional fields:**
```javascript
const sector = bestMatch?.sectorDisp || null;
const yahooSymbol = data.yahooSymbol || null;
```

## State Management

**Pattern:** Global workflow static data with cleanup

```javascript
// Read state
const staticData = $getWorkflowStaticData('global');
const lastSeen = staticData.lastSeen || {};

// Check for duplicates
const tradeId = `${slug}-${trade.id || tradeTime}`;
if (lastSeen[tradeId]) {
  return [];
}

// Record as seen
lastSeen[tradeId] = Date.now();
staticData.lastSeen = lastSeen;

// Cleanup old entries (7-day retention)
const oneWeekAgo = Date.now() - (7 * 24 * 60 * 60 * 1000);
for (const key of Object.keys(lastSeen)) {
  if (lastSeen[key] < oneWeekAgo) {
    delete lastSeen[key];
  }
}
```

## API Integration Patterns

### External API Access

**Pattern:** HTTP Request nodes with User-Agent header for web APIs
```json
"headerParameters": {
  "parameters": [
    {
      "name": "User-Agent",
      "value": "Mozilla/5.0"
    }
  ]
}
```

### URL Template Expressions

**Pattern:** n8n expression syntax for dynamic URLs
```
https://api.prod.nntech.io/shareville/v3/profiles/{{ $json.slug }}
https://query1.finance.yahoo.com/v1/finance/search?q={{ encodeURIComponent($json.instrument.name) }}
```

## Discord Embed Formatting

**Pattern:** Structured embed object with consistent fields

```javascript
const embed = {
  title: `${emoji} ${actionWord}: ${instrumentName}`,
  description: description,
  color: 0x22c55e,  // Hex color as integer
  fields: [
    { name: 'Field Name', value: 'Value', inline: true }
  ],
  footer: { text: `Footer text` },
  timestamp: data.postedAt,
  thumbnail: { url: logoUrl }  // Optional
};
```

**Color scheme:**
- Green (BUY): `0x22c55e`
- Red (SELL): `0xef4444`
- Blue (Info): `0x3b82f6`
- Yellow (Caution): `0xeab308`
- Gray (Neutral): `0x6b7280` or `0x808080`

## Configuration Patterns

### Hardcoded Configuration

**Trader list in Code node:**
```javascript
const traders = [
  { slug: 'bagnis', name: 'Bagnis', focus: 'Scalping/swing' },
  { slug: 'mingus', name: 'Mingus', focus: 'Nordic defense' },
  // ...
];
```

### Environment Variables

**Pattern:** Set via Docker Compose, consumed by n8n
- Timezone: `GENERIC_TIMEZONE=Europe/Copenhagen`
- Network: `N8N_HOST=192.168.0.25`

### Webhook URLs

**Pattern:** Hardcoded in HTTP Request nodes (not ideal but current state)
```
https://discord.com/api/webhooks/...
```

## Node Positioning

**Pattern:** Linear left-to-right flow, 200px horizontal spacing
```json
"position": [100, 300]   // Start
"position": [300, 300]   // Step 2
"position": [500, 300]   // Step 3
```

## Comments in Code

**Pattern:** Section headers with equals signs for major blocks
```javascript
// ==================== 10X INVESTOR ANALYSIS ====================
function generateAnalysis(data) {
  // ...
}

// ==================== BUILD DISCORD EMBED ====================
const analysis = data.yahooSymbol ? generateAnalysis(data) : null;
```

**Inline comments for clarity:**
```javascript
// Deduplicate by instrument ID
const uniqueStocks = [...new Map(
  trades.map(t => [t.instrument.id, {...}])
).values()];
```

## Date/Time Handling

**Pattern:** Danish locale for display, ISO 8601 for storage
```javascript
const dateStr = tradeDate.toLocaleDateString('da-DK', {
  day: 'numeric',
  month: 'short',
  year: 'numeric',
  hour: '2-digit',
  minute: '2-digit'
});
```

**Timestamp access:**
```javascript
const tradeTime = new Date(trade.postedAt).getTime();
```

## File Organization

```
shareville-tracker-n8n/
├── compose.yaml                  # Docker deployment config
├── shareville-workflow.json      # Main workflow (15 nodes)
├── portfolio-news-workflow.json  # Secondary workflow (9 nodes)
└── README.md                     # Documentation
```

**Workflow complexity:**
- `shareville-workflow.json`: 15 nodes, complex data enrichment
- `portfolio-news-workflow.json`: 9 nodes, simpler news aggregation

---

*Convention analysis: 2026-01-18*
