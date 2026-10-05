---
name: cademi-cli-integrations
description: "Manage API credentials, events and webhook delivery — `cademi integrations` (10 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Integrations Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

Credentials control API access. Events describe account activity, webhooks
deliver notifications, and event-streams configure streaming access.
Requests provide API request history; schema retrieves the OpenAPI document.

**Related:** `cademi auth`, `cademi listen`, `cademi api`, `cademi account capabilities`

**See also in this domain:** `references/integrations-credentials.md`, `references/integrations-webhooks.md`

Live catalog for this file: `cademi commands integrations --json` (offline, no credential needed).

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

### `cademi integrations event-streams` — Manage event stream connections

Configure and inspect streaming access. Use listen to consume Server-Sent
Events in the terminal or forward them to a local endpoint.

Related: `cademi listen`, `cademi integrations events`

#### `cademi integrations event-streams create`

Create an event stream · `POST /api/v3/event-streams` · permission `event_streams.manage`

Creates an event stream that delivers the account's public events over Server-Sent Events (SSE), with the same payloads and ordering as the events collection. Optional filters restrict the stream to specific event types and resource types. Use `start` to choose whether the stream also delivers retained events published before it was created (`earliest`, the default) or only events published afterwards (`now`).

Requires an `Idempotency-Key` header. The credential must also have the `events.read` permission; otherwise the request fails with `permission_denied`.

The number of active event streams is limited per credential and per account. Creating a stream beyond these limits returns `429` with the `too_many_streams` error code in the standard error envelope, without the rate-limit fields.

**Flag sets:** output, body, idempotency

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `expires_in_hours` | integer |  |  |
| `filters` | object |  |  |
| `filters.event_types` | array of string |  |  |
| `filters.resource_types` | array of string |  |  |
| `start` | string |  | Where the stream starts. `earliest` (default) also delivers the retained events published before the stream was created. `now` starts after the most recent event available when the stream is created, so only later events are delivered.… |

Full schema: `cademi commands integrations event-streams create --schema --json`

Legacy path: `cademi event-streams create`

**Examples:**

```bash
# partial update
cademi integrations event-streams create -F expires_in_hours=1 --json

# full body from a file
cademi integrations event-streams create --data @body.json --json
```

#### `cademi integrations event-streams delete <event_stream_id>`

Revoke an event stream · `DELETE /api/v3/event-streams/{event_stream_id}` · permission `event_streams.manage`

Revokes the event stream and closes any open connection to it shortly afterward.

The stream is not removed: it remains available through the retrieve and list operations with the `revoked` status.

**Arguments:**
- `event_stream_id` — Public ID of the event stream, prefixed with `str_`. Example: `str_01J8Z3`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi event-streams delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi integrations event-streams delete str_01J8Z3 --yes
```

#### `cademi integrations event-streams get <event_stream_id>`

Retrieve an event stream · `GET /api/v3/event-streams/{event_stream_id}` · permission `event_streams.read`

Returns the event stream's status, filters, delivery cursor, and connection state.

Revoked and expired event streams remain retrievable.

**Arguments:**
- `event_stream_id` — Public ID of the event stream, prefixed with `str_`. Example: `str_01J8Z3`

**Flag sets:** output

Legacy path: `cademi event-streams get`

**Examples:**

```bash
# get
cademi integrations event-streams get str_01J8Z3 --json
```

#### `cademi integrations event-streams list`

List event streams · `GET /api/v3/event-streams` · permission `event_streams.read`

Lists the event streams created by the current credential, using cursor-based pagination.

Event streams created by other credentials are never included, even within the same account.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

Legacy path: `cademi event-streams list`

**Examples:**

```bash
# list: one page, machine-readable
cademi integrations event-streams list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi integrations event-streams list --limit 200 --raw --json

