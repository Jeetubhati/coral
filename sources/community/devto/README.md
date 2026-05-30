# dev.to

**Version:** 0.1.0
**Backend:** HTTP
**Tables:** 3
**Base URL:** `https://dev.to/api`

Query articles, drafts, engagement metrics, and tags from [dev.to](https://dev.to) (Forem) via the public dev.to API.

## Authentication

Requires a dev.to API key (`DEVTO_API_KEY`) for the `articles_me` table.
The public `articles` and `tags` tables do not require authentication, but
the same `api-key` header is sent on every request.

Generate a key at [dev.to/settings/extensions](https://dev.to/settings/extensions).

```bash
DEVTO_API_KEY=your_key_here \
  coral source add --file sources/community/devto/manifest.yaml
```

Run the command from the repository root, or add interactively:

```bash
coral source add --file sources/community/devto/manifest.yaml --interactive
```

## Tables

| Table | Description | Filters | Auth |
|---|---|---|---|
| `articles_me` | Your own articles and drafts on dev.to, with view, reaction, and comment counts. | — | Required |
| `articles` | Public articles by a given dev.to username. | `username` (required) | Optional |
| `tags` | Top tags on dev.to. | — | Optional |

## Quick Start

```sql
-- Your top performing posts
coral sql "SELECT title, page_views_count, positive_reactions_count
           FROM devto.articles_me
           ORDER BY page_views_count DESC LIMIT 10"

-- Your unpublished drafts
coral sql "SELECT title, slug FROM devto.articles_me WHERE published = false"

-- Public articles by another author
coral sql "SELECT title, positive_reactions_count
           FROM devto.articles
           WHERE username = 'ben'
           ORDER BY positive_reactions_count DESC LIMIT 10"

-- Top tags
coral sql "SELECT id, name, bg_color_hex FROM devto.tags ORDER BY id LIMIT 10"
```

## Notes

- The dev.to API uses an `api-key` HTTP header (no `Bearer` prefix).
- Pagination is page-numbered. The manifest sets `per_page=1000` to fit
  prolific authors in one or two pages, since SQL `LIMIT` is not pushed
  down to the upstream API.
- The `articles_me` table includes drafts (`state=all`); filter on
  `published = true` or `published = false` to narrow.
