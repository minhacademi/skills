---
name: cademi-cli-gamification
description: "Inspect points and student rankings — `cademi gamification` (2 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Gamification Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

Scores record gamification points and rankings compare students. Configure
scoring behavior in settings gamification; individual adjustments and
scores are available under users.

**Related:** `cademi users scores`, `cademi reports rankings`

**Related settings:**
- `cademi settings gamification` — Controls ranking presentation and the activities that award points. Changes affect scoring when subsequent activity events are processed; saving settings does not recalculate previously awarded points or replay past activities. (`cademi settings gamification get|update`)

Live catalog for this file: `cademi commands gamification --json` (offline, no credential needed).

## Shared flag sets

Each command lists the sets it accepts. The flags of a set are:

**output**
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document)
- `--jq <string>` — Filter data (or --raw envelope); strings print unquoted, other values as JSON; overrides --output
- `--json` — Print JSON (same as --output json)
- `-o, --output <string>` — Output format: json, table or yaml (default: table for lists on a terminal, JSON otherwise)
- `--raw` — Print the full response envelope instead of data

**output-basic**
- `--jq <string>` — Filter data (or --raw envelope); strings print unquoted, other values as JSON; overrides --output
- `--json` — Print JSON (same as --output json)
- `-o, --output <string>` — Output format: json, table or yaml (default: table for lists on a terminal, JSON otherwise)

## Commands

### `cademi gamification rankings` — Retrieve the points ranking

Related: `cademi reports rankings`

Related settings:
- `cademi settings gamification` — Controls ranking presentation and the activities that award points. Changes affect scoring when subsequent activity events are processed; saving settings does not recalculate previously awarded points or replay past activities. (`cademi settings gamification get|update`)

#### `cademi gamification rankings list`

Retrieve the points ranking · `GET /api/v3/gamification/rankings` · permission `rankings.read`

Returns the users ranked by points earned in the selected period. When `period` is omitted, all-time points are used; when `limit` is omitted, up to 100 positions are returned.

Results are limited to the products and users accessible with the current credentials. Filtering by `product_id[]` narrows that scope and never widens it: requesting products outside the scope returns an empty ranking.

Each entry includes the user's name and avatar only when the credentials also have the `users.read` permission; otherwise the `user` object is `null`.

Rankings are eventually consistent. Manual point adjustments are reflected on the next request, while automatically awarded points may take up to `cache_ttl_seconds` to appear.

**Flag sets:** output

**Flags:**
- `--from <string>` — Required when 'period' is 'custom'.
- `--limit <int64>` — Maximum number of items to return, from 1 to 100; minimum: 1; maximum: 100
- `--period <string>` — Time window of the ranking. With 'custom', send 'from' and 'to'. (all, today, last_7_days, last_30_days, custom)
- `--product-id <stringSlice>` — Only points related to these products, by public ID. Repeat the parameter to send several products.
- `--to <string>` — Required when 'period' is 'custom'.

Legacy path: `cademi rankings list`

**Examples:**

```bash
# list: one page, machine-readable
cademi gamification rankings list --limit 20 --json
```

### `cademi gamification scores` — List point entries

Related: `cademi users scores`, `cademi users score-adjustments`

Related settings:
- `cademi settings gamification` — Controls ranking presentation and the activities that award points. Changes affect scoring when subsequent activity events are processed; saving settings does not recalculate previously awarded points or replay past activities. (`cademi settings gamification get|update`)

#### `cademi gamification scores list`

List point entries · `GET /api/v3/gamification/scores` · permission `scores.read`

Returns point entries from the points ledger, using cursor-based pagination. Points awarded automatically (lessons, comments, questions, exams, course milestones, and certificates) and manual adjustments are returned in the same collection and can be distinguished with the `origin` filter.

Resetting a user's progress or revoking a certificate does not remove previously awarded points. Reopening or deleting an exam attempt removes the points awarded for passing it, and the corresponding entries no longer appear in the collection.

Results are limited to the users and products accessible with the current credentials.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--created-after <string>` — Only items created after this date and time (ISO 8601).
- `--created-before <string>` — Only items created before this date and time (ISO 8601).
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--origin <string>` — Only entries with this origin, such as a lesson, a comment, or a manual adjustment.
- `--product-id <string>` — Only entries related to the product with this public ID.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (created_at, -created_at)
- `--user-id <string>` — Only entries of the user with this public ID.

Legacy path: `cademi scores list`

**Examples:**

```bash
# list: one page, machine-readable
cademi gamification scores list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi gamification scores list --limit 200 --raw --json

# every page, projected
cademi gamification scores list --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