# every page, projected
cademi integrations event-streams list --all --jq '[.[] | {id}]'
```

### `cademi integrations event-streams events` — Connect to an event stream

Related: `cademi listen`, `cademi integrations events`

#### `cademi integrations event-streams events get <event_stream_id>`

Connect to an event stream · `GET /api/v3/event-streams/{event_stream_id}/events` · permission `event_streams.read`

Opens the Server-Sent Events connection for an event stream. Send `Accept: text/event-stream` and authenticate with the `Authorization` header; credentials in the query string are not accepted.

Each message carries the event's public ID in `id`, its type in `event`, and in `data` the same JSON returned when retrieving the event. Without a `Last-Event-ID` header, delivery resumes from the stream's stored cursor; with it, delivery resumes after that event. Keep-alive comments are sent periodically.

Only one connection per stream is allowed at a time. The server closes the connection after a maximum duration, or when the stream expires, is revoked, or the credential is revoked; reconnect with `Last-Event-ID` to continue. Each connection extends the stream's expiration.

Delivery has a latency of a few seconds and is not real-time.

**Arguments:**
- `event_stream_id` — Public ID of the event stream, prefixed with `str_`. Example: `str_01J8Z3`

Legacy path: `cademi event-streams events get`

**Examples:**

```bash
# get
cademi integrations event-streams events get str_01J8Z3 --json
```

### `cademi integrations events` — Inspect account events available to integrations

Inspect event records here. Use webhooks to deliver notifications or listen
to consume a live stream from the terminal.

Related: `cademi integrations webhooks`, `cademi listen`

#### `cademi integrations events get <event_id>`

Retrieve an event · `GET /api/v3/events/{event_id}` · permission `events.read`

Retrieves a public event by its public ID.

Malformed IDs, internal event types, and events that are not accessible with the current credentials all return `404 Not Found`.

**Arguments:**
- `event_id` — Public ID of the event, prefixed with `evt_`. Example: `evt_01J8Z3TESTE`

**Flag sets:** output

Legacy path: `cademi events get`

**Examples:**

```bash
# get
cademi integrations events get evt_01J8Z3TESTE --json
```

#### `cademi integrations events list`

List events · `GET /api/v3/events` · permission `events.read`

Lists the public events of the current account. Internal event types are never returned.

Results are paginated by cursor and sorted by ID in ascending order unless `sort` specifies otherwise. The collection can be filtered by event type, resource type, resource ID, and occurrence time.

Newly recorded events become visible after a short delay of a few seconds, so the most recent events may not appear immediately.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--occurred-after <string>` — Only items that occurred after this date and time (ISO 8601).
- `--occurred-before <string>` — Only items that occurred before this date and time (ISO 8601).
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--resource-id <string>` — Only events about the resource with this public ID.
- `--resource-type <string>` — Only events about resources of this type.
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--type <stringSlice>` — Only events of these types. Repeat the parameter to match any of several types.

Legacy path: `cademi events list`

**Examples:**

```bash
# list: one page, machine-readable
cademi integrations events list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi integrations events list --limit 200 --raw --json

# every page, projected
cademi integrations events list --all --jq '[.[] | {id}]'
```

### `cademi integrations requests` — Inspect API request history

Inspect requests when investigating API calls and integration failures.

Related: `cademi account audit-entries`, `cademi doctor`

#### `cademi integrations requests get <request_id>`

Retrieve a request · `GET /api/v3/requests/{request_id}` · permission `audit.read`

Retrieves a logged API request together with the audit entries it produced.

`data.request` describes the request itself, and `data.effects` lists the resulting audit entries in ascending order of `occurred_at`.

Requests that do not exist or are not accessible with the current credentials return `404 Not Found` with the `not_found` error code.

**Arguments:**
- `request_id` — Public ID of the request. Example: `01J8Z3TESTE`

**Flag sets:** output

Legacy path: `cademi requests get`

**Examples:**

```bash
# get
cademi integrations requests get 01J8Z3TESTE --json
```

#### `cademi integrations requests list`

List requests · `GET /api/v3/requests` · permission `audit.read`

Lists the authenticated API requests made on the current account, most recent first by default.

Results are paginated with a cursor. The collection can be filtered by outcome, action code, affected resource (`resource_type` and `resource_id`), and time range (`occurred_after` and `occurred_before`).

**Flag sets:** output-basic

**Flags:**
- `--action-code <string>` — Only requests with this action code.
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--occurred-after <string>` — Only items that occurred after this date and time (ISO 8601).
- `--occurred-before <string>` — Only items that occurred before this date and time (ISO 8601).
- `--outcome <stringSlice>` — Only requests with these outcomes. Repeat the parameter to match any of several outcomes.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--resource-id <string>` — Only requests that affected the resource with this public ID.
- `--resource-type <string>` — Only requests that affected resources of this type.
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (occurred_at, -occurred_at)

Legacy path: `cademi requests list`

**Examples:**

```bash
# list: one page, machine-readable
cademi integrations requests list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi integrations requests list --limit 200 --raw --json

# every page, projected
cademi integrations requests list --all --jq '[.[] | {id}]'
```

### `cademi integrations schema` — Retrieve the API's OpenAPI contract

Retrieves the API contract. Use commands --json for the CLI's own command
catalog, including canonical paths, legacy paths and available flags.

Related: `cademi commands`, `cademi api`

#### `cademi integrations schema get`

Retrieve the OpenAPI document · `GET /api/v3/openapi.json`

Returns the latest published release of the API v3 OpenAPI document. Any authenticated credential can retrieve it.

The response includes an `ETag` derived from the document content. Send this value in the `If-None-Match` header to receive `304 Not Modified` when the document has not changed.

**Flag sets:** output

Legacy path: `cademi openapi get`

**Examples:**

```bash
# get
cademi integrations schema get --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
