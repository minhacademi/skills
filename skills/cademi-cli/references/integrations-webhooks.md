---
name: cademi-cli-integrations-webhooks
description: "Configure webhook endpoints and inspect notification delivery — `cademi integrations webhooks` (13 commands)"
metadata:
  cademi-cli: "0.3.1"
  cademi-api: "3.13.1"
---

# Integrations Webhooks Commands

> cademi 0.3.1, API 3.13.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi integrations` — Manage API credentials, events and webhook delivery.

Webhooks send account events to external endpoints. deliveries records
outgoing notifications and their attempts; replays request event replay.
Access-granting deliveries belong to sales deliveries.

**Related:** `cademi integrations events`

**See also in this domain:** `references/integrations.md`, `references/integrations-credentials.md`

Live catalog for this file: `cademi commands integrations webhooks --json` (offline, no credential needed).

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

### `cademi integrations webhooks create`

Create a webhook endpoint · `POST /api/v3/webhooks` · permission `webhooks.create`

Creates a webhook endpoint that receives events for the subscribed event types.

The destination URL must use HTTPS and point to a public host on port 443, 80, or 8443. Private IP addresses and other non-public destinations are rejected before the endpoint is created, in sandbox accounts too.

The response includes the signing secret. It is returned only once and cannot be retrieved again.

Setting `payload_detail` to `full` requires the personal data permissions described in that field. Without them, the request fails with `403` and the error code `personal_data_permission_required`, and the endpoint is not created.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `description` | string, nullable |  |  |
| `event_filters` | object, nullable |  | Restricts which events of each event type the endpoint receives, by fields of the event. Each key is one of the `event_types` of the endpoint, and its value holds the filters for that event type; event types without a key are delivered… |
| `event_types` | array of string | yes |  |
| `payload_detail` | string |  | How much detail each delivery carries, for all event types of the endpoint. `ids` (the default) sends the event `data` only, which identifies people by ID and carries no personal data stored by Cademí; its only free text is the note… |
| `resource_filters` | object, nullable |  | Restricts which events the endpoint receives, per resource type. Each key is the resource type of an event type, as listed in the event catalog (such as `product` or `lesson`), and each value lists the IDs of that type to deliver, as… |
| `url` | string | yes |  |

Full schema: `cademi commands integrations webhooks create --schema --json`

Legacy path: `cademi webhooks create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi integrations webhooks create -f 'event_types[]=<value>' -f url=<url> --json

# full body from a file
cademi integrations webhooks create --data @body.json --json
```

### `cademi integrations webhooks delete <webhook_id>`

Delete a webhook endpoint · `DELETE /api/v3/webhooks/{webhook_id}` · permission `webhooks.delete`

Deletes a webhook endpoint so that it no longer receives events.

Deliveries that are still pending are canceled. Deliveries that have already completed are not affected.

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi webhooks delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi integrations webhooks delete whk_01J8Z3 --yes
```

### `cademi integrations webhooks get <webhook_id>`

Retrieve a webhook endpoint · `GET /api/v3/webhooks/{webhook_id}` · permission `webhooks.read`

Retrieves a webhook endpoint, including its current `revision` and a `health` summary for the last 24 hours. The signing secret is never included.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the endpoint to avoid overwriting a newer version.

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`

**Flag sets:** output

Legacy path: `cademi webhooks get`

**Examples:**

```bash
# get
cademi integrations webhooks get whk_01J8Z3 --json
```

### `cademi integrations webhooks list`

List webhook endpoints · `GET /api/v3/webhooks` · permission `webhooks.read`

Returns the webhook endpoints of the current account, paginated by cursor. The collection can be filtered by status and event type. Signing secrets are never included.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--event-type <string>` — Only endpoints subscribed to this event type.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (created_at, -created_at)
- `--status <string>` — Only endpoints with this status. (active, inactive)

Legacy path: `cademi webhooks list`

**Examples:**

```bash
# list: one page, machine-readable
cademi integrations webhooks list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi integrations webhooks list --limit 200 --raw --json

# every page, projected
cademi integrations webhooks list --all --jq '[.[] | {id}]'
```

### `cademi integrations webhooks update <webhook_id>`

Update a webhook endpoint · `PATCH /api/v3/webhooks/{webhook_id}` · permission `webhooks.update`, `webhooks.activate (if field:status)`

Updates the URL, subscribed event types, resource filters, event filters, description, payload detail level, or status of a webhook endpoint.

Changing `status` requires the `webhooks.activate` permission in addition to `webhooks.update`. A new URL is subject to the same destination restrictions as the create operation.

Changing `payload_detail` to `full`, or changing the event types of an endpoint whose `payload_detail` is `full`, requires the personal data permissions described in that field. Without them, the request fails with `403` and the error code `personal_data_permission_required`, and nothing is changed. Deliveries already queued keep the `expanded` object built when they were created, and send it only while the endpoint still accepts that level: after lowering the level, a retry is sent without `expanded`.

