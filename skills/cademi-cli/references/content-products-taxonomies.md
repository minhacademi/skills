---
name: cademi-cli-content-products-taxonomies
description: "Classify a product's lessons with taxonomies and terms — `cademi content products taxonomies` (10 commands)"
metadata:
  cademi-cli: "0.2.1"
  cademi-api: "3.10.0"
---

# Content Products Taxonomies Commands

> cademi 0.2.1, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi content` — Organize showcases, products and learning content.

Define taxonomies and their terms for a product. Assign terms through
lessons taxonomy-terms. Supported classification depends on product format.

**Related:** `cademi content products lessons taxonomy-terms`

**See also in this domain:** `references/content-banners.md`, `references/content-products.md`, `references/content-products-access-schedules.md`, `references/content-products-exams.md`, `references/content-products-lessons.md`, `references/content-products-modules.md`, `references/content-showcases.md`

Live catalog for this file: `cademi commands content products taxonomies --json` (offline, no credential needed).

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

### `cademi content products taxonomies create <product_id>`

Create a taxonomy · `POST /api/v3/products/{product_id}/taxonomies` · permission `taxonomies.create`

Creates a taxonomy in a product, with a singular `name` and a `plural_name`.

Taxonomies are available only for products in the `blog` format. For other formats, the API returns `409 Conflict` with the `format_unsupported` error code.

Taxonomies cannot be created in replicated products. Attempts to do so return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes |  |
| `plural_name` | string | yes |  |

Full schema: `cademi commands content products taxonomies create --schema --json`

Legacy path: `cademi taxonomies create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products taxonomies create prd_42 -f name=<name> -f plural_name=<plural_name> --json

# full body from a file
cademi content products taxonomies create prd_42 --data @body.json --json
```

### `cademi content products taxonomies delete <product_id> <taxonomy_id>`

Delete a taxonomy · `DELETE /api/v3/products/{product_id}/taxonomies/{taxonomy_id}` · permission `taxonomies.delete`

Moves the taxonomy and all of its terms to the trash. The terms are also removed from every lesson they were assigned to.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid deleting a taxonomy that has changed since it was retrieved.

Taxonomies of replicated products are read-only and cannot be deleted. Attempts to delete them return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `taxonomy_id` — Public ID of the taxonomy, prefixed with `tax_`. Example: `tax_9`

**Flag sets:** output, idempotency, confirm

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

Legacy path: `cademi taxonomies delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products taxonomies delete prd_42 tax_9 --yes
```

### `cademi content products taxonomies get <product_id> <taxonomy_id>`

Retrieve a taxonomy · `GET /api/v3/products/{product_id}/taxonomies/{taxonomy_id}` · permission `taxonomies.read`

Retrieves a taxonomy of a product by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating or deleting the taxonomy to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `taxonomy_id` — Public ID of the taxonomy, prefixed with `tax_`. Example: `tax_9`

**Flag sets:** output

Legacy path: `cademi taxonomies get`

**Examples:**

```bash
# get
cademi content products taxonomies get prd_42 tax_9 --json
```

### `cademi content products taxonomies list <product_id>`

List taxonomies · `GET /api/v3/products/{product_id}/taxonomies` · permission `taxonomies.read`

Lists the taxonomies of a product. Results are paginated with `limit` and `cursor`.

Taxonomies are available only for products in the `blog` format. For other formats, the API returns `409 Conflict` with the `format_unsupported` error code instead of an empty collection.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

Legacy path: `cademi taxonomies list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products taxonomies list prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products taxonomies list prd_42 --limit 200 --raw --json

# every page, projected
cademi content products taxonomies list prd_42 --all --jq '[.[] | {id}]'
```

### `cademi content products taxonomies update <product_id> <taxonomy_id>`

Update a taxonomy · `PATCH /api/v3/products/{product_id}/taxonomies/{taxonomy_id}` · permission `taxonomies.update`

Updates the `name` and `plural_name` of a taxonomy.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version.

Taxonomies of replicated products are read-only. Attempts to update them return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `taxonomy_id` — Public ID of the taxonomy, prefixed with `tax_`. Example: `tax_9`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string |  |  |
| `plural_name` | string |  |  |

Full schema: `cademi commands content products taxonomies update --schema --json`

Legacy path: `cademi taxonomies update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products taxonomies update prd_42 tax_9 -f name=<name> --if-match '"<etag>"' --json

