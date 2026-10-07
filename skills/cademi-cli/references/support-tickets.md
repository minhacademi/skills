---
name: cademi-cli-support-tickets
description: "Manage support conversations and their messages — `cademi support tickets` (9 commands)"
metadata:
  cademi-cli: "0.2.7"
  cademi-api: "3.12.1"
---

# Support Tickets Commands

> cademi 0.2.7, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi support` — Handle student comments, questions and support tickets.

Tickets represent support conversations. Inspect messages within a ticket
and use departments to organize the support queue.

**Related:** `cademi support tickets replies list`, `cademi support tickets replies create`, `cademi support departments`

**Related settings:**
- `cademi settings support` — Controls availability of the student support channel and how administrators open conversations. (`cademi settings support get|update`)

**See also in this domain:** `references/support.md`, `references/support-comments.md`, `references/support-faqs.md`, `references/support-questions.md`

**Group examples:**

```bash
cademi support tickets list --status open
cademi support tickets replies list tkt_42 --limit 50 --raw
cademi support tickets replies create tkt_42 -f text=Hello
```

Live catalog for this file: `cademi commands support tickets --json` (offline, no credential needed).

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

### `cademi support tickets create`

Create a ticket · `POST /api/v3/support/tickets` · permission `tickets.create`

Opens a support ticket on behalf of the specified user. The ticket is owned by that user, but its first message is attributed to the credential that made the request, not to the user.

If `user_id`, `department_id`, or `product_id` does not reference a resource accessible with the current credentials, the request is rejected with a validation error for that field.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `department_id` | string, nullable |  |  |
| `message` | object | yes |  |
| `message.file_ids` | array of string |  |  |
| `message.text` | string | yes |  |
| `product_id` | string, nullable |  |  |
| `subject` | string | yes |  |
| `user_id` | string | yes |  |

Full schema: `cademi commands support tickets create --schema --json`

Legacy path: `cademi tickets create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi support tickets create -F 'message={...}' -f message.text=<message.text> -f subject=<subject> -f user_id=<user_id> --json

# full body from a file
cademi support tickets create --data @body.json --json
```

### `cademi support tickets get <ticket_id>`

Retrieve a ticket · `GET /api/v3/support/tickets/{ticket_id}` · permission `tickets.read`

Retrieves a support ticket by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the ticket to avoid overwriting a newer version.

If the credentials are restricted to specific users or products, tickets outside that scope return `404 Not Found`, the same response as a ticket that does not exist.

**Arguments:**
- `ticket_id` — Public ID of the ticket, prefixed with `tkt_`. Example: `tkt_42`

**Flag sets:** output

Legacy path: `cademi tickets get`

**Examples:**

```bash
# get
cademi support tickets get tkt_42 --json
```

### `cademi support tickets list`

List tickets · `GET /api/v3/support/tickets` · permission `tickets.read`

Returns the support tickets in the account, using cursor-based pagination.

The collection can be filtered by user, product, support department, status, a text search, and last update time, and sorted by `updated_at`.

If the credentials are restricted to specific users or products, only tickets within that scope are returned.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--department-id <string>` — Only tickets assigned to the department with this public ID.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--product-id <string>` — Only tickets about the product with this public ID.
- `--q <string>` — Text search term.
- `--queue <string>` — Inbox queue: open (awaiting staff), answered (awaiting user), or closed. (open, answered, closed)
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (updated_at, -updated_at)
- `--status <string>` — Lifecycle status: open or closed. (open, closed)
- `--updated-after <string>` — Only items updated after this date and time (ISO 8601).
- `--user-id <string>` — Only tickets opened by the user with this public ID.

Legacy path: `cademi tickets list`

**Examples:**

```bash
# list: one page, machine-readable
cademi support tickets list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi support tickets list --limit 200 --raw --json

# every page, projected
cademi support tickets list --all --jq '[.[] | {id}]'
```

### `cademi support tickets update <ticket_id>`

Update a ticket · `PATCH /api/v3/support/tickets/{ticket_id}` · permission `tickets.update`, `tickets.close (if field:status)`

Updates the subject or support department of a ticket, or closes and reopens it through `status`.

Changing `status` requires the `tickets.close` permission in addition to `tickets.update`.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `ticket_id` — Public ID of the ticket, prefixed with `tkt_`. Example: `tkt_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `department_id` | string, nullable |  |  |
| `status` | string |  | One of: `open`, `closed` |
| `subject` | string |  |  |

Full schema: `cademi commands support tickets update --schema --json`

Legacy path: `cademi tickets update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi support tickets update tkt_42 -f department_id=<department_id> --if-match '"<etag>"' --json

# full body from a file
cademi support tickets update tkt_42 --data @body.json --json
```

