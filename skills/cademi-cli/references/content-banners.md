---
name: cademi-cli-content-banners
description: "Manage banners shown in the student area — `cademi content banners` (7 commands)"
metadata:
  cademi-cli: "0.2.6"
  cademi-api: "3.12.1"
---

# Content Banners Commands

> cademi 0.2.6, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi content` — Organize showcases, products and learning content.

Banners present content and destinations to students. A banner's visibility
or destination does not itself grant access to a product.

**Related:** `cademi content products`

**Related settings:**
- `cademi settings branding` — Controls the student-area visual identity and certificate logos. (`cademi settings branding get|update`)

**See also in this domain:** `references/content-products.md`, `references/content-products-access-schedules.md`, `references/content-products-exams.md`, `references/content-products-lessons.md`, `references/content-products-modules.md`, `references/content-products-taxonomies.md`, `references/content-showcases.md`

Live catalog for this file: `cademi commands content banners --json` (offline, no credential needed).

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

### `cademi content banners create`

Create a banner · `POST /api/v3/banners` · permission `banners.create`

Creates a content banner.

The banner `type` must be available for the account's current layout; otherwise, the API returns the `banner_type_unavailable` error code. Link URLs must use the HTTP or HTTPS scheme, and HTML content is sanitized before it is stored.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `html` | string |  |  |
| `images` | object |  |  |
| `link` | object |  |  |
| `name` | string | yes |  |
| `showcases` | array of string |  |  |
| `type` | string | yes | One of: `netflix`, `simple`, `html` |

Full schema: `cademi commands content banners create --schema --json`

Legacy path: `cademi banners create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content banners create -f name=<name> -f type=netflix --json

# full body from a file
cademi content banners create --data @body.json --json
```

### `cademi content banners delete <banner_id>`

Delete a banner · `DELETE /api/v3/banners/{banner_id}` · permission `banners.delete`

Moves the banner to the trash instead of permanently deleting it. Trashed banners can be restored through the update operation.

Replicated banners are read-only and cannot be deleted. Attempts to delete them return `403` with the `replica_readonly` error code.

**Arguments:**
- `banner_id` — Public ID of the banner, prefixed with `ban_`. Example: `ban_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi banners delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content banners delete ban_42 --yes
```

### `cademi content banners get <banner_id>`

Retrieve a banner · `GET /api/v3/banners/{banner_id}` · permission `banners.read`

Retrieves a content banner by its public ID.

The response includes an `ETag` header representing the current revision. Send this value in the `If-Match` header when updating the banner to avoid overwriting a newer version.

Banners in the trash are not returned and respond with `404 Not Found`. To restore one, send `deleted: false` to the update operation.

**Arguments:**
- `banner_id` — Public ID of the banner, prefixed with `ban_`. Example: `ban_42`

**Flag sets:** output

Legacy path: `cademi banners get`

**Examples:**

```bash
# get
cademi content banners get ban_42 --json
```

### `cademi content banners list`

List banners · `GET /api/v3/banners` · permission `banners.read`

Lists the content banners of the current account.

The collection can be filtered by `status`, `showcase_id`, and `deleted`. Trashed banners are excluded by default; set `deleted=true` to list only trashed banners.

Results are paginated with `limit` and `cursor` and sorted by display position in ascending order. Use `sort=-position` to reverse the order.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200; API default: 50
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--showcase-id <string>` — Only banners displayed in the showcase with this public ID.
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (position, -position)
- `--status <string>` — Only banners with this status. (active, inactive)

Legacy path: `cademi banners list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content banners list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content banners list --limit 200 --raw --json

# every page, projected
cademi content banners list --all --jq '[.[] | {id}]'
```

### `cademi content banners update <banner_id>`

Update a banner · `PATCH /api/v3/banners/{banner_id}` · permission `banners.update`, `banners.publish (if field:status)`

Updates an existing content banner. Only the fields included in the request are changed.

Updating `status` requires the `banners.publish` permission in addition to `banners.update`. Setting `deleted` to `false` restores a banner from the trash. Changing the banner `type` is subject to the same layout availability rule as creation.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to prevent overwriting changes made since the banner was retrieved.

Replicated banners accept only `status` updates. Including any other field returns `403` with the `replica_readonly` error code.

**Arguments:**
- `banner_id` — Public ID of the banner, prefixed with `ban_`. Example: `ban_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | Set false to restore a deleted banner. true is not accepted; use the delete operation. One of: `false` |
| `html` | string |  | HTML content. Unsupported markup is sanitized before storage. |
| `images` | object |  | Images for simple banners, referenced by account file IDs. |
| `images.desktop` | string, nullable |  | Account file public ID prefixed with file_. null preserves the current image. |
| `images.mobile` | string, nullable |  | Account file public ID prefixed with file_. null preserves the current image. |
| `link` | object |  | Replaces the link configuration. Omitted link children use their defaults. |
| `link.kind` | string |  | Destination type; none disables the link. One of: `none`, `url`, `product` |
| `link.product_id` | string, nullable |  | Product public ID prefixed with prd_, used when kind=product. |
| `link.target` | string |  | Whether the link opens in the same tab or a new tab. One of: `_self`, `_blank` |
| `link.url` | string, nullable |  | HTTP or HTTPS URL used when kind=url. |
| `name` | string |  | Administrator-facing name of the banner. |
| `showcases` | array of string |  | Replaces the associated showcases with this list of shw_ public IDs. |
| `status` | string |  | Publication status. Changing this field requires banners.publish. Replicas accept only this field. One of: `active`, `inactive` |
| `type` | string |  | Banner format. Availability depends on the account layout. One of: `netflix`, `simple`, `html` |

Full schema: `cademi commands content banners update --schema --json`

Legacy path: `cademi banners update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content banners update ban_42 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi content banners update ban_42 --data @body.json --json
```

### `cademi content banners copies` — Duplicate a banner

Related: `cademi content products`

Related settings:
- `cademi settings branding` — Controls the student-area visual identity and certificate logos. (`cademi settings branding get|update`)

#### `cademi content banners copies create <banner_id>`

Duplicate a banner · `POST /api/v3/banners/{banner_id}/copies` · permission `banners.create`

Creates a copy of an existing content banner, including its showcase assignments. The copy receives a new public ID and a name derived from the original.

**Arguments:**
- `banner_id` — Public ID of the banner, prefixed with `ban_`. Example: `ban_42`

**Flag sets:** output, idempotency

Legacy path: `cademi banners copies create`

**Examples:**

```bash
# run
cademi content banners copies create ban_42 --json
```

### `cademi content banners order` — Reorder banners

Related: `cademi content banners list`, `cademi guide reordering`

#### `cademi content banners order update`

Reorder banners · `PUT /api/v3/banners/order` · permission `banners.update`

Sets the display order of the account's content banners.

The `ids` array must contain the public ID of every banner that is not in the trash, both active and inactive, in the desired order. If the supplied set does not match, the API returns the `order_set_mismatch` error code and the existing order remains unchanged.

Discover the required set through GET /banners, following every cursor page without narrowing filters such as status, kind or search. A fully paginated list is still limited by credential visibility; verify that the credential can read the entire required set before reordering. On order_set_mismatch, re-read the set and check scope and visibility instead of blindly retrying stale IDs.

The response returns the complete reordered collection in a single response, including collections larger than 200 items. This reorder response is not paginated.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `ids` | array of string | yes |  |

Full schema: `cademi commands content banners order update --schema --json`

Legacy path: `cademi banners order update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content banners order update -f 'ids[]=<value>' --json

# full body from a file
cademi content banners order update --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
