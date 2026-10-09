---
name: cademi-cli-sales
description: "Connect gateways, deliveries and incoming sales — `cademi sales` (21 commands)"
metadata:
  cademi-cli: "0.3.1"
  cademi-api: "3.13.1"
---

# Sales Commands

> cademi 0.3.1, API 3.13.1. The live catalog is always `cademi commands <prefix> --json`.

Gateways send sales events. Deliveries connect gateway products to content
access. Inspect events and processing attempts to investigate sales handling.
Delivery access and user enrollment state can be inspected separately.

**Related:** `cademi users enrollments`, `cademi integrations webhooks`

**Group examples:**

```bash
cademi sales deliveries list
cademi sales events list
```

Live catalog for this file: `cademi commands sales --json` (offline, no credential needed).

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

## Commands

### `cademi sales deliveries` — Define content access granted by a sale or enrollment

A delivery defines covered products, access duration and release schedules.
products shows coverage; rules describes delivery rules. These deliveries
are access configurations. Webhook delivery attempts live in integrations.

Related: `cademi sales gateways`, `cademi users enrollments`, `cademi content products access-schedules`

```bash
cademi sales deliveries products list dlv_7
```

#### `cademi sales deliveries create`

Create a delivery · `POST /api/v3/sales/deliveries` · permission `deliveries.create`

Creates a delivery for a payment gateway from the gateway catalog. The selected gateway determines whether a secret is required.

The secret is write-only: responses indicate only whether a secret is configured.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `external_ids` | array of string |  |  |
| `gateway` | string | yes |  |
| `kind` | string |  | One of: `one_time`, `subscription` |
| `name` | string | yes |  |
| `products` | object |  | Products and showcases the delivery grants access to, in the same shape as the body of `PUT /sales/deliveries/{delivery_id}/products`. If omitted, the delivery starts without products. |
| `products.entries` | array of object |  |  |
| `products.ignored_external_product_ids` | array of string |  | Read-only: accepted for compatibility with the response shape and ignored. The ignored gateway product codes apply to the whole account and cannot be set through the API. |
| `products.object` | string |  | Accepted for compatibility with the response shape and ignored. One of: `delivery_products` |
| `schedules` | array of object |  |  |
| `secret` | string, nullable |  |  |
| `tags` | array of string |  |  |

Full schema: `cademi commands sales deliveries create --schema --json`

Legacy path: `cademi deliveries create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi sales deliveries create -f gateway=<gateway> -f name=<name> --json

# full body from a file
cademi sales deliveries create --data @body.json --json
```

#### `cademi sales deliveries get <delivery_id>`

Retrieve a delivery · `GET /api/v3/sales/deliveries/{delivery_id}` · permission `deliveries.read`

Archived deliveries can still be retrieved, which allows them to be restored through the update operation.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when modifying the delivery to avoid overwriting a newer version.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output

Legacy path: `cademi deliveries get`

**Examples:**

```bash
# get
cademi sales deliveries get dlv_42 --json
```

#### `cademi sales deliveries list`

List deliveries · `GET /api/v3/sales/deliveries` · permission `deliveries.read`

Credentials restricted to specific products only see deliveries whose products are all within the scope of the current credentials.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--external-id <string>` — Only deliveries whose 'external_ids' include this value.
- `--gateway <string>` — Only deliveries of the gateway with this public ID.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--product-id <string>` — Only deliveries that grant access to the product with this public ID.
- `--q <string>` — Text search term.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only deliveries with this status. (active, archived)

Legacy path: `cademi deliveries list`

**Examples:**

```bash
# list: one page, machine-readable
cademi sales deliveries list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi sales deliveries list --limit 200 --raw --json

# every page, projected
cademi sales deliveries list --all --jq '[.[] | {id}]'
```

#### `cademi sales deliveries update <delivery_id>`

Update a delivery · `PATCH /api/v3/sales/deliveries/{delivery_id}` · permission `deliveries.update`, `deliveries.archive (if field:status)`, `deliveries.archive (if field:deleted)`

Updates an existing delivery.

Changing `status` or `deleted` requires the `deliveries.archive` permission in addition to `deliveries.update`. `deleted` is an alias of `status`: `true` archives the delivery and `false` makes it active again, because an archived delivery is the deleted one. When both are sent, `status` prevails. Archiving a delivery stops it from processing new sales but does not revoke access from users who have already purchased.

To avoid overwriting a newer version, send the delivery's current `ETag` in the `If-Match` header.