### `cademi support tickets replies` — Manage support tickets replies

Related: `cademi support tickets replies list`, `cademi support tickets replies create`, `cademi support departments`

Related settings:
- `cademi settings support` — Controls availability of the student support channel and how administrators open conversations. (`cademi settings support get|update`)

#### `cademi support tickets replies create <ticket_id>`

Create a ticket reply · `POST /api/v3/support/tickets/{ticket_id}/replies` · permission `tickets.update`

Adds a reply to the ticket conversation on behalf of the support team. The reply is attributed to the credential that made the request (`author.kind` is `credential`).

**Arguments:**
- `ticket_id` — Public ID of the ticket, prefixed with `tkt_`. Example: `tkt_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `file_ids` | array of string |  |  |
| `text` | string, nullable |  | Reply text. May be omitted when `file_ids` is present. |

Full schema: `cademi commands support tickets replies create --schema --json`

Legacy path: `cademi tickets replies create`

**Examples:**

```bash
# partial update
cademi support tickets replies create tkt_42 -f text=<text> --json

# full body from a file
cademi support tickets replies create tkt_42 --data @body.json --json
```

#### `cademi support tickets replies delete <ticket_id> <reply_id>`

Delete a ticket reply · `DELETE /api/v3/support/tickets/{ticket_id}/replies/{reply_id}` · permission `tickets.update`

Removes a support team reply from the ticket conversation. The reply is no longer visible to the user, and the related in-app notification is removed.

Only replies written by the support team (`author.kind` of `admin` or `credential`) can be deleted. Attempting to delete any other message returns `403` with the `permission_denied` error code and `details[].field` set to `author.kind`.

**Arguments:**
- `ticket_id` — Public ID of the ticket, prefixed with `tkt_`. Example: `tkt_42`
- `reply_id` — Public ID of the reply, prefixed with `trp_`. Example: `trp_9`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi tickets replies delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi support tickets replies delete tkt_42 trp_9 --yes
```

#### `cademi support tickets replies get <ticket_id> <reply_id>`

Retrieve a ticket reply · `GET /api/v3/support/tickets/{ticket_id}/replies/{reply_id}` · permission `tickets.read`

Retrieves a single message from the ticket conversation. The message must belong to the ticket in the path; otherwise the API returns `404 Not Found`.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the reply to avoid overwriting a newer version.

**Arguments:**
- `ticket_id` — Public ID of the ticket, prefixed with `tkt_`. Example: `tkt_42`
- `reply_id` — Public ID of the reply, prefixed with `trp_`. Example: `trp_9`

**Flag sets:** output

Legacy path: `cademi tickets replies get`

**Examples:**

```bash
# get
cademi support tickets replies get tkt_42 trp_9 --json
```

#### `cademi support tickets replies list <ticket_id>`

List ticket replies · `GET /api/v3/support/tickets/{ticket_id}/replies` · permission `tickets.read`

Returns the ticket conversation in chronological order, oldest message first, using cursor-based pagination. The collection includes messages from the user and from the support team. Messages are ordered by created_at and then id. There is no newest-first, reverse, or last-page query: obtaining the latest messages requires traversing to the end. Use page.next_cursor unchanged to continue; null or empty means the end. A small limit bounds each page, not the total conversation.

**Arguments:**
- `ticket_id` — Public ID of the ticket, prefixed with `tkt_`. Example: `tkt_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Opaque continuation token from page.next_cursor of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of messages per page; minimum: 1; maximum: 200; API default: 50
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

Legacy path: `cademi tickets replies list`

**Examples:**

```bash
# list: one page, machine-readable
cademi support tickets replies list tkt_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi support tickets replies list tkt_42 --limit 200 --raw --json

# every page, projected
cademi support tickets replies list tkt_42 --all --jq '[.[] | {id}]'
```

#### `cademi support tickets replies update <ticket_id> <reply_id>`

Update a ticket reply · `PATCH /api/v3/support/tickets/{ticket_id}/replies/{reply_id}` · permission `tickets.update`

Corrects the text of a support team reply. The original creation time of the reply is preserved, and the user is not notified again.

Only replies written by the support team (`author.kind` of `admin` or `credential`) can be edited. Attempting to edit any other message returns `403` with the `permission_denied` error code and `details[].field` set to `author.kind`.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `ticket_id` — Public ID of the ticket, prefixed with `tkt_`. Example: `tkt_42`
- `reply_id` — Public ID of the reply, prefixed with `trp_`. Example: `trp_9`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `text` | string | yes |  |

Full schema: `cademi commands support tickets replies update --schema --json`

Legacy path: `cademi tickets replies update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi support tickets replies update tkt_42 trp_9 -f text=<text> --json

# full body from a file
cademi support tickets replies update tkt_42 trp_9 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
