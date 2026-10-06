---
name: cademi-cli-files
description: "Manage stored files and upload sessions — `cademi files` (14 commands)"
metadata:
  cademi-cli: "0.2.3"
  cademi-api: "3.10.1"
---

# Files Commands

> cademi 0.2.3, API 3.10.1. The live catalog is always `cademi commands <prefix> --json`.

Files are stored assets referenced by other resources. uploads provides
low-level upload sessions. Use upload and download for complete file transfers.

**Related:** `cademi upload`, `cademi download`, `cademi content products lessons attachments`

Live catalog for this file: `cademi commands files --json` (offline, no credential needed).

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

### `cademi files delete <file_id>`

Delete a file · `DELETE /api/v3/files/{file_id}` · permission `files.write`

Permanently deletes the file from storage and removes it from the account's file inventory. The file is not moved to the trash and cannot be restored.

If the storage service is unavailable, the file is not deleted and the request can be retried.

The `Idempotency-Key` header is optional for this operation.

**Arguments:**
- `file_id` — Public ID of the file, prefixed with `file_`. Example: `file_42`

**Flag sets:** output, idempotency, confirm

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi files delete file_42 --yes
```

### `cademi files get <file_id>`

Retrieve a file · `GET /api/v3/files/{file_id}` · permission `files.read`

Retrieves a file by its public ID, including its content type and SHA-256 checksum.

A file uploaded through the API that no resource uses within 24 hours of its upload is deleted, and retrieving it then returns `404`. Until then, `expires_at` shows the deadline.

**Arguments:**
- `file_id` — Public ID of the file, prefixed with `file_`. Example: `file_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi files get file_42 --json
```

### `cademi files list`

List files · `GET /api/v3/files` · permission `files.read`

Lists the files stored in the current account, using cursor-based pagination. Results are sorted by creation date, newest first.

The collection can be filtered by purpose and by creation date range.

A file uploaded through the API that no resource uses within 24 hours of its upload is deleted and stops being listed. Until then, `expires_at` shows the deadline.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--created-after <string>` — Only items created after this date and time (ISO 8601).
- `--created-before <string>` — Only items created before this date and time (ISO 8601).
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--purpose <string>` — Only files uploaded for this purpose.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (-created_at)

**Examples:**

```bash
# list: one page, machine-readable
cademi files list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi files list --limit 200 --raw --json

# every page, projected
cademi files list --all --jq '[.[] | {id}]'
```

### `cademi files download-links` — Create a download link

#### `cademi files download-links create <file_id>`

Create a download link · `POST /api/v3/files/{file_id}/download-links` · permission `files.read`

Issues a short-lived, pre-signed URL for downloading the file.

The link lifetime is set with `ttl_seconds`. When omitted, it defaults to the maximum of 900 seconds. Set `inline` to `true` to have the browser open the file instead of downloading it as an attachment (the default), and use `download_name` to set the file name the browser uses when saving it.

This operation requires the `Idempotency-Key` header. Replaying a request with the same key returns the original response, including the original URL, which may already have expired. To obtain a valid link, send a new request with a different key.

**Arguments:**
- `file_id` — Public ID of the file, prefixed with `file_`. Example: `file_42`

**Flag sets:** output, body, idempotency

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `download_name` | string, nullable |  |  |
| `inline` | boolean |  |  |
| `ttl_seconds` | integer |  |  |

Full schema: `cademi commands files download-links create --schema --json`

**Examples:**

```bash
# partial update
cademi files download-links create file_42 -f download_name=<download_name> --json

# full body from a file
cademi files download-links create file_42 --data @body.json --json
```

### `cademi files exports` — Request and inspect data exports

Exports produce downloadable data. Inspect the export's status and result;
file operations and download handle the resulting files.

Related: `cademi files`, `cademi download`, `cademi operations`

#### `cademi files exports create`

Create an export · `POST /api/v3/exports` · permission `exports.create`

Requests a data export. The file is generated asynchronously by an `export.generate` operation, whose ID is returned in `operation_id`; use the retrieve operation to follow the export status.

Each entry in `columns` must belong to the column catalog of the selected `resource`, and `filters` accepts the same keys as the users collection: `product_id`, `tag_id`, `showcase_id`, `delivery_id`, `access`, `type`, and `status`. Unknown columns or filters are rejected with a validation error.

Requesting personal data columns (`email`, `document`, or `phone`) also requires the `users.read_personal` permission. Without it, the request is rejected with `permission_denied` instead of silently omitting those columns.

The generated file is a UTF-8 CSV. It does not expire: it is kept until the export is deleted.

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `columns` | array of string | yes |  |
| `filters` | object |  |  |
| `resource` | string | yes | One of: `users`, `user_activity` |
| `user_id` | string |  | Public ID of the user whose activity is exported. Required when `resource` is `user_activity`; a missing value, or one that does not reference a user accessible with the current credentials, is rejected with a validation error on this… |

Full schema: `cademi commands files exports create --schema --json`

Legacy path: `cademi exports create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi files exports create -f 'columns[]=<value>' -f resource=users --wait --json