Replicated deliveries are read-only; updating or archiving them returns `403` with the `replica_readonly` error code.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | Alias of `status`: `true` archives the delivery and `false` makes it active again. Requires `deliveries.archive`. |
| `external_ids` | array of string |  |  |
| `hidden` | boolean |  |  |
| `name` | string |  |  |
| `secret` | string, nullable |  | Gateway secret for the delivery. Set to `null` to remove the secret; omit the field to keep the current value. |
| `status` | string |  | One of: `active`, `archived` |

Full schema: `cademi commands sales deliveries update --schema --json`

Legacy path: `cademi deliveries update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi sales deliveries update dlv_42 -F deleted=true --if-match '"<etag>"' --json

# full body from a file
cademi sales deliveries update dlv_42 --data @body.json --json
```

### `cademi sales deliveries copies` — Duplicate a delivery

Related: `cademi sales gateways`, `cademi users enrollments`, `cademi content products access-schedules`

#### `cademi sales deliveries copies create <delivery_id>`

Duplicate a delivery · `POST /api/v3/sales/deliveries/{delivery_id}/copies` · permission `deliveries.create`

Creates a copy of the delivery and returns it.

The copy does not include the gateway product codes or the secret, because these identify the original delivery with the gateway. Configure them on the copy before using it to process sales.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output, idempotency

Legacy path: `cademi deliveries copies create`

**Examples:**

```bash
# run
cademi sales deliveries copies create dlv_42 --json
```

### `cademi sales deliveries products` — Manage sales deliveries products

Related: `cademi sales gateways`, `cademi users enrollments`, `cademi content products access-schedules`

#### `cademi sales deliveries products list <delivery_id>`

List delivery products · `GET /api/v3/sales/deliveries/{delivery_id}/products` · permission `deliveries.read`

Returns the set of products the delivery grants access to, including the access duration of each entry and the release schedule applied to each product.

`ignored_external_product_ids` is read-only. It lists the gateway product codes ignored during sales processing for the entire account, not only for this delivery, and cannot be changed through the API.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output

Legacy path: `cademi deliveries products list`

**Examples:**

```bash
# get
cademi sales deliveries products list dlv_42 --json
```

#### `cademi sales deliveries products update <delivery_id>`

Replace delivery products · `PUT /api/v3/sales/deliveries/{delivery_id}/products` · permission `deliveries.update`

Replaces the delivery's entire product set with the supplied entries.

The delivery is live: the new set applies to existing enrollments unless the user has an individual override (`schedule_id` lock or duration). Every referenced product must be accessible with the current credentials. Replicated deliveries are read-only; replacing their products returns `403` with the `replica_readonly` error code.

To avoid overwriting a newer version, send the delivery's current `ETag` in the `If-Match` header.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `entries` | array of object | yes |  |

Full schema: `cademi commands sales deliveries products update --schema --json`

Legacy path: `cademi deliveries products update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi sales deliveries products update dlv_42 --data '{"entries":[...]}' --json

# full body from a file
cademi sales deliveries products update dlv_42 --data @body.json --json
```

### `cademi sales deliveries rules` — Manage sales deliveries rules

Related: `cademi sales gateways`, `cademi users enrollments`, `cademi content products access-schedules`

#### `cademi sales deliveries rules list <delivery_id>`

List delivery release schedules · `GET /api/v3/sales/deliveries/{delivery_id}/rules` · permission `deliveries.read`

Returns the release schedules applied by the delivery, with one entry for each product it grants access to.

Products without a release schedule are included with a null `schedule_id`, meaning their content is released without a schedule.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output

Legacy path: `cademi deliveries rules list`

**Examples:**

```bash
# get
cademi sales deliveries rules list dlv_42 --json
```

#### `cademi sales deliveries rules update <delivery_id>`

Replace delivery release schedules · `PUT /api/v3/sales/deliveries/{delivery_id}/rules` · permission `deliveries.update`

Replaces the release schedules applied by the delivery.

The delivery is live: existing enrollments follow the new schedules unless the user has an individual `schedule_id` lock. That lock is changed per user in the product access update operation.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `entries` | array of object | yes |  |

Full schema: `cademi commands sales deliveries rules update --schema --json`

Legacy path: `cademi deliveries rules update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi sales deliveries rules update dlv_42 --data '{"entries":[...]}' --json

# full body from a file
cademi sales deliveries rules update dlv_42 --data @body.json --json
```

### `cademi sales deliveries tags` — Manage sales deliveries tags

