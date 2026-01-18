# Requirements

## v1 Requirements

### News Source Routing

- [x] **NEWS-01**: Detect stock currency (SEK vs other) from Shareville instrument data
- [x] **NEWS-02**: Route SEK stocks to MFN.se news fetcher
- [x] **NEWS-03**: Route non-SEK stocks to Yahoo Finance (existing)
- [x] **NEWS-04**: Merge news from both sources into unified format

### MFN.se Integration

- [x] **MFN-01**: Fetch MFN.se page for Swedish stock by company name
- [x] **MFN-02**: Parse HTML to extract press release headlines
- [x] **MFN-03**: Extract publication dates for each headline
- [x] **MFN-04**: Extract links to full press releases
- [x] **MFN-05**: Handle MFN.se errors gracefully (return empty, don't fail workflow)

### State Management

- [x] **STATE-01**: Track seen MFN.se articles (prevent duplicates)
- [x] **STATE-02**: Clean up old seen articles (7-day retention)

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
| NEWS-01 | 1 | ✅ Complete |
| NEWS-02 | 1 | ✅ Complete |
| NEWS-03 | 1 | ✅ Complete |
| NEWS-04 | 1 | ✅ Complete |
| MFN-01 | 1 | ✅ Complete |
| MFN-02 | 1 | ✅ Complete |
| MFN-03 | 1 | ✅ Complete |
| MFN-04 | 1 | ✅ Complete |
| MFN-05 | 1 | ✅ Complete |
| STATE-01 | 1 | ✅ Complete |
| STATE-02 | 1 | ✅ Complete |

---
*Last updated: 2026-01-18 - All requirements complete*