# full body from a file
cademi files exports create --data @body.json --wait --json
```

#### `cademi files exports delete <export_id>`

Delete an export · `DELETE /api/v3/exports/{export_id}` · permission `exports.delete`

Permanently deletes the generated file immediately and moves the export to the trash.

Generated files do not expire. Use this operation to remove exported data that is no longer needed.

**Arguments:**
- `export_id` — Public ID of the export, prefixed with `exp_`. Example: `exp_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi exports delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi files exports delete exp_42 --yes
```

#### `cademi files exports get <export_id>`

Retrieve an export · `GET /api/v3/exports/{export_id}` · permission `exports.read`

Retrieves an export by its public ID.

When generation finishes, `status` becomes `completed` and the response includes `file_id` and `total`. If generation fails, `status` becomes `failed` and `failure_reason` describes the cause.

Generated files do not expire: the file is kept until the export is deleted, and `expires_at` is `null`. The `expired` status applies only to exports generated while files were retained for 7 days; their file no longer exists and `file_id` is `null`.

The export never includes a download URL. To download the file, request a download link for `file_id` from the files API.

**Arguments:**
- `export_id` — Public ID of the export, prefixed with `exp_`. Example: `exp_42`

**Flag sets:** output

Legacy path: `cademi exports get`

**Examples:**

```bash
# get
cademi files exports get exp_42 --json
```

#### `cademi files exports list`

List exports · `GET /api/v3/exports` · permission `exports.read`

Lists the exports of the current account using cursor-based pagination. The collection can be filtered by `resource`. Exports requested outside the API are also listed, with `file_id` set to `null`.

Exports never include a download URL. To download a file, request a short-lived download link for its `file_id` from the files API.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--resource <string>` — Only exports of this kind of data. (users, user_activity)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)

Legacy path: `cademi exports list`

**Examples:**

```bash
# list: one page, machine-readable
cademi files exports list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi files exports list --limit 200 --raw --json

# every page, projected
cademi files exports list --all --jq '[.[] | {id}]'
```

### `cademi files uploads` — Manage low-level upload sessions and completion

These operations expose upload sessions and their completion. Use upload
to transfer a local file through the complete upload workflow.

Related: `cademi upload`, `cademi files`

#### `cademi files uploads create`

Create an upload · `POST /api/v3/uploads` · permission `files.write`

Opens a multipart upload session and returns the upload plan: the part size (`part_size`) and the number of parts (`parts_total`).

The API never receives file content. Request presigned URLs through the upload parts operation and send each part directly to the storage service, then complete the upload to obtain the file.

Each `purpose` accepts a fixed list of file extensions (the last suffix of `filename`, case-insensitive) and a maximum size:

_Full description: `cademi commands files uploads create --json --jq '.commands[0].description'`_

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `content_type` | string, nullable |  |  |
| `filename` | string | yes | File name. Its last extension must be in the list accepted by `purpose`. |
| `purpose` | string | yes | What the file is for: `image`, `pdf`, `import`, `editor`, or `document`. It determines which extensions and which maximum size the upload accepts: `image` takes `.jpg`, `.jpeg`, `.png`, `.gif`, and `.webp` up to 10 MiB; `pdf` takes `.pdf`… |
| `sha256` | string, nullable |  |  |
| `size_bytes` | integer | yes | Declared file size in bytes, up to the limit of `purpose`. |

Full schema: `cademi commands files uploads create --schema --json`

Legacy path: `cademi uploads create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi files uploads create -f filename=<filename> -f purpose=image -F size_bytes=1 --json

# full body from a file
cademi files uploads create --data @body.json --json
```

#### `cademi files uploads get <upload_id>`

Retrieve an upload · `GET /api/v3/uploads/{upload_id}` · permission `files.read`