Related: `cademi sales gateways`, `cademi users enrollments`, `cademi content products access-schedules`

#### `cademi sales deliveries tags list <delivery_id>`

List delivery tags · `GET /api/v3/sales/deliveries/{delivery_id}/tags` · permission `deliveries.read`

Returns the tags automatically applied to users who purchase through the delivery.

Deleted tags are not included, because they are no longer applied.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output

Legacy path: `cademi deliveries tags list`

**Examples:**

```bash
# get
cademi sales deliveries tags list dlv_42 --json
```

#### `cademi sales deliveries tags update <delivery_id>`

Replace delivery tags · `PUT /api/v3/sales/deliveries/{delivery_id}/tags` · permission `deliveries.update`

Replaces the set of tags automatically applied by the delivery.

The new set applies only to subsequent sales. Tags are not added to or removed from users who have already purchased; to apply the current tags to them, use the tag application operation.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `tag_ids` | array of string | yes |  |

Full schema: `cademi commands sales deliveries tags update --schema --json`

Legacy path: `cademi deliveries tags update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi sales deliveries tags update dlv_42 -f 'tag_ids[]=<value>' --json

# full body from a file
cademi sales deliveries tags update dlv_42 --data @body.json --json
```

### `cademi sales deliveries tags applications` — Apply delivery tags to existing users

Related: `cademi sales gateways`, `cademi users enrollments`, `cademi content products access-schedules`

#### `cademi sales deliveries tags applications create <delivery_id>`

Apply delivery tags to existing users · `POST /api/v3/sales/deliveries/{delivery_id}/tags/applications` · permission `deliveries.update`

Starts an asynchronous operation that applies the delivery's tags to every user with active access through the delivery. The operation processes one item per user.

Repeating the operation does not duplicate tags already applied to a user.

**Arguments:**
- `delivery_id` — Public ID of the delivery, prefixed with `dlv_`. Example: `dlv_42`

**Flag sets:** output, idempotency, async

Legacy path: `cademi deliveries tags applications create`

**Examples:**

```bash
# run
cademi sales deliveries tags applications create dlv_42 --wait --json
```

### `cademi sales events` — Inspect incoming sales events and processing attempts

Events are incoming sales notifications. Inspect processing attempts to
investigate failures. Creating an attempt requests reprocessing.

Related: `cademi sales transactions`, `cademi sales deliveries`

#### `cademi sales events get <event_id>`

Retrieve a sales event · `GET /api/v3/sales/events/{event_id}` · permission `sales_events.read`

Retrieves a sales event received from a payment gateway.

The original request headers sent by the gateway are never returned. The raw gateway payload is included only for credentials with the `sales_events.read_payload` permission, and is returned with sensitive data masked.

Imported data is not available through this operation; use the imports resource instead.

**Arguments:**
- `event_id` — Public ID of the event, prefixed with `sev_`. Example: `sev_42`

**Flag sets:** output

Legacy path: `cademi sales-events get`

**Examples:**

```bash
# get
cademi sales events get sev_42 --json
```

#### `cademi sales events list`

List sales events · `GET /api/v3/sales/events` · permission `sales_events.read`

Returns the sales events received from payment gateways.

The raw gateway payload is included only for credentials with the `sales_events.read_payload` permission, and payment and identity document fields are masked.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--delivery-id <string>` — Only events of the delivery with this public ID.
- `--gateway <string>` — Only events received from the gateway with this public ID.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--processing <string>` — Only events with this processing state. (queued, processed, failed, ignored)
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--received-after <string>` — Only items received after this date and time (ISO 8601).
- `--received-before <string>` — Only items received before this date and time (ISO 8601).
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only events with this sale status.
- `--transaction-id <string>` — Only events of the transaction with this public ID.

Legacy path: `cademi sales-events list`

**Examples:**

```bash
# list: one page, machine-readable
cademi sales events list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi sales events list --limit 200 --raw --json