_Full description: `cademi commands integrations webhooks update --json --jq '.commands[0].description'`_

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `description` | string, nullable |  |  |
| `event_filters` | object, nullable |  | Restricts which events of each event type the endpoint receives, by fields of the event. Each key is one of the `event_types` of the endpoint, and its value holds the filters for that event type; event types without a key are delivered… |
| `event_types` | array of string |  |  |
| `payload_detail` | string |  | How much detail each delivery carries, for all event types of the endpoint. `ids` (the default) sends the event `data` only, which identifies people by ID and carries no personal data stored by Cademí; its only free text is the note… |
| `resource_filters` | object, nullable |  | Restricts which events the endpoint receives, per resource type. Each key is the resource type of an event type, as listed in the event catalog (such as `product` or `lesson`), and each value lists the IDs of that type to deliver, as… |
| `status` | string |  | One of: `active`, `inactive` |
| `url` | string |  |  |

Full schema: `cademi commands integrations webhooks update --schema --json`

Legacy path: `cademi webhooks update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi integrations webhooks update whk_01J8Z3 -f description=<description> --if-match '"<etag>"' --json

# full body from a file
cademi integrations webhooks update whk_01J8Z3 --data @body.json --json
```

### `cademi integrations webhooks deliveries` — Inspect outgoing webhook deliveries and retry attempts

These deliveries are outgoing HTTP notifications. Inspect attempts for
delivery results; creating an attempt requests another delivery attempt.
Use list-all for the collection across webhook endpoints.

Related: `cademi integrations events`

#### `cademi integrations webhooks deliveries get <webhook_id> <webhook_delivery_id>`

Retrieve a webhook delivery · `GET /api/v3/webhooks/{webhook_id}/deliveries/{webhook_delivery_id}` · permission `webhooks.deliveries.read`

Retrieves a webhook delivery, including `attempts_count`, `next_attempt_at`, and its most recent attempt.

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`
- `webhook_delivery_id` — Public ID of the webhook delivery, prefixed with `whd_`. Example: `whd_01J8Z3`

**Flag sets:** output

Legacy path: `cademi webhooks deliveries get`

**Examples:**

```bash
# get
cademi integrations webhooks deliveries get whk_01J8Z3 whd_01J8Z3 --json
```

#### `cademi integrations webhooks deliveries list <webhook_id>`

List deliveries for a webhook endpoint · `GET /api/v3/webhooks/{webhook_id}/deliveries` · permission `webhooks.deliveries.read`

Returns the deliveries of a webhook endpoint, paginated by cursor. The collection can be filtered by status, event ID, event type, and creation time. Deliveries are retained for at least 60 days after they are created, together with their attempts. A delivery that has not reached a final status (`pending` or `in_progress`) is never removed.

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--created-after <string>` — Only items created after this date and time (ISO 8601).
- `--created-before <string>` — Only items created before this date and time (ISO 8601).
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--event-id <string>` — Only deliveries of the event with this public ID.
- `--event-type <string>` — Only deliveries of events of this type.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (created_at, -created_at)
- `--status <string>` — Only deliveries with this status. (pending, in_progress, delivered, failed, dead, canceled)

Legacy path: `cademi webhooks deliveries list`

**Examples:**

```bash
# list: one page, machine-readable
cademi integrations webhooks deliveries list whk_01J8Z3 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi integrations webhooks deliveries list whk_01J8Z3 --limit 200 --raw --json

# every page, projected
cademi integrations webhooks deliveries list whk_01J8Z3 --all --jq '[.[] | {id}]'
```

#### `cademi integrations webhooks deliveries list-all`

List all webhook deliveries · `GET /api/v3/webhooks/deliveries` · permission `webhooks.deliveries.read`

Returns webhook deliveries across all webhook endpoints accessible with the current credentials, paginated by cursor. The collection can be filtered by webhook endpoint, status, event type, and creation time. Deliveries are retained for at least 60 days after they are created, together with their attempts. A delivery that has not reached a final status (`pending` or `in_progress`) is never removed.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--created-after <string>` — Only items created after this date and time (ISO 8601).
- `--created-before <string>` — Only items created before this date and time (ISO 8601).
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--event-type <string>` — Only deliveries of events of this type.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (created_at, -created_at)
- `--status <string>` — Only deliveries with this status. (pending, in_progress, delivered, failed, dead, canceled)
- `--webhook-id <string>` — Only deliveries to the webhook endpoint with this public ID.

Legacy path: `cademi webhooks deliveries list-all`

**Examples:**

```bash
# list: one page, machine-readable
cademi integrations webhooks deliveries list-all --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi integrations webhooks deliveries list-all --limit 200 --raw --json

# every page, projected
cademi integrations webhooks deliveries list-all --all --jq '[.[] | {id}]'
```

### `cademi integrations webhooks deliveries attempts` — Manage integrations webhooks deliveries attempts

Related: `cademi integrations events`

#### `cademi integrations webhooks deliveries attempts create <webhook_id> <webhook_delivery_id>`

Resend a webhook delivery · `POST /api/v3/webhooks/{webhook_id}/deliveries/{webhook_delivery_id}/attempts` · permission `webhooks.replay`

Manually resends a completed webhook delivery. Only deliveries with status `delivered`, `failed`, or `dead` can be resent.

The destination URL is validated again before the delivery is queued. The delivery keeps the same event ID, and the new attempt is recorded when it is actually sent; previous attempts are preserved.

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`
- `webhook_delivery_id` — Public ID of the webhook delivery, prefixed with `whd_`. Example: `whd_01J8Z3`

