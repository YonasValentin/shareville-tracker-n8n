# Testing Patterns

**Analysis Date:** 2026-01-18

## Test Framework

**Status: No Automated Testing**

This is an n8n workflow project. There are:
- No test files (`.test.*`, `.spec.*`)
- No test framework configuration
- No CI/CD pipeline
- No unit tests for embedded JavaScript

**Testing is manual** through the n8n UI.

## Manual Testing Approach

### n8n Built-in Testing

**Test Workflow Button:**
```
Location: n8n UI → Open Workflow → Click "Test Workflow" (play button)
```

**What it does:**
1. Executes the entire workflow once
2. Shows real-time execution progress through nodes
3. Displays output data at each node
4. Captures errors with stack traces

### Node-Level Testing

**Execute single node:**
1. Click on any node in the workflow
2. Click "Execute node" in the node panel
3. Uses pinned data or fetches new data

**Pin test data:**
1. Execute workflow once
2. Right-click on node output
3. Select "Pin data"
4. Future tests use pinned data instead of live API calls

## Test Data Management

### Static Data Testing

**Problem:** Workflow uses `$getWorkflowStaticData('global')` for state persistence

**Manual reset:**
```
n8n UI → Settings → Static Data → Clear
```

**State keys used:**
- `shareville-workflow.json`: `lastSeen` (trade deduplication)
- `portfolio-news-workflow.json`: `seenNews` (news deduplication)

### API Response Testing

**Yahoo Finance responses** can fail due to:
- Rate limiting
- Cookie/crumb expiration
- Regional restrictions

**Current mitigation:**
```json
"options": {
  "response": {
    "response": {
      "neverError": true
    }
  }
}
```

Workflows continue execution even on API errors.

## Recommended Testing Scenarios

### Trader Tracker Workflow

| Test Case | How to Verify |
|-----------|---------------|
| New trade detection | Clear static data, run workflow, check Discord |
| Duplicate prevention | Run workflow twice, second run sends no alerts |
| API failure handling | Disconnect network, verify workflow completes |
| Empty feed handling | Test with inactive trader slug |
| BUY vs SELL formatting | Check Discord embed colors (green vs red) |

### Portfolio News Workflow

| Test Case | How to Verify |
|-----------|---------------|
| News detection | Clear static data, run workflow, check Discord |
| No news case | Use stock with no recent news |
| Duplicate news filtering | Run twice, second run sends no alerts |
| Multiple stocks | Verify all portfolio stocks are processed |

## Code Node Testing

### Isolation Testing

**Approach:** Copy JavaScript to standalone Node.js for testing

**Example: Extract and test `generateAnalysis` function**
```javascript
// test-analysis.js (not in codebase, but recommended)
function generateAnalysis(data) {
  // ... copy function from Format Discord Message node ...
}

// Test cases
const testData = {
  trailingPE: '15.5',
  forwardPE: '12.3',
  profitMargin: '25%',
  recommendationKey: 'buy',
  // ...
};

console.log(generateAnalysis(testData));
```

### Mock Data Patterns

**Trade object structure:**
```javascript
const mockTrade = {
  id: 'test-123',
  type: 'TRADE',
  tradeType: 'BUY',
  postedAt: '2026-01-18T12:00:00Z',
  instrument: {
    id: 'inst-1',
    name: 'Test Stock',
    legacyInstrumentId: '12345',
    logoUrl: 'https://example.com/logo.png'
  },
  price: 100.50,
  currency: 'DKK',
  percentageChange: 5.2
};
```

**Yahoo Finance response structure:**
```javascript
const mockYahooSearch = {
  quotes: [
    {
      symbol: 'TEST.CO',
      exchange: 'CPH',
      exchDisp: 'Copenhagen',
      sectorDisp: 'Technology',
      industryDisp: 'Software'
    }
  ],
  news: [
    {
      title: 'Test News Article',
      link: 'https://example.com/article',
      publisher: 'Test Publisher',
      uuid: 'news-uuid-123'
    }
  ]
};
```

## Coverage Gaps

### No Automated Tests For:
- JavaScript logic in Code nodes
- Data transformation correctness
- Embed formatting validation
- State management edge cases
- API response parsing

### Recommended Test Additions:
1. Extract JavaScript functions to separate `.js` files
2. Add Jest/Vitest tests for extracted functions
3. Create n8n test data fixtures
4. Document test scenarios in workflow descriptions

## Deployment Testing

### Docker Health Check

**Defined in `compose.yaml`:**
```yaml
healthcheck:
  test: ["CMD", "wget", "-q", "--spider", "http://localhost:5678/healthz"]
  interval: 30s
  timeout: 10s
  retries: 3
```

**Manual verification:**
```bash
curl http://192.168.0.25:5678/healthz
```

### Workflow Import Testing

**After importing workflow JSON:**
1. Open workflow in n8n
2. Verify all nodes are connected
3. Check no credential errors (webhooks are hardcoded)
4. Run test execution
5. Verify Discord messages received

## Debugging Tools

### n8n Execution History

**Location:** n8n UI → Executions (sidebar)

**Provides:**
- Execution timestamps
- Success/failure status
- Full data at each node
- Error messages with stack traces
- Retry capability

### Container Logs

```bash
# Via Dockge UI or SSH
docker logs n8n --tail 100 -f
```

### Workflow Debug Mode

**Enable verbose output:**
1. Click node
2. View input/output data
3. Check "Always Output Data" for debugging empty returns

## Test Commands Summary

**No automated test commands exist.** All testing is manual via:

| Action | Location |
|--------|----------|
| Test full workflow | n8n UI → Test Workflow button |
| Test single node | n8n UI → Click node → Execute Node |
| View executions | n8n UI → Executions sidebar |
| Check health | `curl http://192.168.0.25:5678/healthz` |
| View logs | `docker logs n8n --tail 100 -f` |
| Clear state | n8n UI → Settings → Static Data → Clear |

---

*Testing analysis: 2026-01-18*
