# Codebase Concerns

**Analysis Date:** 2026-01-18

## Tech Debt

**Hardcoded Trader List:**
- Issue: Trader list is embedded directly in workflow JSON code node
- Files: `shareville-workflow.json` (line 23, "Trader List" node)
- Impact: Requires workflow modification to add/remove traders
- Fix approach: Move to n8n credentials/environment or external config endpoint

**Hardcoded Profile Slug:**
- Issue: Portfolio news workflow has hardcoded profile `yonas-v`
- Files: `portfolio-news-workflow.json` (line 24, "Get My Profile" node)
- Impact: Requires workflow modification to change monitored profile
- Fix approach: Use n8n variable or environment variable

**Duplicate State Management Logic:**
- Issue: Both workflows implement nearly identical deduplication logic
- Files: `shareville-workflow.json` (lines 79-86, "Check If New" node), `portfolio-news-workflow.json` (lines 106-113, "Filter New News" node)
- Impact: Changes need to be made in two places; risk of drift
- Fix approach: Extract to shared Code node or custom n8n node

**Large Monolithic Code Nodes:**
- Issue: "Format Discord Message" node contains ~200 lines of JavaScript
- Files: `shareville-workflow.json` (lines 237-244)
- Impact: Difficult to test, debug, and maintain
- Fix approach: Split into multiple Code nodes or extract helper functions

## Known Bugs

**Yahoo Finance Cookie/Crumb May Fail Silently:**
- Symptoms: Financial data returns null for all fields, Discord embed shows minimal info
- Files: `shareville-workflow.json` (lines 147-183, "Get Yahoo Cookie" and "Get Yahoo Crumb" nodes)
- Trigger: Yahoo Finance changes authentication mechanism or rate limits
- Workaround: `neverError: true` prevents workflow failure but masks the issue

**Symbol Mismatch for Nordic Stocks:**
- Symptoms: Wrong financial data displayed (incorrect company)
- Files: `shareville-workflow.json` (lines 117, "Process Yahoo Search" node)
- Trigger: Stock name ambiguity - e.g., "Novo Nordisk" matches multiple tickers
- Workaround: Nordic exchanges prioritized but not guaranteed

## Security Considerations

**CRITICAL - Discord Webhook URL Exposed:**
- Risk: Webhook URL hardcoded in plain text allows anyone with repo access to spam the Discord channel
- Files: `shareville-workflow.json` (line 249), `portfolio-news-workflow.json` (line 128)
- Current mitigation: None
- Recommendations:
  1. Move webhook URL to n8n credentials (HTTP Header Auth or custom credential type)
  2. Regenerate webhook URL after moving to credentials
  3. Add `.gitignore` for any future sensitive config

**Unauthenticated API Access:**
- Risk: Shareville and Yahoo APIs called without authentication could be rate limited or blocked
- Files: All HTTP Request nodes
- Current mitigation: User-Agent header set for Yahoo requests
- Recommendations: Consider official API access if available; add rate limiting between requests

**No Input Validation:**
- Risk: Malformed API responses could cause unexpected behavior
- Files: All Code nodes that process API responses
- Current mitigation: Optional chaining (`?.`) used in some places
- Recommendations: Add explicit validation for required fields; add error handling

**HTTP Deployment (Not HTTPS):**
- Risk: n8n admin interface accessible over unencrypted HTTP
- Files: `compose.yaml` (lines 17-19, 29)
- Current mitigation: Local network only (192.168.0.25)
- Recommendations: Consider reverse proxy with SSL for any remote access

## Performance Bottlenecks

**Sequential API Calls for Each Trader:**
- Problem: 5 traders = 10+ sequential HTTP requests per run
- Files: `shareville-workflow.json` workflow connections
- Cause: n8n processes items sequentially through HTTP nodes by default
- Improvement path: Use "Batch" mode on HTTP nodes or split into parallel branches

