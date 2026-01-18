# Shareville Discord Tracker

Automated Discord alerts when tracked Nordnet/Shareville traders make trades. Includes Yahoo Finance integration for market data.

## Features

- Tracks 5 Shareville traders for new trades
- Rich Discord embeds with stock info
- Yahoo Finance integration (current price, 52-week range, sector)
- Related news when available
- State management to prevent duplicate alerts

## Files

```
shareville-tracker-n8n/
├── compose.yaml              # Docker Compose for n8n (official config)
├── shareville-workflow.json  # n8n workflow to import
└── README.md                 # This file
```

## Deployment via Dockge

Based on official [n8n Docker docs](https://docs.n8n.io/hosting/installation/docker/) and [Dockge docs](https://github.com/louislam/dockge).

### Step 1: Create n8n Stack in Dockge

1. Open Dockge: `http://192.168.0.25:5001`
2. Click **+ Compose** (top right)
3. Set stack name: `n8n`
4. Paste the contents of `compose.yaml`:

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - GENERIC_TIMEZONE=Europe/Copenhagen
      - TZ=Europe/Copenhagen
      - N8N_HOST=192.168.0.25
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_RUNNERS_ENABLED=true
      - NODE_ENV=production
    volumes:
      - n8n_data:/home/node/.n8n
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:5678/healthz"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  n8n_data:
```

5. Click **Deploy**
6. Wait for status to show **running** (green)

### Step 2: Setup n8n

1. Open: `http://192.168.0.25:5678`
2. Create your admin account
3. Complete the onboarding wizard

### Step 3: Import Workflow

1. In n8n, click **Workflows** in sidebar
2. Click **⋮** menu → **Import from File**
3. Select `shareville-workflow.json`
4. Open the imported workflow

### Step 4: Test & Activate

1. Click **Test Workflow** (play button at bottom)
2. Watch execution progress through nodes
3. Check Discord for test alerts
4. Toggle **Active** switch (top right) to enable auto-polling

## Discord Alert Format

Alerts appear as rich embeds:

```
┌─────────────────────────────────────────┐
│ 🟢 BOUGHT: Clavister              [logo]│
│ Mingus bought this stock                │
│ 🔗 Nordnet • Yahoo Finance              │
├─────────────────────────────────────────┤
│ 💰 Trade Price  📊 Position P&L  💼 Strategy
│ 3.89 SEK        +15.2%           Nordic defense
│                                         │
│ 💵 Current      📉 52W Range    📍 Position
│ 3.92 SEK        1.89 - 6.24     🟡 47%
│                                         │
│ 🏢 Sector       🔧 Industry     📈 Ticker
│ Technology      Software        CLAV.ST
├─────────────────────────────────────────┤
│ 📰 Related News                         │
│ • Article title here...                 │
├─────────────────────────────────────────┤
│ Trade executed: 15. jan 2026, 17:42     │
└─────────────────────────────────────────┘
```

## Tracked Traders

| Slug | Name | Focus |
|------|------|-------|
| bagnis | Bagnis | Scalping/swing |
| mingus | Mingus | Nordic defense |
| wajme | Wajme | Concentrated tech |
| wake76 | Wake76 | Momentum |
| mahisse | Mahisse | Long-term value |

### Modifying Traders

Edit the **Trader List** node in the workflow:

```json
[
  { "slug": "bagnis", "name": "Bagnis", "focus": "Scalping/swing" },
  { "slug": "new-trader", "name": "New Trader", "focus": "Their strategy" }
]
```

## Troubleshooting

### No messages appearing
- Verify workflow is **Active** (green toggle)
- Check Discord webhook URL in **Send to Discord** node
- Test workflow manually with play button

### Too many alerts on first run
- Normal behavior - catches recent trades
- State management prevents duplicates after first run

### View container logs
```bash
# Via Dockge: Click on n8n stack → Terminal tab
# Or SSH to Pi:
docker logs n8n --tail 100 -f
```

### Restart n8n
In Dockge: Click **Restart** button on n8n stack

## Architecture

```
┌─────────────────┐
│ Schedule Trigger│ (every 5 min)
└────────┬────────┘
         ▼
┌─────────────────┐     ┌──────────────────┐
│ Shareville API  │────▶│ Yahoo Finance API│
│ - Get profiles  │     │ - Search ticker  │
│ - Get trades    │     │ - Get quote data │
└────────┬────────┘     └────────┬─────────┘
         │                       │
         └───────────┬───────────┘
                     ▼
         ┌───────────────────┐
         │ State Management  │
         │ (skip duplicates) │
         └─────────┬─────────┘
                   ▼
         ┌───────────────────┐
         │ Discord Webhook   │
         │ (rich embed)      │
         └───────────────────┘
```

## Resource Usage

- **RAM:** ~200-300MB
- **CPU:** Minimal (spikes every 5 min)
- **Disk:** ~50MB for n8n + data
