# SCOPE — Release Radar

Phase-1 (Explore) output. Phase 2 reads this to scaffold.

## Product brief

Release Radar is a full-stack web app for tracking upcoming release dates of games, movies, and
TV seasons. A user searches a catalog of media, opens an item's detail page, and adds it to their
personal tracks; on the day an item releases, the app emails the users tracking it. It's a
fundamentals project — Java/Spring Boot REST API, PostgreSQL, and a web frontend built plain-JS
first then React — with no AI in the core.

## Core entities + relationships

Full schema in [`docs/erd.dbml`](docs/erd.dbml). 14 tables.

- **`user`** — accounts; `password_hash`, never plaintext. Has many `track`.
- **`media_item`** — parent table (class-table inheritance) for anything dated and trackable.
  Common columns: `title`, `release_date`, `type`, `genre`, `description`.
  - **`game` / `movie` / `season`** — children; share the parent's PK (is-a, 1:1). Each holds
    only type-specific columns (`game`: num_players, online/offline flags; `movie`: runtime;
    `season`: season_number, num_episodes, series_id).
- **`tv_series`** — searchable grouping; NOT a media_item, has no release date. One series has
  many seasons (1→M); the season is the dated, trackable unit.
- **`company`** — one flat table. Role (developer/publisher/studio) is not stored here.
- **`platform`** — lookup (PS5, Xbox Series X, …).
- **`tag`** — lookup; a tag exists once and is shared.
- **`media_asset`** — images/trailers; many per media_item (M→1 back to media_item).

Relationships:

| Relationship | Cardinality | Mechanism |
|---|---|---|
| user ↔ media_item | many-to-many | `track` join (with attributes: date_tracked, is_notified) |
| media_item → game/movie/season | one-to-one | shared primary key (inheritance) |
| tv_series → season | one-to-many | FK `season.series_id` |
| media_item ↔ company | many-to-many | `media_company` join (+ `role`) |
| game ↔ platform | many-to-many | `game_platform` join |
| media_item ↔ tag | many-to-many | `media_tag` join |
| media_item → media_asset | one-to-many | FK `media_asset.media_item_id` |

Design principles held: single source of truth (release_date lives only on media_item);
many-to-many always via a join table with a unique constraint on its natural key; role lives on
the relationship (join), not the entity.

## Walking skeleton

The one thinnest end-to-end flow that proves the stack is wired, built first, before any feature:

> **User types a search query in the web UI → `GET /api/media?q=<query>` → Spring queries
> Postgres → JSON list of media items renders on screen.**

Touches frontend → REST API → database → frontend. Depends on nothing else (no auth, no
tracking, no notifications). Everything else hangs off this proven pipe.

## Ranked stretch (deliberately NOT first)

In priority order — pull in only after the skeleton and core search/list work:

1. **Accounts** — register / log in (BCrypt hash, simplest session). Brutally scoped: no
   "remember me", no email verification. Enables tracking. First stretch because it adds forms —
   real frontend reps.
2. **Track** — add/remove a media_item to a user's tracks; view "my tracks".
3. **Media detail page** — description, assets (images/trailers), clickable tags → tag page
   listing all media sharing that tag.
4. **Notifications** — scheduled/background job that emails users on an item's release date;
   flips `track.is_notified`.

The catalog is seeded with hand-written fixture data — no scraper in scope.

## Out of scope (and why)

- **FastAPI / Python service** — cut. Running FastAPI alongside Spring Boot to serve the same
  REST API is an anti-pattern: duplicated routing/logic, two runtimes to deploy, no capability
  gained. FastAPI would only earn a place doing a *different* job (e.g. a scraper), and scraping
  is not in scope. Release Radar is **one backend: Spring + Postgres**.
- The job's desire for Python/FastAPI experience is a **separate learning goal** — satisfy it
  with a small standalone FastAPI exercise, not by wedging a second backend into this app. The
  point of this week is the **web frontend**; two backends would steal that time.

## Next step

Get this scope signed off by instructor, then run Phase 2 (Scaffold).