**Flag sets:** output, idempotency

Legacy path: `cademi webhooks deliveries attempts create`

**Examples:**

```bash
# run
cademi integrations webhooks deliveries attempts create whk_01J8Z3 whd_01J8Z3 --json
```

#### `cademi integrations webhooks deliveries attempts get <webhook_id> <webhook_delivery_id> <attempt_id>`

Retrieve a delivery attempt · `GET /api/v3/webhooks/{webhook_id}/deliveries/{webhook_delivery_id}/attempts/{attempt_id}` · permission `webhooks.deliveries.read`

Retrieves a single attempt of a webhook delivery. The response excerpt captured from the receiving endpoint is masked.

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`
- `webhook_delivery_id` — Public ID of the webhook delivery, prefixed with `whd_`. Example: `whd_01J8Z3`
- `attempt_id` — Public ID of the attempt, prefixed with `wha_`. Example: `wha_42`

**Flag sets:** output

Legacy path: `cademi webhooks deliveries attempts get`

**Examples:**

```bash
# get
cademi integrations webhooks deliveries attempts get whk_01J8Z3 whd_01J8Z3 wha_42 --json
```

#### `cademi integrations webhooks deliveries attempts list <webhook_id> <webhook_delivery_id>`

List delivery attempts · `GET /api/v3/webhooks/{webhook_id}/deliveries/{webhook_delivery_id}/attempts` · permission `webhooks.deliveries.read`

Returns every attempt of a webhook delivery in a single response, ordered by attempt `number`, together with the time of the next scheduled attempt. Response excerpts captured from the receiving endpoint are masked. Attempts are retained with their delivery, for at least 60 days after the delivery is created.

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`
- `webhook_delivery_id` — Public ID of the webhook delivery, prefixed with `whd_`. Example: `whd_01J8Z3`

**Flag sets:** output

Legacy path: `cademi webhooks deliveries attempts list`

**Examples:**

```bash
# get
cademi integrations webhooks deliveries attempts list whk_01J8Z3 whd_01J8Z3 --json
```

### `cademi integrations webhooks replays` — Replay webhook events

Related: `cademi integrations events`

#### `cademi integrations webhooks replays create <webhook_id>`

Replay webhook events · `POST /api/v3/webhooks/{webhook_id}/replays` · permission `webhooks.replay`

Resends, in bulk, events that were already delivered to this webhook endpoint. The selection can be narrowed by `event_ids`, by an `occurred_after`/`occurred_before` time range, and by `event_types`. Only events that occurred before the request was received and are still retained are eligible.

The replay is processed asynchronously as an operation with one item per event. Use the `Location` header to track its progress and per-event results.

A replay adds new attempts to the existing deliveries; it never creates new deliveries. The request is rejected if no eligible events match the selection.

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`

**Flag sets:** output, body, idempotency, async

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `event_ids` | array of string, nullable |  |  |
| `event_types` | array of string, nullable |  |  |
| `occurred_after` | date-time, nullable |  |  |
| `occurred_before` | date-time, nullable |  |  |

Full schema: `cademi commands integrations webhooks replays create --schema --json`

Legacy path: `cademi webhooks replays create`

**Examples:**

```bash
# partial update
cademi integrations webhooks replays create whk_01J8Z3 -f occurred_after=<occurred_after> --wait --json

# full body from a file
cademi integrations webhooks replays create whk_01J8Z3 --data @body.json --wait --json
```

### `cademi integrations webhooks secret-rotations` — Rotate a webhook signing secret

Related: `cademi integrations events`

#### `cademi integrations webhooks secret-rotations create <webhook_id>`

Rotate a webhook signing secret · `POST /api/v3/webhooks/{webhook_id}/secret-rotations` · permission `webhooks.rotate_secret`

Generates a new signing secret for the webhook endpoint.

The previous secret remains valid for `overlap_hours` (24 hours by default, up to 72). During this period, each delivery carries one signature per active secret.

The new secret is returned only once. Retrying the request with the same `Idempotency-Key` does not return the secret again.

**Arguments:**
- `webhook_id` — Public ID of the webhook, prefixed with `whk_`. Example: `whk_01J8Z3`

**Flag sets:** output, body, idempotency

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `overlap_hours` | integer, nullable |  |  |

Full schema: `cademi commands integrations webhooks secret-rotations create --schema --json`

Legacy path: `cademi webhooks secret-rotations create`

**Examples:**

```bash
# partial update
cademi integrations webhooks secret-rotations create whk_01J8Z3 -F overlap_hours=1 --json

# full body from a file
cademi integrations webhooks secret-rotations create whk_01J8Z3 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
