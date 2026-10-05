---
name: cademi-cli-users-products
description: "Inspect a user's effective product access and progress — `cademi users products` (9 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Users Products Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi users` — Manage students, enrollments and learning progress.

These commands describe products from one user's perspective. To edit the
product itself, use content products. Access and progress are separate views.

**Related:** `cademi content products`, `cademi users enrollments`

**See also in this domain:** `references/users.md`, `references/users-imports.md`

Live catalog for this file: `cademi commands users products --json` (offline, no credential needed).

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

### `cademi users products list <user_id>`

List product access · `GET /api/v3/users/{user_id}/products` · permission `enrollments.read`

Returns the user's effective access to each product, using cursor-based pagination and sorted by product ID.

If the current credentials are restricted to specific products, only those products are included.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (product_id, -product_id)

**Examples:**

```bash
# list: one page, machine-readable
cademi users products list usr_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users products list usr_42 --limit 200 --raw --json

# every page, projected
cademi users products list usr_42 --all --jq '[.[] | {id}]'
```

### `cademi users products access` — Inspect access sources, denial reasons and user overrides

Supply user and product IDs. get returns effective access, enrollment
sources, denial_reason and overrides. update changes individual overrides.
calendar describes effective content release rules, not dated release events.

Related: `cademi users enrollments`, `cademi content products access-schedules`

```bash
cademi users products access get usr_42 prd_7
cademi users products access calendar get usr_42 prd_7
```

#### `cademi users products access get <user_id> <product_id>`

Retrieve product access · `GET /api/v3/users/{user_id}/products/{product_id}/access` · permission `enrollments.read`

Retrieves the user's effective access to a product, including the sources that grant it (`sources[]`), the reason access is denied when applicable (`denial_reason`), and any overrides applied to the user (`overrides`).

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the product access to avoid overwriting a newer version.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_7`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users products access get usr_42 prd_7 --json
```

#### `cademi users products access update <user_id> <product_id>`

Update product access · `PATCH /api/v3/users/{user_id}/products/{product_id}/access` · permission `enrollments.manage_access`

Overrides the user's access to a product. Only the fields supplied in the request are changed:

- `duration` sets the access duration for this product.
- `schedule_id` assigns the user to a schedule; send `null` to remove the assignment.
- `rules_waived` controls whether the product's content release rules are waived for this user.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `duration` | object, nullable |  |  |
| `duration.type` | string |  |  |
| `duration.value` | integer, nullable |  |  |
| `rules_waived` | boolean |  |  |
| `schedule_id` | string, nullable |  |  |

Full schema: `cademi commands users products access update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi users products access update usr_42 prd_7 -f duration.type=<duration.type> --if-match '"<etag>"' --json

# full body from a file
cademi users products access update usr_42 prd_7 --data @body.json --json
```

### `cademi users products access calendar` — Retrieve a release calendar

Related: `cademi users enrollments`, `cademi content products access-schedules`

#### `cademi users products access calendar get <user_id> <product_id>`

Retrieve a release calendar · `GET /api/v3/users/{user_id}/products/{product_id}/access/calendar` · permission `enrollments.read`

Returns the content release rules currently in effect for the user in each module and lesson of the product. The response describes the rules, not a list of dated release events.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_7`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users products access calendar get usr_42 prd_7 --json
```

### `cademi users products progress` — Manage users products progress

Related: `cademi content products`, `cademi users enrollments`

#### `cademi users products progress get <user_id> <product_id>`

Retrieve product progress · `GET /api/v3/users/{user_id}/products/{product_id}/progress` · permission `progress.read`

Retrieves the user's progress in a single product, in the same format used when listing the user's progress across products.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users products progress get usr_42 prd_42 --json
```

### `cademi users products progress adjustments` — Manage users products progress adjustments

Related: `cademi content products`, `cademi users enrollments`

#### `cademi users products progress adjustments create <user_id> <product_id>`

Create a progress adjustment · `POST /api/v3/users/{user_id}/products/{product_id}/progress/adjustments` · permission `progress.adjust`

Creates a progress adjustment for the user in a product. A `reset` adjustment clears the user's progress in the product. The `reason` is stored with the adjustment.

Points and certificates already earned are not reverted by a reset.

A reset requires the user to have started the product. When the user has never opened it, there is no progress to reset and the request fails with `409 progress_not_started` (`details[].reason` is `not_started`).

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `reason` | string | yes |  |
| `type` | string | yes | One of: `reset` |

Full schema: `cademi commands users products progress adjustments create --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users products progress adjustments create usr_42 prd_42 -f reason=<reason> -f type=reset --json

# full body from a file
cademi users products progress adjustments create usr_42 prd_42 --data @body.json --json
```

#### `cademi users products progress adjustments get <user_id> <product_id> <adjustment_id>`

Retrieve a progress adjustment · `GET /api/v3/users/{user_id}/products/{product_id}/progress/adjustments/{adjustment_id}` · permission `progress.read`

Retrieves a progress adjustment recorded for the user in a product.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `adjustment_id` — Public ID of the adjustment, prefixed with `adj_`. Example: `adj_9`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users products progress adjustments get usr_42 prd_42 adj_9 --json
```

#### `cademi users products progress adjustments list <user_id> <product_id>`

List progress adjustments · `GET /api/v3/users/{user_id}/products/{product_id}/progress/adjustments` · permission `progress.read`

Returns the progress adjustments recorded for the user in a product, using cursor-based pagination.

Each adjustment identifies its author as either an administrator (`admin`) or an API credential (`credential`).

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

**Examples:**

```bash
# list: one page, machine-readable
cademi users products progress adjustments list usr_42 prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users products progress adjustments list usr_42 prd_42 --limit 200 --raw --json

# every page, projected
cademi users products progress adjustments list usr_42 prd_42 --all --jq '[.[] | {id}]'
```

### `cademi users products progress lessons` — List lesson progress

Related: `cademi content products`, `cademi users enrollments`

#### `cademi users products progress lessons list <user_id> <product_id>`

List lesson progress · `GET /api/v3/users/{user_id}/products/{product_id}/progress/lessons` · permission `progress.read`

Returns the user's progress in each lesson of a product, using cursor-based pagination. Each entry identifies the lesson and its module by public ID.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

**Examples:**

```bash
# list: one page, machine-readable
cademi users products progress lessons list usr_42 prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users products progress lessons list usr_42 prd_42 --limit 200 --raw --json

# every page, projected
cademi users products progress lessons list usr_42 prd_42 --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
