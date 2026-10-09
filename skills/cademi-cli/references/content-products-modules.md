---
name: cademi-cli-content-products-modules
description: "Manage a product's modules and module groups — `cademi content products modules` (9 commands)"
metadata:
  cademi-cli: "0.3.1"
  cademi-api: "3.13.1"
---

# Content Products Modules Commands

> cademi 0.3.1, API 3.13.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi content` — Organize showcases, products and learning content.

Commands require a product ID. A module group contains submodules; regular
modules contain lessons. Use content list to inspect direct children.
publish-all publishes the module and its nested content, including replicas.

**Related:** `cademi content products lessons`, `cademi content products content`

**See also in this domain:** `references/content-banners.md`, `references/content-products.md`, `references/content-products-access-schedules.md`, `references/content-products-exams.md`, `references/content-products-lessons.md`, `references/content-products-taxonomies.md`, `references/content-showcases.md`

**Group examples:**

```bash
cademi content products modules list prd_42
cademi content products modules content list prd_42 mod_7
```

Live catalog for this file: `cademi commands content products modules --json` (offline, no credential needed).

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

**async**
- `--wait` — When the API answers 202, wait for the operation to finish
- `--wait-timeout <duration>` — Maximum time for --wait, e.g. 30s or 2m (default: no limit; does not cancel the operation)

**confirm**
- `-y, --yes` — Do not ask for confirmation

## Commands

### `cademi content products modules create <product_id>`

Create a module · `POST /api/v3/products/{product_id}/modules` · permission `modules.create`

Creates a module or, when `kind` is `group`, a group of modules in the product. New modules are created as `draft`.

To create a submodule, set `parent_id` to the public ID of a group in the same product. `position` is 1-based among the module's siblings; a value greater than the number of siblings places the module at the end.

Modules cannot be created in replicated products. Attempts to do so return the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `kind` | string | yes | One of: `group`, `module` |
| `name` | string | yes |  |
| `parent_id` | string, nullable |  |  |
| `position` | integer |  |  |

Full schema: `cademi commands content products modules create --schema --json`

Legacy path: `cademi modules create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products modules create prd_42 -f kind=group -f name=<name> --json

# full body from a file
cademi content products modules create prd_42 --data @body.json --json
```

### `cademi content products modules delete <product_id> <module_id>`

Delete a module · `DELETE /api/v3/products/{product_id}/modules/{module_id}` · permission `modules.delete`

Moves the module to the trash, together with its submodules and lessons. Copies of the module in replicated accounts are also moved to the trash.

Restoring the module through the update operation does not restore those copies.

Replicated modules cannot be deleted. Attempts to delete them return the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `module_id` — Public ID of the module, prefixed with `mod_`. Example: `mod_7`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi modules delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products modules delete prd_42 mod_7 --yes
```

### `cademi content products modules get <product_id> <module_id>`

Retrieve a module · `GET /api/v3/products/{product_id}/modules/{module_id}` · permission `modules.read`

Retrieves a module by its public ID, including its typed `settings`.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the module or reordering its contents to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `module_id` — Public ID of the module, prefixed with `mod_`. Example: `mod_7`

**Flag sets:** output

Legacy path: `cademi modules get`

**Examples:**

```bash
# get
cademi content products modules get prd_42 mod_7 --json
```

### `cademi content products modules list <product_id>`

List modules · `GET /api/v3/products/{product_id}/modules` · permission `modules.read`

Returns the modules and groups of a product, ordered by position and paginated with a cursor.

The collection can be filtered by parent group, status, kind, and deletion state.

If the product is not accessible with the current credentials, the API returns `404 Not Found` instead of an empty collection.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--kind <string>` — Only groups or only modules. (group, module)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--parent-id <string>` — Only items inside the group with this public ID.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (position, -position)
- `--status <string>` — Only items with this status. (published, draft)

Legacy path: `cademi modules list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products modules list prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products modules list prd_42 --limit 200 --raw --json

# every page, projected
cademi content products modules list prd_42 --all --jq '[.[] | {id}]'
```

### `cademi content products modules publish-all <product_id> <module_id>`

Publish a module · `POST /api/v3/products/{product_id}/modules/{module_id}/publications` · permission `modules.publish`

Starts an asynchronous operation that publishes the module and everything nested in it.

Copies of the module in replicated accounts are published as well. Replicated modules can also be published through this operation.

The `Idempotency-Key` header is required. Track progress through the returned operation.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `module_id` — Public ID of the module, prefixed with `mod_`. Example: `mod_7`

**Flag sets:** output, idempotency, async

Legacy path: `cademi modules publications create`

**Examples:**

```bash
# run
cademi content products modules publish-all prd_42 mod_7 --wait --json
```

### `cademi content products modules update <product_id> <module_id>`

Update a module · `PATCH /api/v3/products/{product_id}/modules/{module_id}` · permission `modules.update`, `modules.publish (if field:status)`

Updates a module. Only the fields included in the request are changed.

