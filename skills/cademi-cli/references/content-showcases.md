---
name: cademi-cli-content-showcases
description: "Organize products in showcases and showcase groups — `cademi content showcases` (8 commands)"
metadata:
  cademi-cli: "0.2.7"
  cademi-api: "3.12.1"
---

# Content Showcases Commands

> cademi 0.2.7, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi content` — Organize showcases, products and learning content.

Groups contain showcases; showcases contain products. Groups are top-level.
Lists return top-level items by default; use --parent-id to explore a group.

**Related:** `cademi content products`

**See also in this domain:** `references/content-banners.md`, `references/content-products.md`, `references/content-products-access-schedules.md`, `references/content-products-exams.md`, `references/content-products-lessons.md`, `references/content-products-modules.md`, `references/content-products-taxonomies.md`

Live catalog for this file: `cademi commands content showcases --json` (offline, no credential needed).

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

**confirm**
- `-y, --yes` — Do not ask for confirmation

## Commands

### `cademi content showcases create`

Create a showcase · `POST /api/v3/showcases` · permission `showcases.create`

Creates a showcase or a showcase group. New showcases and groups are always created with the `draft` status.

Showcases follow a two-level hierarchy: groups exist only at the top level and can contain showcases. To create a showcase inside a group, set `parent_id` to the group's public ID; if `parent_id` does not reference a group, the request is rejected with `hierarchy_violation`. A showcase created inside a group is always created with the `showcase` kind.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `after_id` | string, nullable |  | Places the new showcase right after this showcase, which must share the same parent. Otherwise the request fails with `validation_failed` on `after_id`. |
| `kind` | string | yes | One of: `group`, `showcase` |
| `name` | string | yes |  |
| `parent_id` | string, nullable |  |  |

Full schema: `cademi commands content showcases create --schema --json`

Legacy path: `cademi showcases create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content showcases create -f kind=group -f name=<name> --json

# full body from a file
cademi content showcases create --data @body.json --json
```

### `cademi content showcases delete <showcase_id>`

Delete a showcase · `DELETE /api/v3/showcases/{showcase_id}` · permission `showcases.delete`

Moves the showcase to the trash. Its child showcases, the products in the showcase, and its copies in replicated accounts are moved to the trash along with it.

Replicated showcases are read-only and cannot be deleted. Attempts to delete them return the `replica_readonly` error code.

**Arguments:**
- `showcase_id` — Public ID of the showcase, prefixed with `shw_`. Example: `shw_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi showcases delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content showcases delete shw_42 --yes
```

### `cademi content showcases get <showcase_id>`

Retrieve a showcase · `GET /api/v3/showcases/{showcase_id}` · permission `showcases.read`

Retrieves a showcase by its public ID, including the number of products it contains.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the showcase to avoid overwriting a newer version.

Showcases that do not exist or are not accessible with the current credentials return `404 Not Found`.

**Arguments:**
- `showcase_id` — Public ID of the showcase, prefixed with `shw_`. Example: `shw_42`

**Flag sets:** output

Legacy path: `cademi showcases get`

**Examples:**

```bash
# get
cademi content showcases get shw_42 --json
```

### `cademi content showcases list`

List showcases · `GET /api/v3/showcases` · permission `showcases.read`

Lists showcases and showcase groups in the account, ordered by position and paginated with a cursor.

By default, only top-level items are returned; use `parent_id` to list the showcases inside a group. The collection can also be filtered by `status`, `kind`, and name (`q`). Set `deleted=true` to list showcases in the trash instead of active ones.

Credentials limited to specific showcases only see those showcases.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--kind <string>` — Only groups or only showcases. (group, showcase)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200; API default: 50
- `--parent-id <string>` — Only showcases inside the group with this public ID. Without it, only top-level items are returned.
- `--q <string>` — Text search term.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (position, -position)
- `--status <string>` — Only items with this status. (published, draft, unlisted, hidden)

Legacy path: `cademi showcases list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content showcases list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content showcases list --limit 200 --raw --json

# every page, projected
cademi content showcases list --all --jq '[.[] | {id}]'
```

### `cademi content showcases update <showcase_id>`

