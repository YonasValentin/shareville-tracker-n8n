# Shareville Discord Tracker

Automated Discord alerts when tracked Nordnet/Shareville traders make trades.

## Files

```
shareville-tracker/
├── compose.yaml              # Docker Compose for n8n
├── shareville-workflow.json  # n8n workflow to import
└── README.md                 # This file
```

## Setup Steps

### 1. Create Discord Webhook

1. Open Discord and go to your server
2. **Server Settings** → **Integrations** → **Webhooks**
3. Click **New Webhook**
4. Name it `Shareville Alerts`
5. Select the channel for alerts
6. Click **Copy Webhook URL**
7. Save this URL - you'll need it in step 4

### 2. Deploy n8n via Dockge

1. Open Dockge: `http://192.168.0.25:5001`
2. Click **+ Compose** (create new stack)
3. Name the stack: `n8n`
4. Paste the contents of `compose.yaml`
5. Click **Deploy**
6. Wait for container to start (check status shows "running")

### 3. Access n8n

1. Open: `http://192.168.0.25:5678`
2. Create your admin account (first-time setup)
3. Complete the onboarding wizard

### 4. Import Workflow

1. In n8n, go to **Workflows** → **Import from File**
2. Select `shareville-workflow.json`
3. Open the imported workflow

### 5. Configure Discord Webhook

Since n8n's Discord node needs credentials, we use a simpler HTTP approach:

1. Click on the **Send to Discord** node
2. In the URL field, replace `{{ $credentials.webhookUrl }}` with your actual Discord webhook URL:
   ```
   https://discord.com/api/webhooks/YOUR_ID/YOUR_TOKEN
   ```
3. Save the node

### 6. Test the Workflow

1. Click **Test Workflow** button (play icon)
2. Watch the execution progress through nodes
3. Check your Discord channel for test messages

**Note:** On first run, you'll get alerts for recent trades. After that, the state management will only alert on new trades.

### 7. Activate the Workflow

1. Toggle the **Active** switch (top right)
2. The workflow will now run every 5 minutes automatically

## Tracked Traders

| Slug | Name | Focus | 3yr Return |
|------|------|-------|------------|
| bagnis | Bagnis | Scalping/swing, PixelFox | +156,696% |
| mingus | Mingus | Nordic defense (Nanexa, Clavister) | +453% |
| wajme | Wajme | Concentrated tech (Palantir 74%) | +558% |
| wake76 | Wake76 | Momentum (Rheinmetall, NVDA) | +156% |
| mahisse | Mahisse | Long-term value (Mærsk, AMD) | +174% |

## Modifying Traders

To add/remove traders, edit the **Trader List** node:

```json
[
  { "slug": "bagnis", "name": "Bagnis", "focus": "Scalping/swing" },
  { "slug": "new-trader", "name": "New Trader", "focus": "Their focus" }
]
```

## Discord Message Format

```
🟢 **BUY**: Clavister Holding
👤 **Trader:** Mingus
💼 *Nordic defense*
💰 **Price:** 2.53 SEK
🔗 View on Shareville
```

## Troubleshooting

### No messages appearing
- Check workflow is **Active** (green toggle)
- Verify Discord webhook URL is correct
- Test the workflow manually

### Too many alerts on first run
- This is normal - it catches recent trades
- After first run, only new trades trigger alerts

### Rate limiting
- Shareville API: No known limits for this usage
- Discord: 30 requests/minute per webhook

### Viewing logs
```bash
# SSH to Pi, then:
docker logs n8n --tail 100 -f
```

## Architecture

```
┌─────────────────┐     ┌──────────────────────────────────┐     ┌─────────────┐
│  Cron Trigger   │────▶│  n8n Workflow                    │────▶│  Discord    │
│  (every 5 min)  │     │  - Fetch trader profiles         │     │  Webhook    │
└─────────────────┘     │  - Get activity feeds            │     └─────────────┘
                        │  - Filter for trades             │
                        │  - Compare with saved state      │
                        │  - Format and send alerts        │
                        └──────────────────────────────────┘
```

## Resource Usage

- **RAM:** ~200-300MB for n8n container
- **CPU:** Minimal (brief spikes every 5 min)
- **Disk:** ~50MB for n8n + workflow data
