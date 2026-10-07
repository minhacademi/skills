---
name: cademi-cli-content-products-access-schedules
description: "Define when a product's content becomes available — `cademi content products access-schedules` (10 commands)"
metadata:
  cademi-cli: "0.2.6"
  cademi-api: "3.12.1"
---

# Content Products Access Schedules Commands

> cademi 0.2.6, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi content` — Organize showcases, products and learning content.

Schedules define release rules for nodes of the product tree. Deliveries
select a schedule for each product, and a user may have an override.
Inspect rules for explicit and inherited settings; inspect user access
for the effective result. A schedule alone does not grant an enrollment.

**Related:** `cademi sales deliveries products`, `cademi users products access`

**See also in this domain:** `references/content-banners.md`, `references/content-products.md`, `references/content-products-exams.md`, `references/content-products-lessons.md`, `references/content-products-modules.md`, `references/content-products-taxonomies.md`, `references/content-showcases.md`

**Group examples:**

```bash
cademi content products access-schedules list prd_42
```

Live catalog for this file: `cademi commands content products access-schedules --json` (offline, no credential needed).

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

### `cademi content products access-schedules create <product_id>`

Create an access schedule · `POST /api/v3/products/{product_id}/access-schedules` · permission `access_schedules.create`

Creates an access schedule for the product, identified by the supplied tag.

Each tag and product pair has at most one access schedule. If one already exists for the pair, the existing schedule is returned with `200 OK` and no new schedule is created.

Access schedules cannot be created for replicated products; these requests return the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `tag_id` | string | yes |  |

Full schema: `cademi commands content products access-schedules create --schema --json`

Legacy path: `cademi products access-schedules create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products access-schedules create prd_42 -f tag_id=<tag_id> --json

# full body from a file
cademi content products access-schedules create prd_42 --data @body.json --json
```

### `cademi content products access-schedules delete <product_id> <access_schedule_id>`

Delete an access schedule · `DELETE /api/v3/products/{product_id}/access-schedules/{access_schedule_id}` · permission `access_schedules.delete`

Moves the access schedule to the trash.

The default access schedule cannot be deleted and returns the `state_conflict` error code. An access schedule in use by a delivery that is not archived cannot be deleted either and also returns `state_conflict`. Replicated access schedules are read-only and return the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `access_schedule_id` — Public ID of the access schedule, prefixed with `acs_`. Example: `acs_7`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi products access-schedules delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products access-schedules delete prd_42 acs_7 --yes
```

### `cademi content products access-schedules get <product_id> <access_schedule_id>`

Retrieve an access schedule · `GET /api/v3/products/{product_id}/access-schedules/{access_schedule_id}` · permission `access_schedules.read`

Retrieves an access schedule by its public ID, including the deliveries that use it (`deliveries[]`).

The response includes an `ETag` representing the current revision of the access schedule. Send this value in the `If-Match` header when updating the access schedule or its release rules to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `access_schedule_id` — Public ID of the access schedule, prefixed with `acs_`. Example: `acs_7`

**Flag sets:** output

Legacy path: `cademi products access-schedules get`

**Examples:**

```bash
# get
cademi content products access-schedules get prd_42 acs_7 --json
```

### `cademi content products access-schedules list <product_id>`

List access schedules · `GET /api/v3/products/{product_id}/access-schedules` · permission `access_schedules.read`

Lists the access schedules of a product. Results are paginated with a cursor.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--tag-id <string>` — Only access schedules that apply to the tag with this public ID.

Legacy path: `cademi products access-schedules list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products access-schedules list prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products access-schedules list prd_42 --limit 200 --raw --json

# every page, projected
cademi content products access-schedules list prd_42 --all --jq '[.[] | {id}]'
```

### `cademi content products access-schedules update <product_id> <access_schedule_id>`

Update an access schedule · `PATCH /api/v3/products/{product_id}/access-schedules/{access_schedule_id}` · permission `access_schedules.update`, `tags.update (if field:name)`

Renames or restores an access schedule.

The access schedule takes its name from its tag, so changing `name` renames the tag everywhere it is used and also requires the `tags.update` permission. Setting `deleted` to `false` restores an access schedule from the trash.

Replicated access schedules are read-only and return the `replica_readonly` error code.

Send the current `ETag` value in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `access_schedule_id` — Public ID of the access schedule, prefixed with `acs_`. Example: `acs_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | Only `false` is accepted. Restores it from the trash; deleting is the DELETE operation. One of: `false` |
| `name` | string |  |  |

Full schema: `cademi commands content products access-schedules update --schema --json`

Legacy path: `cademi products access-schedules update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products access-schedules update prd_42 acs_7 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi content products access-schedules update prd_42 acs_7 --data @body.json --json
```

### `cademi content products access-schedules rules` — Manage content products access-schedules rules