Update a showcase · `PATCH /api/v3/showcases/{showcase_id}` · permission `showcases.update`, `showcases.publish (if field:status)`

Updates an existing showcase. Only the fields present in the request are changed.

Updating `status` requires the `showcases.publish` permission in addition to `showcases.update`. Setting `parent_id` moves the showcase into a group, or to the top level when `null`; moves that would break the two-level hierarchy are rejected with `hierarchy_violation`. Setting `deleted` to `false` restores a showcase from the trash.

`position` is 1-based within the showcase's level, in the new level when `parent_id` changes; a value greater than the number of showcases at that level places the showcase at the end. A showcase moved to another level without `position` is placed at the end.

Replicated showcases accept only `status`; a request that includes any other field returns the `replica_readonly` error code.

_Full description: `cademi commands content showcases update --json --jq '.commands[0].description'`_

**Arguments:**
- `showcase_id` — Public ID of the showcase, prefixed with `shw_`. Example: `shw_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | Only `false` is accepted. Restores it from the trash; deleting is the DELETE operation. One of: `false` |
| `name` | string |  |  |
| `parent_id` | string, nullable |  |  |
| `position` | integer |  |  |
| `settings` | object |  |  |
| `status` | string |  | One of: `published`, `draft`, `unlisted`, `hidden` |

Full schema: `cademi commands content showcases update --schema --json`

Legacy path: `cademi showcases update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content showcases update shw_42 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi content showcases update shw_42 --data @body.json --json
```

### `cademi content showcases copies` — Duplicate a showcase

Related: `cademi content products`

#### `cademi content showcases copies create <showcase_id>`

Duplicate a showcase · `POST /api/v3/showcases/{showcase_id}/copies` · permission `showcases.create`

Creates a copy of the showcase in the same account. Copying a showcase to a different account is not supported.

**Arguments:**
- `showcase_id` — Public ID of the showcase, prefixed with `shw_`. Example: `shw_42`

**Flag sets:** output, idempotency

Legacy path: `cademi showcases copies create`

**Examples:**

```bash
# run
cademi content showcases copies create shw_42 --json
```

### `cademi content showcases order` — Reorder showcases

Related: `cademi content showcases list`, `cademi guide reordering`

#### `cademi content showcases order update`

Reorder showcases · `PUT /api/v3/showcases/order` · permission `showcases.update`

Sets the display order of the showcases at one level of the hierarchy: the top level when `parent_id` is omitted, or the showcases inside the group identified by `parent_id`.

The request must contain the complete set of showcase IDs at that level. If the supplied set does not match, the request is rejected with `order_set_mismatch` and the existing order remains unchanged.

Discover the required set through GET /showcases with the same parent_id (omit it for the top level), following every cursor page without narrowing filters such as status, kind or search. A fully paginated list is still limited by credential visibility; verify that the credential can read the entire required set before reordering. On order_set_mismatch, re-read the set and check scope and visibility instead of blindly retrying stale IDs.

_Full description: `cademi commands content showcases order update --json --jq '.commands[0].description'`_

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `ids` | array of string | yes |  |
| `parent_id` | string, nullable |  |  |

Full schema: `cademi commands content showcases order update --schema --json`

Legacy path: `cademi showcases order update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content showcases order update -f 'ids[]=<value>' --json

# full body from a file
cademi content showcases order update --data @body.json --json
```

### `cademi content showcases products` — List showcase products

Related: `cademi content products`

#### `cademi content showcases products list <showcase_id>`

List showcase products · `GET /api/v3/showcases/{showcase_id}/products` · permission `showcases.read`, `products.read`

Lists the products in a showcase in summary form, ordered by position and paginated with a cursor. Requires both the `showcases.read` and `products.read` permissions.

Credentials limited to specific products only see those products.

**Arguments:**
- `showcase_id` — Public ID of the showcase, prefixed with `shw_`. Example: `shw_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200; API default: 50
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (position, -position)

Legacy path: `cademi showcases products list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content showcases products list shw_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content showcases products list shw_42 --limit 200 --raw --json

# every page, projected
cademi content showcases products list shw_42 --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
