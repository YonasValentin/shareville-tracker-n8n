# Requirements

## v1 Requirements

### News Source Routing

- [ ] **NEWS-01**: Detect stock currency (SEK vs other) from Shareville instrument data
- [ ] **NEWS-02**: Route SEK stocks to MFN.se news fetcher
- [ ] **NEWS-03**: Route non-SEK stocks to Yahoo Finance (existing)
- [ ] **NEWS-04**: Merge news from both sources into unified format

### MFN.se Integration

- [ ] **MFN-01**: Fetch MFN.se page for Swedish stock by company name
- [ ] **MFN-02**: Parse HTML to extract press release headlines
- [ ] **MFN-03**: Extract publication dates for each headline
- [ ] **MFN-04**: Extract links to full press releases
- [ ] **MFN-05**: Handle MFN.se errors gracefully (return empty, don't fail workflow)

### State Management

- [ ] **STATE-01**: Track seen MFN.se articles (prevent duplicates)
- [ ] **STATE-02**: Clean up old seen articles (7-day retention)

## v2 Requirements (Deferred)

- [ ] Danish stock news sources (Børsen, etc.)
- [ ] German stock news sources (if Yahoo fails)
- [ ] News sentiment analysis
- [ ] Price change correlation with news

## Out of Scope

- Paid news APIs — MFN.se is free and sufficient
- Real-time feeds — 30-min polling is adequate
- Custom UI — Discord is the interface
- Nordic stocks outside Sweden — Yahoo works for large-caps

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| NEWS-01 | 1 | Pending |
| NEWS-02 | 1 | Pending |
| NEWS-03 | 1 | Pending |
| NEWS-04 | 2 | Pending |
| MFN-01 | 1 | Pending |
| MFN-02 | 1 | Pending |
| MFN-03 | 1 | Pending |
| MFN-04 | 1 | Pending |
| MFN-05 | 2 | Pending |
| STATE-01 | 2 | Pending |
| STATE-02 | 2 | Pending |

---
*Last updated: 2026-01-18*
