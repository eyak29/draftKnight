# API Resources & Modernization Notes

A running record of APIs worth using so we stop relying on fragile web scraping
(FantasyPros logins, DraftKings CSV/HTML changes, manual injury-news pulls).

## The API Mega List

- **Repo:** https://github.com/cporter202/API-mega-list
- A broad, general-purpose collection of APIs across ~17 categories (AI, Agents,
  Automation, Developer Tools, E-commerce, Social Media, Lead Gen, News, Real
  Estate, SEO, Travel, Video, etc.).
- **Note for draftKnight:** it does **not** include sports / fantasy football /
  NFL / DraftKings / player-stats / injury APIs, so it doesn't directly replace
  our current data sources. Useful general reference; keep it bookmarked. The
  AI category could be handy if we ever layer projection modeling or news
  summarization on top.

## Candidate APIs (researched)

What draftKnight needs to replace its scraping: (a) DFS salaries + slates,
(b) player projections / ECR, (c) injury status & news, (d) actual results for
the Performance tab. Here's how the main options stack up.

| API | DK salaries + slates | Projections | Injuries/news | Actual results | Format | Pricing | Fit |
|---|---|---|---|---|---|---|---|
| **FantasyNerds** | ✅ DK/FD/Yahoo + slate IDs (wk 1-18) | ✅ proj pts + "Bang for Buck" | ✅ | partial | REST JSON | **$199.95/yr** per sport | **Best overall fit** |
| **SportsData.io** | ✅ DK/FD/Yahoo + slates + ownership % | ✅ BAKER engine, all positions | ✅ | ✅ live + final | REST JSON | Sales quote (enterprise; free trial tier) | Most complete, likely priciest |
| **FantasyPros API** | ❌ | ✅ ECR (what we use today) | ✅ news/notes | partial | REST JSON | Partner/enterprise — contact sales | Matches current data, but API access is gated |
| **ESPN (unofficial)** | ❌ | ✅ team/player projections | ✅ | ✅ | JSON | **Free, no key** | Great free supplement; can break w/o notice |
| **nflverse / nfl_data_py** | ❌ | ❌ (historical only) | rosters | ✅ pbp/weekly/seasonal | CSV/parquet | **Free, open source** | Ideal for Performance/results tracking |

### Notes on each

- **FantasyNerds** — The standout. Its API directly returns DraftKings (plus
  FanDuel/Yahoo) **salaries, projected points, and DFS slate IDs**, which is
  almost exactly what draftKnight scrapes today, but as clean JSON. Flat
  $199.95/yr, self-serve. Docs: https://api.fantasynerds.com/docs/nfl
- **SportsData.io** — Enterprise-grade. BAKER projection engine, DK/FD/Yahoo
  salaries + slates, **DFS ownership projections**, injuries, live & final
  scoring. The most complete, but pricing is a sales quote (likely well above
  hobby budget). Has a free trial tier with limited/delayed data.
  Sales: sales@sportsdata.io
- **FantasyPros API** — This is the source we already use for ECR. The official
  API is a partner/enterprise arrangement (contact via Partners HQ); the
  $3.99-$22.99/mo "Premium" plans are website access, not API. Community R
  package `ffpros` exists for lighter scraping.
- **ESPN unofficial endpoints** — Free, no API key, JSON GET requests. Provides
  projections, rosters, injuries, news. Undocumented and can change without
  notice, so needs robust error handling. Good zero-cost supplement.
  Endpoint lists: https://gist.github.com/nntrn/ee26cb2a0716de0947a0a4e9a157bc1c
- **nflverse** (`nflfastR` / `nfl_data_py` / `nflreadr`) — Free, open source,
  updated nightly in season. Play-by-play back to 1999, weekly/seasonal stats,
  rosters, schedules, and **ID mappings across sites** (useful for our
  recurring player-name-mismatch bugs). No forward projections and no DFS
  salaries, but ideal for the **actual-points / Performance** tracking feature.

## Recommendation

1. **FantasyNerds** as the primary data source — it covers salaries +
   projections + slates in one self-serve, affordable JSON API, replacing the
   most fragile scraping (DK import + FantasyPros login).
2. **ESPN unofficial** + **nflverse** as free supplements for injuries/news and
   actual results, and for ID mapping to kill the name-mismatch bugs.
3. Treat **SportsData.io** as the upgrade path if this ever needs ownership
   projections or enterprise reliability.

## Why this matters (from our release history)

Most past fixes were scraping breakage, not features:

- FantasyPros DFS paywall broke projections (v1.7).
- DK changed their CSV format, breaking imports (v1.7).
- Painful FantasyPros-login-through-Excel flow (v1.5).
- Recurring player name-mismatch fixes (Trubisky, Mahomes, Mike Thomas).

Clean JSON APIs would eliminate most of these classes of bugs.

## Open questions / next steps

- [ ] Decide: keep patching the VBA/Excel tool, or move the data layer to
      something scriptable that can call these APIs.
- [ ] Shortlist 2-3 concrete sports APIs and compare pricing + coverage.
- [ ] Prototype one API call (projections or salaries) as a proof of concept.
