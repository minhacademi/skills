---
name: cademi-cli-users-imports
description: "Import student records and inspect import processing — `cademi users imports` (10 commands)"
metadata:
  cademi-cli: "0.2.5"
  cademi-api: "3.12.0"
---

# Users Imports Commands

> cademi 0.2.5, API 3.12.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi users` — Manage students, enrollments and learning progress.

Imports create or update student data in bulk. Inspect the operation's
required input and processing results before retrying a failed import.

**Related:** `cademi files`, `cademi operations`

**See also in this domain:** `references/users-learning.md`, `references/users-profile.md`, `references/users.md`, `references/users-products.md`

Live catalog for this file: `cademi commands users imports --json` (offline, no credential needed).

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

### `cademi users imports create`

Create an import · `POST /api/v3/imports` · permission `imports.create`

Creates a user import from a spreadsheet that was previously uploaded with `purpose=import`.

Creating an import does not process any rows. Request an analysis next, then start a processing attempt once the analysis is complete.

If imports are disabled for the current account, the request is rejected with the `feature_disabled` error code.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `duplicate_policy` | string |  | One of: `skip`, `update` |
| `file_id` | string | yes |  |
| `type` | string | yes | One of: `students_access` |

Full schema: `cademi commands users imports create --schema --json`

Legacy path: `cademi imports create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users imports create -f file_id=<file_id> -f type=students_access --json

# full body from a file
cademi users imports create --data @body.json --json
```

### `cademi users imports delete <import_id>`

Delete an import · `DELETE /api/v3/imports/{import_id}` · permission `imports.delete`

Moves the import to the trash. Only imports that have not been processed can be deleted.

Deleting an import does not undo its effects: rows already queued for processing continue, and enrollments already granted remain in place.

**Arguments:**
- `import_id` — Public ID of the import, prefixed with `imp_`. Example: `imp_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi imports delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi users imports delete imp_42 --yes
```

### `cademi users imports get <import_id>`

Retrieve an import · `GET /api/v3/imports/{import_id}` · permission `imports.read`

Retrieves an import by its public ID, including imports in the trash, which are returned with `deleted` set to `true`.

File-level errors, such as an unreadable spreadsheet or a missing required column, are reported in `failure_reason`.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the import to avoid overwriting a newer version.

**Arguments:**
- `import_id` — Public ID of the import, prefixed with `imp_`. Example: `imp_42`

**Flag sets:** output

Legacy path: `cademi imports get`

**Examples:**

```bash
# get
cademi users imports get imp_42 --json
```

### `cademi users imports list`

List imports · `GET /api/v3/imports` · permission `imports.read`

Returns user imports created from spreadsheets, using cursor-based pagination.

Imports in the trash are excluded by default. Set `deleted=true` to list them.

The `rows` summary of each import reflects its analysis. To follow the processing state of individual rows, use the list import rows operation.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--created-after <string>` — Only items created after this date and time (ISO 8601).
- `--created-before <string>` — Only items created before this date and time (ISO 8601).
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only imports with this status. (pending, analyzed, processing, processed, failed)
- `--type <string>` — Only imports of this type.

Legacy path: `cademi imports list`

**Examples:**

```bash
# list: one page, machine-readable
cademi users imports list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users imports list --limit 200 --raw --json

# every page, projected
cademi users imports list --all --jq '[.[] | {id}]'
```

### `cademi users imports update <import_id>`

Update an import · `PATCH /api/v3/imports/{import_id}` · permission `imports.update`

Updates the duplicate policy of an import or restores it from the trash by setting `deleted` to `false`. To delete an import, use the delete operation.

