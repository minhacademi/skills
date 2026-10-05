---
name: cademi-cli-reports
description: "Inspect activity, learning results and account-wide records — `cademi reports` (13 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Reports Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

Reports provide aggregate views across the account. enrollments lists
grants across users; users enrollments manages an individual user's grants.
Export jobs are managed under files exports; this domain reports their metrics.

**Related:** `cademi users`, `cademi content products`, `cademi operations`

**Group examples:**

```bash
cademi reports list
cademi reports enrollments list
```

Live catalog for this file: `cademi commands reports --json` (offline, no credential needed).

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

**body**
- `-d, --data <string>` — JSON body: a literal, @file or @- for stdin
- `-f, --field <stringArray>` — Add a string field: key=value (nested: a.b=v, arrays: tags[]=v)
- `-F, --typed-field <stringArray>` — Add a typed field: key=true\|false\|null\|number\|JSON, or key=@file to read text; applied after all -f fields

**idempotency**
- `--idempotency-key <string>` — Reuse the key returned with a failed request to repeat it (default: automatic UUIDv7)

## Commands

### `cademi reports list`

List available reports · `GET /api/v3/reports` · permission `reports.read`

Returns the catalog of available reports, including the filters each report accepts, its columns, and the source and freshness of its data. Columns marked as personal data are returned only when the credentials also have the `users.read_personal` permission.

The catalog is static and contains no account data.

Returns the whole catalog in a single response while preserving the standard collection response format.

**Flag sets:** output

**Examples:**

```bash
# get
cademi reports list --json
```

### `cademi reports activity` — List recent user activity

#### `cademi reports activity list`

List recent user activity · `GET /api/v3/reports/activity` · permission `reports.read`

Returns recent user activity, such as comments, questions, and tickets, within the requested date range.

This report does not support cursor pagination. `from` and `to` are required, `from` must not be later than `to`, and the range cannot exceed 90 days. The response contains at most `limit` entries (maximum and default: 100).

The `email`, `document`, and `phone` fields are included only when the credentials also have the `users.read_personal` permission.

**Flag sets:** output

**Flags:**
- `--from <string>` — First day of the date range (ISO 8601).
- `--limit <int64>` — Maximum number of items to return, from 1 to 100; minimum: 1; maximum: 100
- `--to <string>` — Last day of the date range (ISO 8601).

**Examples:**

```bash
# list: one page, machine-readable
cademi reports activity list --limit 20 --json
```

### `cademi reports certificates` — List certificate issuance by product

#### `cademi reports certificates list`

List certificate issuance by product · `GET /api/v3/reports/certificates` · permission `reports.read`

Returns the number of certificates issued for each product.

This report contains aggregate counts only and does not identify individual users. Certificates issued to a specific user are available through the certificates endpoints, which require their own permissions.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)

**Examples:**

```bash
# list: one page, machine-readable
cademi reports certificates list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi reports certificates list --limit 200 --raw --json

# every page, projected
cademi reports certificates list --all --jq '[.[] | {id}]'
```

### `cademi reports email-bounces` — Manage reports email-bounces

#### `cademi reports email-bounces list`

List email bounces · `GET /api/v3/reports/email-bounces` · permission `reports.read`

Returns user email addresses that bounced, were dropped, were reported as spam, or were deferred. Results are paginated with a cursor.

By default, only email bounces that have not been ignored are returned. Set `ignored=true` to list ignored email bounces instead.

The `email` field is included only when the credentials also have the `users.read_personal` permission.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--ignored` — When 'true', returns only ignored email bounces instead of the active ones.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)

**Examples:**

```bash
# list: one page, machine-readable
cademi reports email-bounces list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi reports email-bounces list --limit 200 --raw --json

# every page, projected
cademi reports email-bounces list --all --jq '[.[] | {id}]'
```

#### `cademi reports email-bounces update <bounce_id>`

Ignore an email bounce · `PATCH /api/v3/reports/email-bounces/{bounce_id}` · permission `email_bounces.ignore`

Marks an email bounce as ignored so the user's address can receive email from the account again. Requires the `email_bounces.ignore` permission.

The only accepted value is `ignored: true`. This change cannot be reverted through the API.

To make the update conditional, send an `If-Match` header built from the bounce's `revision` field, in the format `W/"email_bounce:<bounce_id>:<revision>"`. If the bounce has changed since that revision, the update is rejected.

**Arguments:**
- `bounce_id` — Public ID of the bounce, prefixed with `bnc_`. Example: `bnc_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `ignored` | boolean | yes | One of: `true` |

Full schema: `cademi commands reports email-bounces update --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi reports email-bounces update bnc_42 -f ignored=true --json

# full body from a file
cademi reports email-bounces update bnc_42 --data @body.json --json
```

### `cademi reports enrollments` — List enrollment grants across users

This collection is for account-wide discovery. To create, inspect or
update an individual enrollment, use users enrollments with a user ID.

Related: `cademi users enrollments`

#### `cademi reports enrollments list`

List enrollments · `GET /api/v3/enrollments` · permission `enrollments.read`

Lists enrollments across all users in the account.

At least one of `user_id`, `product_id`, `delivery_id`, or `updated_after` must be supplied; requests without any of these filters are rejected with a validation error. When the credentials are restricted to specific products or users, only enrollments within that scope are returned. Revoked enrollments are excluded unless `status=revoked` or `deleted=true` is supplied.