Related: `cademi sales deliveries products`, `cademi users products access`

#### `cademi content products access-schedules rules create <product_id> <access_schedule_id>`

Create a release rule · `POST /api/v3/products/{product_id}/access-schedules/{access_schedule_id}/rules` · permission `access_schedules.update`

Creates the release rule for a node of the product tree that does not yet have its own rule. The node is identified by `level` (`product`, `section`, or `lesson`) and `target_id`; a `section` is identified by its module ID (`mod_`).

If the node already has its own rule, the request returns the `already_exists` error code; use the update operation to change it instead.

The `hidden` type is not accepted at the `product` level, and the `percentage` type is read-only and cannot be set.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `access_schedule_id` — Public ID of the access schedule, prefixed with `acs_`. Example: `acs_7`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `level` | string | yes | One of: `product`, `section`, `lesson` |
| `params` | object | yes |  |
| `target_id` | string | yes |  |
| `type` | string | yes | One of: `free`, `hidden`, `coming_soon`, `days_after_start`, `date`, `exam`, `percentage` |

Full schema: `cademi commands content products access-schedules rules create --schema --json`

Legacy path: `cademi products access-schedules rules create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products access-schedules rules create prd_42 acs_7 -f level=product -F 'params={...}' -f target_id=<target_id> -f type=free --json

# full body from a file
cademi content products access-schedules rules create prd_42 acs_7 --data @body.json --json
```

#### `cademi content products access-schedules rules delete <product_id> <access_schedule_id> <release_rule_id>`

Delete a release rule · `DELETE /api/v3/products/{product_id}/access-schedules/{access_schedule_id}/rules/{release_rule_id}` · permission `access_schedules.delete`

Removes the node's own release rule so that the node inherits the rule of its parent again.

At the `product` level there is no parent to inherit from, so removing the rule sets it to `free`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `access_schedule_id` — Public ID of the access schedule, prefixed with `acs_`. Example: `acs_7`
- `release_rule_id` — Public ID of the release rule, prefixed with `rul_`. Example: `rul_section_45`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi products access-schedules rules delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products access-schedules rules delete prd_42 acs_7 rul_section_45 --yes
```

#### `cademi content products access-schedules rules get <product_id> <access_schedule_id> <release_rule_id>`

Retrieve a release rule · `GET /api/v3/products/{product_id}/access-schedules/{access_schedule_id}/rules/{release_rule_id}` · permission `access_schedules.read`

Retrieves the release rule of a single node. Release rule IDs have the format `rul_<level>_<node>`.

Nodes without their own rule also return a result, with `explicit` set to `false` and `effective` containing the inherited rule.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `access_schedule_id` — Public ID of the access schedule, prefixed with `acs_`. Example: `acs_7`
- `release_rule_id` — Public ID of the release rule, prefixed with `rul_`. Example: `rul_section_45`

**Flag sets:** output

Legacy path: `cademi products access-schedules rules get`

**Examples:**

```bash
# get
cademi content products access-schedules rules get prd_42 acs_7 rul_section_45 --json
```

#### `cademi content products access-schedules rules list <product_id> <access_schedule_id>`

List release rules · `GET /api/v3/products/{product_id}/access-schedules/{access_schedule_id}/rules` · permission `access_schedules.read`

Lists the release rule of each node in the product tree for the access schedule.

Returns all nodes in a single response while preserving the standard collection response format.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `access_schedule_id` — Public ID of the access schedule, prefixed with `acs_`. Example: `acs_7`

**Flag sets:** output

Legacy path: `cademi products access-schedules rules list`

**Examples:**

```bash
# get
cademi content products access-schedules rules list prd_42 acs_7 --json
```

#### `cademi content products access-schedules rules update <product_id> <access_schedule_id> <release_rule_id>`

Update a release rule · `PATCH /api/v3/products/{product_id}/access-schedules/{access_schedule_id}/rules/{release_rule_id}` · permission `access_schedules.update`

Updates the `type` and `params` of a node that already has its own release rule. Other nodes are never changed.

The revision is tracked for the access schedule as a whole. Send the access schedule's current `ETag` value in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `access_schedule_id` — Public ID of the access schedule, prefixed with `acs_`. Example: `acs_7`
- `release_rule_id` — Public ID of the release rule, prefixed with `rul_`. Example: `rul_section_45`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `params` | object | yes |  |
| `type` | string | yes | One of: `free`, `hidden`, `coming_soon`, `days_after_start`, `date`, `exam`, `percentage` |

Full schema: `cademi commands content products access-schedules rules update --schema --json`

Legacy path: `cademi products access-schedules rules update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products access-schedules rules update prd_42 acs_7 rul_section_45 -F 'params={...}' -f type=free --json

# full body from a file
cademi content products access-schedules rules update prd_42 acs_7 rul_section_45 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