The duplicate policy can only be changed before the import is processed.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `import_id` — Public ID of the import, prefixed with `imp_`. Example: `imp_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | One of: `false` |
| `duplicate_policy` | string |  | One of: `skip`, `update` |

Full schema: `cademi commands users imports update --schema --json`

Legacy path: `cademi imports update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi users imports update imp_42 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi users imports update imp_42 --data @body.json --json
```

### `cademi users imports analyses` — Analyze an import

Related: `cademi files`, `cademi operations`

#### `cademi users imports analyses create <import_id>`

Analyze an import · `POST /api/v3/imports/{import_id}/analyses` · permission `imports.analyze`

Starts an asynchronous analysis of the import spreadsheet and returns the operation that tracks it. The `Location` header points to that operation.

Requires the `imports.analyze` permission. An import can be analyzed again before it is processed; the new analysis replaces the previous one.

**Arguments:**
- `import_id` — Public ID of the import, prefixed with `imp_`. Example: `imp_42`

**Flag sets:** output, idempotency, async

Legacy path: `cademi imports analyses create`

**Examples:**

```bash
# run
cademi users imports analyses create imp_42 --wait --json
```

### `cademi users imports processing-attempts` — Manage users imports processing-attempts

Related: `cademi files`, `cademi operations`

#### `cademi users imports processing-attempts create <import_id>`

Create a processing attempt · `POST /api/v3/imports/{import_id}/processing-attempts` · permission `imports.process`

Queues the rows that are ready for processing. Requires the `imports.process` permission.

A successful response confirms that the rows were queued, not that enrollments were granted. Rows are processed asynchronously; use the list import rows operation to follow the result of each row.

The import must have a completed analysis with no errors. Use `mode=initial` (default) for the first attempt, and `mode=retry` to requeue only the rows whose processing failed.

**Arguments:**
- `import_id` — Public ID of the import, prefixed with `imp_`. Example: `imp_42`

**Flag sets:** output, body, idempotency, async

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `mode` | string |  | One of: `initial`, `retry` |

Full schema: `cademi commands users imports processing-attempts create --schema --json`

Legacy path: `cademi imports processing-attempts create`

**Examples:**

```bash
# partial update
cademi users imports processing-attempts create imp_42 -f mode=initial --wait --json

# full body from a file
cademi users imports processing-attempts create imp_42 --data @body.json --wait --json
```

#### `cademi users imports processing-attempts get <import_id> <attempt_id>`

Retrieve a processing attempt · `GET /api/v3/imports/{import_id}/processing-attempts/{attempt_id}` · permission `imports.read`

Retrieves a processing attempt of the specified import. An attempt that belongs to a different import returns `404 Not Found`.

If the attempt failed, `error` explains why the rows could not be queued. Errors for individual rows are reported by the list import rows operation.

**Arguments:**
- `import_id` — Public ID of the import, prefixed with `imp_`. Example: `imp_42`
- `attempt_id` — Public ID of the attempt, prefixed with `pat_`. Example: `pat_7`

**Flag sets:** output

Legacy path: `cademi imports processing-attempts get`

**Examples:**

```bash
# get
cademi users imports processing-attempts get imp_42 pat_7 --json
```

#### `cademi users imports processing-attempts list <import_id>`

List processing attempts · `GET /api/v3/imports/{import_id}/processing-attempts` · permission `imports.read`

Returns the processing attempts requested through the API for the specified import. Processing started outside the API does not appear in this list.

Returns all attempts in a single response while preserving the standard collection response format.

**Arguments:**
- `import_id` — Public ID of the import, prefixed with `imp_`. Example: `imp_42`

**Flag sets:** output

Legacy path: `cademi imports processing-attempts list`

**Examples:**

```bash
# get
cademi users imports processing-attempts list imp_42 --json
```

### `cademi users imports rows` — List import rows

Related: `cademi files`, `cademi operations`

#### `cademi users imports rows list <import_id>`

List import rows · `GET /api/v3/imports/{import_id}/rows` · permission `imports.read`

Returns the analyzed rows of an import, using cursor-based pagination.

The `email` of each row is returned only when the current credentials also have the `users.read_personal` permission.

The `processing` object reports the processing state of each row after a processing attempt has queued it.

**Arguments:**
- `import_id` — Public ID of the import, prefixed with `imp_`. Example: `imp_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--q <string>` — Text search term.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (row_number, -row_number)
- `--status <string>` — Only rows with this status. (ready, not_ready, ignored)

Legacy path: `cademi imports rows list`

**Examples:**

```bash
# list: one page, machine-readable
cademi users imports rows list imp_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users imports rows list imp_42 --limit 200 --raw --json

# every page, projected
cademi users imports rows list imp_42 --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
