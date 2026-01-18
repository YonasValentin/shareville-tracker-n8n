# Roadmap

## Current Milestone: v1.1 - MFN.se News Integration

### Phase Overview

| # | Phase | Goal | Requirements | Status |
|---|-------|------|--------------|--------|
| 1 | MFN.se Scraper | Fetch and parse Swedish press releases | NEWS-01, NEWS-02, MFN-01, MFN-02, MFN-03, MFN-04 | Pending |
| 2 | Hybrid Integration | Route stocks to correct source, merge results | NEWS-03, NEWS-04, MFN-05, STATE-01, STATE-02 | Pending |

---

## Phase 1: MFN.se Scraper

**Goal**: Create n8n nodes that fetch and parse MFN.se press releases for Swedish stocks.

**Requirements Covered**:
- NEWS-01: Detect stock currency
- NEWS-02: Route SEK stocks to MFN.se
- MFN-01: Fetch MFN.se page
- MFN-02: Parse HTML for headlines
- MFN-03: Extract dates
- MFN-04: Extract links

**Success Criteria**:
1. Given stock "Clavister", workflow fetches MFN.se page successfully
2. Headlines are extracted with correct titles (not HTML garbage)
3. Dates are parsed into consistent format
4. Links point to actual press release pages

**Dependencies**: None (greenfield addition)

---

## Phase 2: Hybrid Integration

**Goal**: Integrate MFN.se into existing workflow, route by currency, merge results.

**Requirements Covered**:
- NEWS-03: Route non-SEK to Yahoo
- NEWS-04: Merge news from both sources
- MFN-05: Handle errors gracefully
- STATE-01: Track seen MFN.se articles
- STATE-02: Clean up old articles

**Success Criteria**:
1. SEK stocks (Clavister, Nanexa) get news from MFN.se
2. Non-SEK stocks (NVIDIA, Palantir) continue using Yahoo
3. Discord alerts show news from both sources correctly
4. No duplicate alerts after first run
5. MFN.se errors don't crash the workflow

**Dependencies**: Phase 1 complete

---

## Progress Tracking

- **Total Requirements**: 11
- **Completed**: 0
- **In Progress**: 0
- **Remaining**: 11

---
*Last updated: 2026-01-18*