Changing `status` requires the `modules.publish` permission. Setting `target_product_id` moves the module to another product and requires the `modules.update` permission on that product; if the target product is not accessible with the current credentials, the API returns `404 Not Found`. Setting `deleted` to `false` restores a module from the trash.

`position` is 1-based among the module's siblings, in the destination when the module is moved; a value greater than the number of siblings places the module at the end. A module moved to another group or product without `position` is placed at the end.

Replicated modules accept only `status`. Requests that include any other field return the `replica_readonly` error code.

_Full description: `cademi commands content products modules update --json --jq '.commands[0].description'`_

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `module_id` — Public ID of the module, prefixed with `mod_`. Example: `mod_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | Only `false` is accepted. Restores it from the trash; deleting is the DELETE operation. One of: `false` |
| `image` | string, nullable |  |  |
| `name` | string |  |  |
| `parent_id` | string, nullable |  |  |
| `position` | integer |  |  |
| `settings` | object |  |  |
| `status` | string |  | One of: `published`, `draft` |
| `target_product_id` | string |  |  |

Full schema: `cademi commands content products modules update --schema --json`

Legacy path: `cademi modules update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products modules update prd_42 mod_7 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi content products modules update prd_42 mod_7 --data @body.json --json
```

### `cademi content products modules content` — Inspect and order a module's direct children

Lists direct submodules and lessons in display order. Supply both product
and module IDs; use products content list for the product's root level.

Related: `cademi content products content`

#### `cademi content products modules content list <product_id> <module_id>`

List module contents · `GET /api/v3/products/{product_id}/modules/{module_id}/content` · permission `modules.read`

Returns the direct contents of a module, meaning its submodules and the lessons placed directly in it, in display order, using cursor-based pagination. Items in the trash are not included.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `module_id` — Public ID of the module, prefixed with `mod_`. Example: `mod_7`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Opaque continuation token from page.next_cursor of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of entries per page; minimum: 1; maximum: 200; API default: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

Legacy path: `cademi modules content list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products modules content list prd_42 mod_7 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products modules content list prd_42 mod_7 --limit 200 --raw --json

# every page, projected
cademi content products modules content list prd_42 mod_7 --all --jq '[.[] | {id}]'
```

### `cademi content products modules content order` — Reorder module contents

Related: `cademi content products modules content list`, `cademi content products modules get`, `cademi guide reordering`

#### `cademi content products modules content order update <product_id> <module_id>`

Reorder module contents · `PUT /api/v3/products/{product_id}/modules/{module_id}/content/order` · permission `modules.update`

Sets the display order of the submodules and lessons placed directly in the module.

The `ids` array accepts both module public IDs (`mod_`) and lesson public IDs (`les_`) in the desired order. The array must contain the complete set of BOTH direct submodules and direct lessons currently in the module. Omitting a kind is allowed only when that kind has no items; otherwise the entire request is rejected. The request requires at least one ID; an empty array is rejected. Positions are assigned independently within each kind, starting at 1, so interleaving IDs does not assign one global mixed ranking. For example, with only lessons [les_1, les_2], sending [les_2, les_1] sets their positions to 1 and 2. With both kinds present, sending only lesson IDs is rejected. If the supplied set does not match, the API returns the `order_set_mismatch` error code and the existing order remains unchanged.

_Full description: `cademi commands content products modules content order update --json --jq '.commands[0].description'`_

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `module_id` — Public ID of the module, prefixed with `mod_`. Example: `mod_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `ids` | array of string | yes |  |

Full schema: `cademi commands content products modules content order update --schema --json`

Legacy path: `cademi modules content order update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products modules content order update prd_42 mod_7 -f 'ids[]=<value>' --json

# full body from a file
cademi content products modules content order update prd_42 mod_7 --data @body.json --json
```

### `cademi content products modules copies` — Duplicate a module

Related: `cademi content products lessons`, `cademi content products content`

#### `cademi content products modules copies create <product_id> <module_id>`

Duplicate a module · `POST /api/v3/products/{product_id}/modules/{module_id}/copies` · permission `modules.create`

Starts an asynchronous operation that copies the module, including its contents, into the product given in `target_product_id`. Optionally, use `parent_id` to place the copy inside a group of the target product and `name` to rename it.

Requires the `modules.create` permission on the target product. Replicated modules cannot be copied, and modules cannot be copied into replicated products; both cases return the `replica_readonly` error code.

The `Idempotency-Key` header is required. Track progress through the returned operation; when it completes, the item result contains `copy_id`, the public ID of the new module.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `module_id` — Public ID of the module, prefixed with `mod_`. Example: `mod_7`

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string, nullable |  |  |
| `parent_id` | string, nullable |  |  |
| `target_product_id` | string | yes |  |

Full schema: `cademi commands content products modules copies create --schema --json`

Legacy path: `cademi modules copies create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products modules copies create prd_42 mod_7 -f target_product_id=<target_product_id> --wait --json

# full body from a file
cademi content products modules copies create prd_42 mod_7 --data @body.json --wait --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
