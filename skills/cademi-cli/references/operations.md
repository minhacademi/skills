---
name: cademi-cli-operations
description: "Inspect and wait for asynchronous API operations — `cademi operations` (9 commands)"
metadata:
  cademi-cli: "0.2.3"
  cademi-api: "3.10.1"
---

# Operations Commands

> cademi 0.2.3, API 3.10.1. The live catalog is always `cademi commands <prefix> --json`.

Some writes return an operation instead of a completed result. Use wait
with its operation ID, or --wait on commands that start asynchronous work.

**Group examples:**

```bash
cademi operations wait op_42
```

Live catalog for this file: `cademi commands operations --json` (offline, no credential needed).

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

### `cademi operations get <operation_id>`

Retrieve an operation · `GET /api/v3/operations/{operation_id}` · permission `operations.read`

Retrieves the status and progress of an asynchronous operation. Poll this endpoint until the operation reaches a terminal status: `succeeded`, `partially_succeeded`, `failed`, or `canceled`.

**Arguments:**
- `operation_id` — Public ID of the operation, prefixed with `op_`. Example: `op_01J8Z3`

**Flag sets:** output, async

**Examples:**

```bash
# get
cademi operations get op_01J8Z3 --json
```

### `cademi operations items <operation_id>`

List operation items · `GET /api/v3/operations/{operation_id}/items` · permission `operations.read`

Lists the items of a batch operation in the order they were submitted, using cursor-based pagination.

**Arguments:**
- `operation_id` — Public ID of the operation, prefixed with `op_`. Example: `op_01J8Z3`

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
cademi operations items op_01J8Z3 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi operations items op_01J8Z3 --limit 200 --raw --json

# every page, projected
cademi operations items op_01J8Z3 --all --jq '[.[] | {id}]'
```

### `cademi operations list`

List operations · `GET /api/v3/operations` · permission `operations.read`

Lists the asynchronous operations of the current account, using cursor-based pagination.

The collection can be filtered by status and type and sorted by creation date.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (created_at, -created_at)
- `--status <string>` — Only operations with this status. (queued, running, succeeded, partially_succeeded, failed, canceled)
- `--type <string>` — Only operations of this type. (credential.revoke, sandbox.reset, webhook.replay, product.publish_all, product.copy, module.publish_all, module.copy, email.send_test, lesson.copy, access.apply_all, sandbox.run_scenario, configuration.apply, delivery.apply_tags, sales_event.reprocess, diamond.reprocess_membership, import.analyze, import.process, export.generate)

**Examples:**

```bash
# list: one page, machine-readable
cademi operations list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi operations list --limit 200 --raw --json

# every page, projected
cademi operations list --all --jq '[.[] | {id}]'
```

### `cademi operations update <operation_id>`

Cancel an operation · `PATCH /api/v3/operations/{operation_id}` · permission `operations.manage`

Requests cancellation of an operation. Set `cancellation_requested` to `true` and optionally provide a `reason`.

Cancellation is cooperative: a queued operation is canceled immediately, while a running operation stops between items. Items that have already been processed are not reverted.

Setting `cancellation_requested` to `false` has no effect; a cancellation request cannot be withdrawn. Operations that have already reached a terminal status return `operation_not_cancelable`.

**Arguments:**
- `operation_id` — Public ID of the operation, prefixed with `op_`. Example: `op_01J8Z3`

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `cancellation_requested` | boolean | yes |  |
| `reason` | string, nullable |  |  |

Full schema: `cademi commands operations update --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi operations update op_01J8Z3 -F cancellation_requested=true --wait --json

# full body from a file
cademi operations update op_01J8Z3 --data @body.json --wait --json
```

### `cademi operations wait <operation_id>`

Wait for an asynchronous operation to finish

Poll the operation (every 2s for the first 30s, then every 10s) until it is
succeeded, partially_succeeded, failed or canceled, and print it.
Exits with 0 for succeeded AND partially_succeeded: inspect status and item errors
before treating every item as successful (cademi operations items <operation_id>).
Exits with 1 for failed, canceled or a local wait timeout; Ctrl-C exits with 130.
A timeout or Ctrl-C stops local polling without canceling the server operation.
Polling API errors use the exit codes in cademi guide errors --help.
The terminal operation is printed on stdout even when failed or canceled.

**Arguments:**
- `operation_id`

**Flag sets:** output-basic

**Flags:**
- `--timeout <duration>` — Give up after a duration, e.g. 30s or 2m (default: no limit)

### `cademi operations attempts` — Manage operations attempts

#### `cademi operations attempts create <operation_id>`

Resume an operation · `POST /api/v3/operations/{operation_id}/attempts` · permission `operations.manage`

Resumes a failed operation by queuing a new execution attempt. The request has no body.

Only operations in the `failed` status that still have recoverable items (items in the `pending` status, or failed items with `retryable` set to `true`) can be resumed. Otherwise, the API returns `operation_not_resumable`. Items that have already succeeded are not executed again.

Repeating the request with the same `Idempotency-Key` returns the same operation without resuming it again.

**Arguments:**
- `operation_id` — Public ID of the operation, prefixed with `op_`. Example: `op_01J8Z3`

**Flag sets:** output, idempotency, async

**Examples:**

```bash
# run
cademi operations attempts create op_01J8Z3 --wait --json
```

#### `cademi operations attempts get <operation_id> <attempt_id>`

Retrieve an operation attempt · `GET /api/v3/operations/{operation_id}/attempts/{attempt_id}` · permission `operations.read`

Retrieves a single execution attempt of an operation, identified by its sequence number within that operation.

**Arguments:**
- `operation_id` — Public ID of the operation, prefixed with `op_`. Example: `op_01J8Z3`
- `attempt_id` — Sequence number of the attempt within the operation. Example: `1`

**Flag sets:** output

**Examples:**

```bash
# get
cademi operations attempts get op_01J8Z3 1 --json
```

#### `cademi operations attempts list <operation_id>`

List operation attempts · `GET /api/v3/operations/{operation_id}/attempts` · permission `operations.read`

Lists the execution attempts of an operation, most recent first, using cursor-based pagination.

**Arguments:**
- `operation_id` — Public ID of the operation, prefixed with `op_`. Example: `op_01J8Z3`

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
cademi operations attempts list op_01J8Z3 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi operations attempts list op_01J8Z3 --limit 200 --raw --json

# every page, projected
cademi operations attempts list op_01J8Z3 --all --jq '[.[] | {id}]'
```

### `cademi operations batches` — Create a batch operation

#### `cademi operations batches create`

Create a batch operation · `POST /api/v3/operations/batches` · permission `operations.manage`

Creates an asynchronous batch operation. All items in the batch are processed with the same operation `type`, which also determines the structure of each item's `payload`.

The operation is accepted with a `Location` header pointing to it and processed asynchronously. Poll the retrieve operation to track its progress.

In addition to `operations.manage`, the credential must have the permission required by the selected operation type. Repeating the request with the same `Idempotency-Key` returns the same operation instead of creating a new one.

The request is rejected with `operation_type_unknown` for an unsupported type, `duplicate_item_key` when item keys are not unique, `too_many_items` when the batch exceeds the maximum number of items for the type, and `operation_backlog_exceeded` when the account has too many unfinished operations.

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | yes |  |
| `type` | string | yes | Operation type code. Must be one of the supported operation types. |

Full schema: `cademi commands operations batches create --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi operations batches create --data '{"items":[...]}' -f type=<type> --wait --json

# full body from a file
cademi operations batches create --data @body.json --wait --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
