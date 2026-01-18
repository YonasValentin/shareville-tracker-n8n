# Shareville Discord Tracker

Automated Discord alerts for Nordnet/Shareville activity. Two workflows:

1. **Trader Tracker** - Alerts when tracked traders make trades
2. **Portfolio News** - Alerts when news is published about stocks you own

## Features

### Trader Tracker (`shareville-workflow.json`)
- Tracks 5 Shareville traders for new trades
- Rich Discord embeds with stock info
- Yahoo Finance integration (current price, 52-week range, sector)
- Investment analysis with bullish/bearish signals
- State management to prevent duplicate alerts

### Portfolio News Alert (`portfolio-news-workflow.json`)
- Reads your portfolio from your Shareville profile
- **Hybrid news sources**: MFN.se for Swedish stocks, Yahoo Finance for others
- Sends Discord alerts when new articles are published
- Runs every 30 minutes
- State management to prevent duplicate news alerts

## Files

```
shareville-tracker-n8n/
├── compose.yaml                  # Docker Compose for n8n
├── shareville-workflow.json      # Trader Tracker workflow
├── portfolio-news-workflow.json  # Portfolio News Alert workflow
└── README.md                     # This file
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

### Step 3: Import Workflows

1. In n8n, click **Workflows** in sidebar
2. Click **⋮** menu → **Import from File**
3. Import both workflow files:
   - `shareville-workflow.json` (Trader Tracker)
   - `portfolio-news-workflow.json` (Portfolio News)
4. Open each imported workflow

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

## Portfolio News Alert

Monitors news for stocks in your Shareville portfolio (`yonas-v`).

### How it Works

1. Fetches your Shareville profile
2. Extracts unique stocks from your trades
3. Routes by currency:
   - **SEK stocks** → Scrapes MFN.se for Swedish press releases
   - **Other stocks** → Searches Yahoo Finance for news
4. Merges news from both sources
5. Filters out already-seen news
6. Sends Discord alerts for new articles

### News Sources

| Currency | Source | Coverage |
|----------|--------|----------|
| SEK | MFN.se | Swedish press releases (Clavister, Nanexa, etc.) |
| USD/EUR/other | Yahoo Finance | US and international news |

**Why MFN.se?** Yahoo Finance returns garbage for Nordic small-cap stocks. MFN.se (Modular Finance News) provides excellent coverage of Swedish press releases.

### Alert Format

**US/International stocks (Yahoo Finance):**
```
┌─────────────────────────────────────────┐
│ 📰 News Alert: NVIDIA              [logo]│
├─────────────────────────────────────────┤
│ You own this stock                      │
│                                         │
│ • NVIDIA Q4 earnings beat estimates...  │
│ • Jensen Huang announces new AI chip... │
│                                         │
│ 🔗 Nordnet • Yahoo Finance              │
│ 📈 Ticker: NVDA  💱 USD                 │
├─────────────────────────────────────────┤
│ Updated: 18. jan 2026, 12:30            │
└─────────────────────────────────────────┘
```

**Swedish stocks (MFN.se):**
```
┌─────────────────────────────────────────┐
│ 🇸🇪 News Alert: Clavister          [logo]│
├─────────────────────────────────────────┤
│ You own this stock                      │
│                                         │
│ • Clavister awarded 280 MSEK...         │
│ • Clavister Q4 report 2025...           │
│                                         │
│ 🔗 Nordnet • MFN.se                     │
│ 💱 SEK                                  │
├─────────────────────────────────────────┤
│ Updated: 18. jan 2026, 12:30            │
└─────────────────────────────────────────┘
```

### Changing Profile

To monitor a different Shareville profile, edit the **Get My Profile** node URL:

```
https://api.prod.nntech.io/shareville/v3/profiles/slug/YOUR-SLUG-HERE
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

### Trader Tracker
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
         └─────────┬─────────┘
                   ▼
         ┌───────────────────┐
         │ Discord Webhook   │
         └───────────────────┘
```

### Portfolio News Alert
```
┌─────────────────┐
│ Schedule Trigger│ (every 30 min)
└────────┬────────┘
         ▼
┌─────────────────┐
│ Shareville API  │
│ - Get profile   │
│ - Get trades    │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Route by        │
│ Currency        │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌──────────┐
│MFN.se │ │Yahoo     │
│(SEK)  │ │(other)   │
└───┬───┘ └────┬─────┘
    │          │
    └────┬─────┘
         ▼
┌─────────────────┐
│ Merge & Filter  │
│ (skip seen)     │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Discord Webhook │
└─────────────────┘
```

## Resource Usage

- **RAM:** ~200-300MB
- **CPU:** Minimal (spikes every 5 min)
- **Disk:** ~50MB for n8n + data