# full body from a file
cademi content products taxonomies update prd_42 tax_9 --data @body.json --json
```

### `cademi content products taxonomies terms` — Manage content products taxonomies terms

Related: `cademi content products lessons taxonomy-terms`

#### `cademi content products taxonomies terms create <product_id> <taxonomy_id>`

Create a taxonomy term · `POST /api/v3/products/{product_id}/taxonomies/{taxonomy_id}/terms` · permission `taxonomies.update`

Creates a term in a taxonomy. Requires the `taxonomies.update` permission.

The optional `image` field accepts the ID of a file previously uploaded to the account.

Terms cannot be created in taxonomies of replicated products. Attempts to do so return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `taxonomy_id` — Public ID of the taxonomy, prefixed with `tax_`. Example: `tax_9`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `image` | string, nullable |  |  |
| `name` | string | yes |  |

Full schema: `cademi commands content products taxonomies terms create --schema --json`

Legacy path: `cademi taxonomies terms create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products taxonomies terms create prd_42 tax_9 -f name=<name> --json

# full body from a file
cademi content products taxonomies terms create prd_42 tax_9 --data @body.json --json
```

#### `cademi content products taxonomies terms delete <product_id> <taxonomy_id> <taxonomy_term_id>`

Delete a taxonomy term · `DELETE /api/v3/products/{product_id}/taxonomies/{taxonomy_id}/terms/{taxonomy_term_id}` · permission `taxonomies.update`

Moves the term to the trash and removes it from every lesson it was assigned to.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid deleting a term that has changed since it was retrieved.

Terms of replicated products are read-only and cannot be deleted. Attempts to delete them return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `taxonomy_id` — Public ID of the taxonomy, prefixed with `tax_`. Example: `tax_9`
- `taxonomy_term_id` — Public ID of the taxonomy term, prefixed with `ttm_`. Example: `ttm_3`

**Flag sets:** output, idempotency, confirm

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

Legacy path: `cademi taxonomies terms delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products taxonomies terms delete prd_42 tax_9 ttm_3 --yes
```

#### `cademi content products taxonomies terms get <product_id> <taxonomy_id> <taxonomy_term_id>`

Retrieve a taxonomy term · `GET /api/v3/products/{product_id}/taxonomies/{taxonomy_id}/terms/{taxonomy_term_id}` · permission `taxonomies.read`

Retrieves a taxonomy term by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating or deleting the term to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `taxonomy_id` — Public ID of the taxonomy, prefixed with `tax_`. Example: `tax_9`
- `taxonomy_term_id` — Public ID of the taxonomy term, prefixed with `ttm_`. Example: `ttm_3`

**Flag sets:** output

Legacy path: `cademi taxonomies terms get`

**Examples:**

```bash
# get
cademi content products taxonomies terms get prd_42 tax_9 ttm_3 --json
```

#### `cademi content products taxonomies terms list <product_id> <taxonomy_id>`

List taxonomy terms · `GET /api/v3/products/{product_id}/taxonomies/{taxonomy_id}/terms` · permission `taxonomies.read`

Lists the terms of a taxonomy, sorted alphabetically by name. Results are paginated with `limit` and `cursor`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `taxonomy_id` — Public ID of the taxonomy, prefixed with `tax_`. Example: `tax_9`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

Legacy path: `cademi taxonomies terms list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products taxonomies terms list prd_42 tax_9 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products taxonomies terms list prd_42 tax_9 --limit 200 --raw --json

# every page, projected
cademi content products taxonomies terms list prd_42 tax_9 --all --jq '[.[] | {id}]'
```

#### `cademi content products taxonomies terms update <product_id> <taxonomy_id> <taxonomy_term_id>`

Update a taxonomy term · `PATCH /api/v3/products/{product_id}/taxonomies/{taxonomy_id}/terms/{taxonomy_term_id}` · permission `taxonomies.update`

Updates the `name` and `image` of a taxonomy term. Omitted fields remain unchanged; setting `image` to `null` removes the image. `image` accepts the ID of a file previously uploaded to the account.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version.

Terms of replicated products are read-only. Attempts to update them return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `taxonomy_id` — Public ID of the taxonomy, prefixed with `tax_`. Example: `tax_9`
- `taxonomy_term_id` — Public ID of the taxonomy term, prefixed with `ttm_`. Example: `ttm_3`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `image` | string, nullable |  |  |
| `name` | string |  |  |

Full schema: `cademi commands content products taxonomies terms update --schema --json`

Legacy path: `cademi taxonomies terms update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products taxonomies terms update prd_42 tax_9 ttm_3 -f image=<image> --if-match '"<etag>"' --json

# full body from a file
cademi content products taxonomies terms update prd_42 tax_9 ttm_3 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
