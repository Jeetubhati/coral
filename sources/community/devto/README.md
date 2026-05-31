# dev.to

**Version:** 0.1.0
**Backend:** HTTP
**Tables:** 2
**Base URL:** `https://dev.to/api`

Query public articles and top tags from [dev.to](https://dev.to) (Forem)
via the public dev.to API.

## Authentication

**Not required.** This source exposes only the public, unauthenticated
read endpoints of the dev.to API, so it can be registered and queried
with zero credentials.

```bash
coral source add --file sources/community/devto/manifest.yaml
```

Or interactively:

```bash
coral source add --file sources/community/devto/manifest.yaml --interactive
```

## Tables

| Table | Description | Filters |
|---|---|---|
| `articles` | Public articles by a given dev.to username. | `username` (required) |
| `tags` | Top tags on dev.to. | — |

## Quick Start

```sql
-- Top tags on the platform
coral sql "SELECT id, name, bg_color_hex FROM devto.tags ORDER BY id LIMIT 10"

-- Most-loved posts by a specific author
coral sql "SELECT title, positive_reactions_count, comments_count
           FROM devto.articles
           WHERE username = 'ben'
           ORDER BY positive_reactions_count DESC LIMIT 10"

-- Cross-source idea: join dev.to articles with your GitHub repos
coral sql "SELECT a.title, a.url, r.stargazers_count
           FROM devto.articles a
           JOIN github.user_repos r
             ON lower(a.title) LIKE '%' || lower(r.name) || '%'
           WHERE a.username = 'YOUR_USERNAME' LIMIT 20"
```

## Notes

- The dev.to API uses an `api-key` HTTP header for authenticated
  endpoints (no `Bearer` prefix). This source ships **without** any
  auth wiring, so anyone can register and use it.
- Pagination is page-numbered. The manifest sets `per_page=1000` so
  prolific authors fit in one or two pages, since SQL `LIMIT` is not
  pushed down to the upstream API.
- An `articles_me` table (your own articles and drafts, including
  page-view counts) is intentionally **not** part of this initial
  contribution. Coral's current DSL treats any `from: input`
  reference as required at `coral source add` time even when the
  input is marked `required: false`, which would force every user of
  this source to set `DEVTO_API_KEY` just to query the public tables.
  Once the DSL supports optional input references (or a `kind: secret`
  + `default: ""` combination), `articles_me` can land as a follow-up.