# every page, projected
cademi sales events list --all --jq '[.[] | {id}]'
```

### `cademi sales events processing-attempts` — Manage sales events processing-attempts

Related: `cademi sales transactions`, `cademi sales deliveries`

#### `cademi sales events processing-attempts create <event_id>`

Create a processing attempt · `POST /api/v3/sales/events/{event_id}/processing-attempts` · permission `sales_events.reprocess`

Requests that the sales event be processed again. The event is queued for reprocessing; no access is granted directly by this request, and the event status changes once processing completes.

Requires an `Idempotency-Key` header.

**Arguments:**
- `event_id` — Public ID of the event, prefixed with `sev_`. Example: `sev_42`

**Flag sets:** output, idempotency, async

Legacy path: `cademi sales-events processing-attempts create`

**Examples:**

```bash
# run
cademi sales events processing-attempts create sev_42 --wait --json
```

#### `cademi sales events processing-attempts get <event_id> <attempt_id>`

Retrieve a processing attempt · `GET /api/v3/sales/events/{event_id}/processing-attempts/{attempt_id}` · permission `sales_events.read`

Retrieves a processing attempt of a sales event. The attempt must belong to the sales event specified in the path.

**Arguments:**
- `event_id` — Public ID of the event, prefixed with `sev_`. Example: `sev_42`
- `attempt_id` — Public ID of the attempt, prefixed with `pat_`. Example: `pat_7`

**Flag sets:** output

Legacy path: `cademi sales-events processing-attempts get`

**Examples:**

```bash
# get
cademi sales events processing-attempts get sev_42 pat_7 --json
```

#### `cademi sales events processing-attempts list <event_id>`

List processing attempts · `GET /api/v3/sales/events/{event_id}/processing-attempts` · permission `sales_events.read`

Returns all processing attempts of a sales event in a single response while preserving the standard collection response format.

**Arguments:**
- `event_id` — Public ID of the event, prefixed with `sev_`. Example: `sev_42`

**Flag sets:** output

Legacy path: `cademi sales-events processing-attempts list`

**Examples:**

```bash
# get
cademi sales events processing-attempts list sev_42 --json
```

### `cademi sales gateways` — Discover payment gateways supported by deliveries

Inspect the gateway catalog before creating a delivery. Gateway selection
determines integration configuration and whether a secret is required.

Related: `cademi sales deliveries`

#### `cademi sales gateways get <gateway_id>`

Retrieve a gateway · `GET /api/v3/sales/gateways/{gateway_id}` · permission `gateways.read`

Retrieves a payment gateway from the catalog. Gateway IDs use the `gtw_` prefix followed by the gateway key, for example `gtw_<alias>`; the key without the prefix is not accepted.

**Arguments:**
- `gateway_id` — Public ID of the gateway, prefixed with `gtw_`. Example: `gtw_hotmart`

**Flag sets:** output

Legacy path: `cademi gateways get`

**Examples:**

```bash
# get
cademi sales gateways get gtw_hotmart --json
```

#### `cademi sales gateways list`

List gateways · `GET /api/v3/sales/gateways` · permission `gateways.read`

Returns the catalog of supported payment gateways. The catalog is the same for every account, is returned in a single response, and does not include any account credentials.

**Flag sets:** output

Legacy path: `cademi gateways list`

**Examples:**

```bash
# get
cademi sales gateways list --json
```

### `cademi sales transactions` — Inspect sales transactions

Inspect transactions and use sales events for incoming notifications and
processing history. User enrollments describe the resulting access grants.

Related: `cademi sales events`, `cademi users enrollments`

#### `cademi sales transactions get <transaction_id>`

Retrieve a sales transaction · `GET /api/v3/sales/transactions/{transaction_id}` · permission `sales_events.read`

Retrieves a sales transaction, which aggregates the events of a single sale or subscription.

A transaction is addressed only by its own ID. IDs of later events in the same transaction return `404 Not Found`.

**Arguments:**
- `transaction_id` — Public ID of the transaction, prefixed with `trx_`. Example: `trx_42`

**Flag sets:** output

Legacy path: `cademi sales-transactions get`

**Examples:**

```bash
# get
cademi sales transactions get trx_42 --json
```

#### `cademi sales transactions list`

List sales transactions · `GET /api/v3/sales/transactions` · permission `sales_events.read`

Returns sales transactions. Each transaction aggregates the events of a single sale or subscription. A transaction's ID is derived from its first event and does not change when new events are received.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--delivery-id <string>` — Only transactions of the delivery with this public ID.
- `--external-id <string>` — Only transactions with this 'external_id'.
- `--gateway <string>` — Only transactions from the gateway with this public ID.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only transactions with this sale status.
- `--updated-after <string>` — Only items updated after this date and time (ISO 8601).
- `--user-id <string>` — Only transactions of the user with this public ID.

Legacy path: `cademi sales-transactions list`

**Examples:**

```bash
# list: one page, machine-readable
cademi sales transactions list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi sales transactions list --limit 200 --raw --json

# every page, projected
cademi sales transactions list --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