**Yahoo Finance Rate Limiting:**
- Problem: Multiple Yahoo API calls per trade (search, cookie, crumb, financials)
- Files: `shareville-workflow.json` (nodes: "Yahoo Finance Search", "Get Yahoo Cookie", "Get Yahoo Crumb", "Get Financial Data")
- Cause: Four HTTP requests per new trade
- Improvement path: Cache crumb/cookie pair; batch symbol lookups

**Unbounded State Growth:**
- Problem: State cleanup only removes entries older than 7 days
- Files: `shareville-workflow.json` (line 80), `portfolio-news-workflow.json` (line 107)
- Cause: High-volume traders could accumulate thousands of entries
- Improvement path: Add maximum entry limit or more aggressive cleanup

## Fragile Areas

**Yahoo Finance Integration:**
- Files: `shareville-workflow.json` (lines 88-234)
- Why fragile: Unofficial API; authentication via cookie/crumb scraping
- Safe modification: Test changes thoroughly; `neverError: true` masks failures
- Test coverage: None - manual testing only

**Shareville API:**
- Files: All HTTP nodes calling `api.prod.nntech.io`
- Why fragile: Undocumented internal API; may change without notice
- Safe modification: Monitor for API changes; log response structures
- Test coverage: None

**Discord Embed Formatting:**
- Files: `shareville-workflow.json` "Format Discord Message" node
- Why fragile: Complex string building with many conditionals
- Safe modification: Test all code paths (buy/sell, with/without Yahoo data)
- Test coverage: None

## Scaling Limits

**State Storage:**
- Current capacity: Unknown - stored in n8n internal database
- Limit: Large state objects may slow workflow execution
- Scaling path: Use external Redis/database for state; implement pagination

**Number of Traders:**
- Current capacity: 5 traders
- Limit: Linear increase in API calls and execution time
- Scaling path: Parallel execution; rate limiting; batch requests

**Poll Frequency:**
- Current: 5 minutes (trader), 30 minutes (news)
- Limit: Higher frequency may hit API rate limits
- Scaling path: Webhook-based updates if Shareville supports them

## Dependencies at Risk

**Yahoo Finance Unofficial API:**
- Risk: Unofficial; no SLA; may break anytime
- Impact: All financial data and analysis features would fail
- Migration plan: Consider alternative data providers (Alpha Vantage, Finnhub, EOD Historical Data)

**Shareville API:**
- Risk: Internal Nordnet API; not intended for third-party use
- Impact: Entire workflow fails if API changes or access is blocked
- Migration plan: None - core dependency with no alternative

**Discord Webhook:**
- Risk: Discord may change webhook format or rate limits
- Impact: Alerts stop working
- Migration plan: Add alternative notification channels (Telegram, Slack, email)

## Missing Critical Features

**Error Alerting:**
- Problem: Workflow failures go unnoticed
- Blocks: Reliable monitoring - don't know when things break

**Retry Logic:**
- Problem: Transient API failures cause missed trades
- Blocks: Reliability - single failure loses data permanently

**Logging/Audit Trail:**
- Problem: No persistent log of processed trades
- Blocks: Debugging; understanding what was/wasn't sent

**Health Monitoring:**
- Problem: No way to verify workflows are running correctly
- Blocks: Proactive issue detection

## Test Coverage Gaps

**All Code Nodes Untested:**
- What's not tested: JavaScript logic in all Code nodes
- Files: All `.json` workflow files
- Risk: Logic errors, edge cases, null handling issues
- Priority: High - core business logic

**API Response Handling:**
- What's not tested: Behavior with malformed/empty API responses
- Files: All HTTP Request and subsequent Code nodes
- Risk: Workflow crashes or silent failures
- Priority: Medium - defensive coding needed

**State Management:**
- What's not tested: Deduplication logic, cleanup behavior
- Files: "Check If New" and "Filter New News" Code nodes
- Risk: Duplicate alerts or missed alerts
- Priority: High - core deduplication functionality

---

*Concerns audit: 2026-01-18*