Retrieves an upload session by its public ID, including its current status and expiration time. When the session is `completed`, the response includes the resulting `file`.

To see which parts the storage service has already received, use the list upload parts operation.

**Arguments:**
- `upload_id` — Public ID of the upload, prefixed with `upl_`. Example: `upl_01J8Z3`

**Flag sets:** output

Legacy path: `cademi uploads get`

**Examples:**

```bash
# get
cademi files uploads get upl_01J8Z3 --json
```

#### `cademi files uploads update <upload_id>`

Cancel an upload · `PATCH /api/v3/uploads/{upload_id}` · permission `files.write`

Cancels an open upload session. Set `status` to `cancelled`; no other change is supported.

Sessions that are already completed, cancelled, failed, or expired cannot be cancelled and return `state_conflict`. To remove a file produced by a completed upload, use the delete file operation.

**Arguments:**
- `upload_id` — Public ID of the upload, prefixed with `upl_`. Example: `upl_01J8Z3`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `status` | string | yes | One of: `cancelled` |

Full schema: `cademi commands files uploads update --schema --json`

Legacy path: `cademi uploads update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi files uploads update upl_01J8Z3 -f status=cancelled --json

# full body from a file
cademi files uploads update upl_01J8Z3 --data @body.json --json
```

### `cademi files uploads completions` — Complete an upload

Related: `cademi upload`, `cademi files`

#### `cademi files uploads completions create <upload_id>`

Complete an upload · `POST /api/v3/uploads/{upload_id}/completions` · permission `files.write`

Completes a multipart upload after all parts have been sent to the storage service. Provide every part number together with the `ETag` returned by the storage service for that part.

The storage service verifies the integrity of the assembled content. On success, the session moves to `completed` and the response includes the resulting `file`.

The file must be used by a resource within 24 hours (for example as a lesson attachment, an image, an import source or a link in lesson, FAQ, legal term or question reply content). Otherwise it is deleted; `file.expires_at` shows the deadline.

If the content does not match the declared size, checksum, or content type, the API returns `upload_integrity_mismatch` with the exact cause in `details[].reason`. The session is then permanently marked as `failed`, and a new upload session is required.

_Full description: `cademi commands files uploads completions create --json --jq '.commands[0].description'`_

**Arguments:**
- `upload_id` — Public ID of the upload, prefixed with `upl_`. Example: `upl_01J8Z3`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `parts` | array of object | yes |  |

Full schema: `cademi commands files uploads completions create --schema --json`

Legacy path: `cademi uploads completions create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi files uploads completions create upl_01J8Z3 --data '{"parts":[...]}' --json

# full body from a file
cademi files uploads completions create upl_01J8Z3 --data @body.json --json
```

### `cademi files uploads parts` — Manage files uploads parts

Related: `cademi upload`, `cademi files`

#### `cademi files uploads parts create <upload_id>`

Create upload part URLs · `POST /api/v3/uploads/{upload_id}/parts` · permission `files.write`

Returns presigned URLs for uploading the requested parts of an open upload session.

Send each part with a `PUT` request directly to its URL; the content never passes through the API. Each URL is valid for 15 minutes. Keep the `ETag` returned by the storage service for each part, because it is required to complete the upload.

Presigned URLs are short-lived secrets. Do not log or share them.

**Arguments:**
- `upload_id` — Public ID of the upload, prefixed with `upl_`. Example: `upl_01J8Z3`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `part_numbers` | array of integer | yes |  |

Full schema: `cademi commands files uploads parts create --schema --json`

Legacy path: `cademi uploads parts create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi files uploads parts create upl_01J8Z3 --data '{"part_numbers":[...]}' --json

# full body from a file
cademi files uploads parts create upl_01J8Z3 --data @body.json --json
```

#### `cademi files uploads parts list <upload_id>`

List upload parts · `GET /api/v3/uploads/{upload_id}/parts` · permission `files.read`

Lists the parts that the storage service has already received for the upload session.

Use this operation to resume an interrupted upload without resending parts that were already received. The result always reflects the current state reported by the storage service.

**Arguments:**
- `upload_id` — Public ID of the upload, prefixed with `upl_`. Example: `upl_01J8Z3`

**Flag sets:** output

Legacy path: `cademi uploads parts list`

**Examples:**

```bash
# get
cademi files uploads parts list upl_01J8Z3 --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