Each item includes a `links.self` URL pointing to the enrollment under its user.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `--delivery-id <string>` — Only enrollments granted by the delivery with this public ID.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--origin <string>` — Only enrollments with this origin. (manual, sale, subscription)
- `--product-id <string>` — Only enrollments in the product with this public ID.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--status <string>` — Only enrollments with this status. (active, suspended, revoked)
- `--updated-after <string>` — Only items updated after this date and time (ISO 8601).
- `--user-id <string>` — Only enrollments of the user with this public ID.

Legacy path: `cademi enrollments list`

**Examples:**

```bash
# list: one page, machine-readable
cademi reports enrollments list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi reports enrollments list --limit 200 --raw --json

# every page, projected
cademi reports enrollments list --all --jq '[.[] | {id}]'
```

### `cademi reports exams` — List exam performance

#### `cademi reports exams list`

List exam performance · `GET /api/v3/reports/exams` · permission `reports.read`

Returns aggregate performance metrics for each exam. Use `product_id` to limit the report to a single product.

This report does not include individual answers or user names.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--product-id <string>` — Only exams of the product with this public ID.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)

**Examples:**

```bash
# list: one page, machine-readable
cademi reports exams list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi reports exams list --limit 200 --raw --json

# every page, projected
cademi reports exams list --all --jq '[.[] | {id}]'
```

### `cademi reports exports` — Retrieve export metrics

#### `cademi reports exports get`

Retrieve export metrics · `GET /api/v3/reports/exports` · permission `reports.read`

Returns aggregate export metrics for the account, including totals by status and the most recent exports.

To page through all exports, use the exports collection endpoint, which requires the `exports.read` permission.

**Flag sets:** output

**Examples:**

```bash
# get
cademi reports exports get --json
```

### `cademi reports lessons` — List lesson performance for a product

#### `cademi reports lessons list`

List lesson performance for a product · `GET /api/v3/reports/lessons` · permission `reports.read`

Returns performance metrics for each lesson of the product identified by `product_id`, which is required.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--product-id <string>` — Public ID of the product whose lessons are reported.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)

**Examples:**

```bash
# list: one page, machine-readable
cademi reports lessons list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi reports lessons list --limit 200 --raw --json

# every page, projected
cademi reports lessons list --all --jq '[.[] | {id}]'
```

### `cademi reports products` — List product performance

#### `cademi reports products list`

List product performance · `GET /api/v3/reports/products` · permission `reports.read`

Returns one row per product accessible with the current credentials. Products outside that scope are excluded from both the rows and any totals.

This report does not include user data.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)

**Examples:**

```bash
# list: one page, machine-readable
cademi reports products list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi reports products list --limit 200 --raw --json

# every page, projected
cademi reports products list --all --jq '[.[] | {id}]'
```

### `cademi reports rankings` — Retrieve the points ranking

#### `cademi reports rankings list`

Retrieve the points ranking · `GET /api/v3/reports/rankings` · permission `reports.read`, `rankings.read`

Returns the same points ranking as the gamification ranking endpoint. Either the `reports.read` or the `rankings.read` permission grants access.

Results are limited to the products and users accessible with the current credentials. Each entry includes the user's name and avatar only when the credentials also have the `users.read` permission.

**Flag sets:** output

**Flags:**
- `--from <string>` — Required when 'period' is 'custom'.
- `--limit <int64>` — Maximum number of items to return, from 1 to 100; minimum: 1; maximum: 100
- `--period <string>` — Time window of the ranking. With 'custom', send 'from' and 'to'. (all, today, last_7_days, last_30_days, custom)
- `--product-id <stringSlice>` — Only points related to these products, by public ID. Repeat the parameter to send several products.
- `--to <string>` — Required when 'period' is 'custom'.

**Examples:**

```bash
# list: one page, machine-readable
cademi reports rankings list --limit 20 --json
```

### `cademi reports support` — Retrieve support metrics

#### `cademi reports support get`

Retrieve support metrics · `GET /api/v3/reports/support` · permission `reports.read`

Returns message volume and response metrics for each support channel within the requested date range.

`from` and `to` are required, `from` must not be later than `to`, and the range cannot exceed 90 days. Figures are computed from live data.

**Flag sets:** output

**Flags:**
- `--from <string>` — First day of the date range (ISO 8601).
- `--to <string>` — Last day of the date range (ISO 8601).

**Examples:**

```bash
# get
cademi reports support get --json
```

### `cademi reports users` — Retrieve the users report

#### `cademi reports users get`

Retrieve the users report · `GET /api/v3/reports/users` · permission `reports.read`

Returns summary figures and a time series of user registrations and activity.

`from` and `to` are optional and default to the last 90 days. `from` must not be later than `to`, and the range cannot exceed 90 days.

Figures are aggregated periodically rather than in real time. The `consistency` object indicates the data source and how fresh the figures are.

**Flag sets:** output

**Flags:**
- `--from <string>` — First day of the date range (ISO 8601). Defaults to 90 days before 'to'.
- `--to <string>` — Last day of the date range (ISO 8601). Defaults to today.

**Examples:**

```bash
# get
cademi reports users get --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
