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

## What we actually need (NFL / DFS data)

These are the categories that would replace today's scraping. Verify pricing,
auth, and terms of service before building on any of them.

- **Player projections / ECR** — replacement for the FantasyPros web query
  (e.g. SportsData.io, FantasyNerds, Sportradar, FantasyPros' own API).
- **DraftKings player pool + salaries** — instead of pasting the DK link and
  parsing the CSV/HTML, which breaks every time DK changes their format.
- **Injury news / status** — instead of the auto-download scrape.
- **Open / free options** — `nflverse` data and ESPN's unofficial endpoints
  provide clean JSON for stats and rosters.

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
