# Release Radar

A full-stack web app for tracking upcoming release dates of **games, movies, and TV seasons**.
A user searches the catalog, adds items to their personal tracks, and gets notified on the day
an item releases.

This repository is built during a one-week Revature FDE full-stack SWE sprint. It is a
**fundamentals** project — hand-written and defended, not AI-generated. Claude acts as a coach
(guides, questions, reviews) and never writes the solution code.

## What it does

- **Accounts** — register / log in (passwords stored hashed, never plaintext).
- **Search** — find media by release window ("what releases in July?") or find a TV series by
  name and drill into its seasons.
- **Track** — add any media item to your tracks; each track records when you added it and whether
  you've been notified.
- **Notify** — on an item's release date, notify the users tracking it (background/scheduled job).
- **Browse** — each media item has a detail page: description, images/trailers, and clickable
  tags that link to every other item sharing that tag.

## Data model

14 tables. Full schema in [`docs/erd.dbml`](docs/erd.dbml) (diagram: [`docs/erd.png`](docs/erd.png)).

Key design decisions:

- **`media_item` parent (class-table inheritance)** — `game`, `movie`, and `season` share a
  parent holding common columns (`title`, `release_date`, `genre`, `description`) and each child
  holds only type-specific columns. This gives every downstream link (tracks, tags, companies,
  assets) **one** table to reference with real referential integrity.
- **`tv_series` is not a media_item** — a series has no single release date; its *seasons* do. The
  series is the searchable grouping (`series 1 → many seasons`); the season is the dated,
  trackable unit.
- **`company` is one flat table** — a "developer" / "publisher" / "studio" is a role a company
  plays, not a different kind of company. The role lives on the `media_company` join, so one
  company (e.g. Nintendo) can develop one title and publish another.
- **Many-to-many via join tables** — `media_company` (media ↔ company + role),
  `game_platform` (game ↔ platform), `media_tag` (media ↔ tag), and `track` (user ↔ media,
  with its own attributes). Each has a unique constraint on its natural key.
- **Single source of truth** — facts about a thing live on that thing. `release_date` is on
  `media_item`, never copied onto `track`, so a date change updates one row and every tracker
  sees it.

## Stack

- **Backend:** Java + Spring Boot, REST API
- **Database:** PostgreSQL
- **Frontend:** built in two stages — (1) plain JS/HTML/CSS + `fetch` to prove the fundamentals,
  then (2) React
- **Stretch:** a Python + FastAPI service for scraping/ingesting release data; auth; notifications

## Goals

1. Design a clean relational schema and defend every modeling decision.
2. Build and defend a Spring Boot + Postgres REST API.
3. Close the primary skill gap: a real **web** frontend — plain JS first, then React.
4. Practice real debugging against a seeded-bug scaffold.
5. Produce a chat transcript that proves understanding, not just a working repo.

## Repository layout

- `docs/` — sprint guide, prompts (phases 0–4), ERD, candidate reports
- `docs/erd.dbml`, `docs/erd.png` — the data model
- (coming) backend, frontend, and `docker compose` to run the full stack locally

## Status & next steps

- [~] **Phase 1 — Explore:** *nearly done.* Data model designed and defended (ERD committed);
      [`SCOPE.md`](SCOPE.md) written (brief, entities, walking skeleton, ranked stretch).
  - [x] Walking skeleton defined: search query → `GET /api/media?q=` → Postgres → list renders.
  - [x] `SCOPE.md` written — Phase 2 reads this file.
  - [ ] Get scope signed off by instructor.
- [ ] **Phase 2 — Scaffold:** stand up Spring Boot + Postgres skeleton wired to this schema,
      seeded with intentional bugs to hunt
- [ ] **Phase 3 — Build:** fix the bugs, build features (search → track → notify), frontend in
      plain JS then React
- [ ] **Phase 4 — Verify:** explain-to-commit review gate before each commit and end of each day
