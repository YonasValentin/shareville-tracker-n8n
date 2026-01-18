# Shareville Discord Tracker

## What This Is

Automated Discord alerts for Nordnet/Shareville activity. Two n8n workflows that monitor trades and news, then send rich Discord embeds.

## Core Value

**Never miss important news about stocks you own** — especially Nordic small-caps that Yahoo Finance doesn't cover well.

## Problem Statement

The Portfolio News Alert workflow currently uses Yahoo Finance for all stocks. This works great for:
- US stocks (NVIDIA, Coinbase, Palantir)
- Large-cap European stocks (Rheinmetall, SAAB, Renk)

But **fails completely** for Nordic small-caps:
- Clavister → Returns unrelated news (Chile wildfires, soccer)
- Nanexa → No relevant results
- Scandinavian Astor Group → Garbage results

These companies announce press releases through MFN.se (Modular Finance News), which provides excellent coverage of Swedish stocks.

## Solution

Hybrid news fetching based on stock currency:
- **SEK stocks** → Scrape MFN.se for Swedish press releases
- **Other stocks** → Continue using Yahoo Finance

## What We're Building

Enhance the existing `portfolio-news-workflow.json` to:

1. Detect stock currency from Shareville data
2. Route SEK stocks to MFN.se scraper
3. Route other stocks to Yahoo Finance
4. Merge results and send Discord alerts

## Technical Context

- **Platform**: n8n workflow automation
- **Deployment**: Docker on Raspberry Pi (192.168.0.25:5678)
- **Existing workflows**:
  - `shareville-workflow.json` (Trader Tracker)
  - `portfolio-news-workflow.json` (Portfolio News - needs enhancement)
- **Discord**: Webhook-based rich embeds

## Constraints

- Must work within n8n's node-based architecture
- No API keys required (MFN.se is public, Yahoo is unofficial)
- Must handle MFN.se rate limiting gracefully
- State management to prevent duplicate alerts

## Requirements

### Validated

- ✓ Fetch user's Shareville portfolio — existing
- ✓ Extract unique stocks from trades — existing
- ✓ Search Yahoo Finance for news — existing
- ✓ Filter new/unseen news — existing
- ✓ Send Discord alerts with rich embeds — existing

### Active

- [ ] **NEWS-01**: Detect stock currency (SEK vs other) from Shareville data
- [ ] **NEWS-02**: Scrape MFN.se for Swedish stock press releases
- [ ] **NEWS-03**: Parse MFN.se HTML to extract headlines, dates, links
- [ ] **NEWS-04**: Route stocks to correct news source based on currency
- [ ] **NEWS-05**: Merge news from both sources into unified format
- [ ] **NEWS-06**: Handle MFN.se errors gracefully (fallback to Yahoo)

### Out of Scope

- German/Danish-specific news sources — Yahoo works for large-caps
- Real-time websocket feeds — polling every 30min is sufficient
- Paid news APIs (Google News, etc.) — MFN.se is free
- Custom UI dashboard — Discord is the interface

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Use MFN.se for Swedish stocks | Free, excellent coverage, public API/scraping | — Pending |
| Currency-based routing | Shareville provides currency, simple logic | — Pending |
| Scrape vs API | MFN.se API returns null, scraping works | — Pending |

## Success Criteria

1. Swedish stocks (Clavister, Nanexa) get relevant news alerts
2. US/large-cap stocks continue working via Yahoo
3. No duplicate alerts
4. Graceful error handling

---
*Last updated: 2026-01-18 after initialization*
