---
name: cademi-cli-content-products-lessons
description: "Manage lessons and their content within a product — `cademi content products lessons` (16 commands)"
metadata:
  cademi-cli: "0.2.3"
  cademi-api: "3.10.1"
---

# Content Products Lessons Commands

> cademi 0.2.3, API 3.10.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi content` — Organize showcases, products and learning content.

Commands require a product ID. List lessons with --module-id to select a
module, or --unassigned for lessons outside modules. Inspect content for
lesson bodies and attachments for associated files. Blog lessons are created
without a module; other product formats require a module when creating a lesson.

**Related:** `cademi content products lessons content get`, `cademi content products modules`, `cademi content products exams`

**See also in this domain:** `references/content-banners.md`, `references/content-products.md`, `references/content-products-access-schedules.md`, `references/content-products-exams.md`, `references/content-products-modules.md`, `references/content-products-taxonomies.md`, `references/content-showcases.md`

**Group examples:**

```bash
cademi content products lessons list prd_42 --module-id mod_7
cademi content products lessons content get prd_42 les_9
```

Live catalog for this file: `cademi commands content products lessons --json` (offline, no credential needed).

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

### `cademi content products lessons create <product_id>`

Create a lesson · `POST /api/v3/products/{product_id}/lessons` · permission `lessons.create`

Creates a lesson in the product, optionally inside a module. New lessons are created as drafts.

`position` is 1-based among the lesson's siblings; a value greater than the number of siblings places the lesson at the end.

The `Location` header of the response contains the URL of the new lesson. Lessons cannot be created in replicated products; such requests return `replica_readonly`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `format` | string | yes | One of: `standard`, `quiz`, `live_youtube`, `live_vimeo` |
| `module_id` | string, nullable |  |  |
| `name` | string | yes |  |
| `position` | integer |  |  |
| `summary` | string, nullable |  |  |

Full schema: `cademi commands content products lessons create --schema --json`

Legacy path: `cademi lessons create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products lessons create prd_42 -f format=standard -f name=<name> --json

# full body from a file
cademi content products lessons create prd_42 --data @body.json --json
```

### `cademi content products lessons delete <product_id> <lesson_id>`

Delete a lesson · `DELETE /api/v3/products/{product_id}/lessons/{lesson_id}` · permission `lessons.delete`

Moves the lesson to the trash. Trashed lessons can be restored by setting `deleted` to `false` with the update operation.

A lesson that is the source of connected lessons cannot be deleted and returns `state_conflict`; disconnect those lessons first. Replicated lessons are read-only and cannot be deleted; such requests return `replica_readonly`.

Send the lesson's current `ETag` in the `If-Match` header to avoid deleting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output, idempotency, confirm

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

Legacy path: `cademi lessons delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products lessons delete prd_42 les_7 --yes
```

### `cademi content products lessons get <product_id> <lesson_id>`

Retrieve a lesson · `GET /api/v3/products/{product_id}/lessons/{lesson_id}` · permission `lessons.read`

Retrieves a lesson by its public ID. The lesson body is available through the content endpoint.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when changing the lesson to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output

Legacy path: `cademi lessons get`

**Examples:**

```bash
# get
cademi content products lessons get prd_42 les_7 --json
```

### `cademi content products lessons list <product_id>`

List lessons · `GET /api/v3/products/{product_id}/lessons` · permission `lessons.read`

Returns the lessons of a product, using cursor pagination. Lesson bodies are not included; use GET /products/{product_id}/lessons/{lesson_id}/content to retrieve them.

The collection can be filtered by module, status, format, and name. Set `unassigned` to `true` to return only lessons that are not in a module. Trashed lessons are excluded by default; set `deleted` to `true` to return only lessons in the trash.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `--format <string>` — Only lessons in this format. (standard, quiz, live_youtube, live_vimeo)
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200; API default: 50
- `--module-id <string>` — Only lessons in the module with this public ID.
- `--q <string>` — Text search term.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only lessons with this status. (published, draft)
- `--unassigned` — When 'true', returns only lessons that are not in a module.

Legacy path: `cademi lessons list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products lessons list prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products lessons list prd_42 --limit 200 --raw --json

# every page, projected
cademi content products lessons list prd_42 --all --jq '[.[] | {id}]'
```

### `cademi content products lessons update <product_id> <lesson_id>`

Update a lesson · `PATCH /api/v3/products/{product_id}/lessons/{lesson_id}` · permission `lessons.update`, `lessons.publish (if field:status)`

Updates only the fields included in the request.

Changing `status` requires the `lessons.publish` permission. Moving the lesson to another product with `target_product_id` requires the `lessons.update` permission on the destination product; lessons that are connected to a source lesson, or that are the source of connected lessons, cannot be moved. Setting `deleted` to `false` restores a lesson from the trash. The exam set in `exam_id` must belong to the same product.

`position` is 1-based among the lesson's siblings, in the destination when the lesson is moved; a value greater than the number of siblings places the lesson at the end. A lesson moved to another module or product without `position` is placed at the end.

Replicated lessons accept only `status`; any other field is rejected with `replica_readonly`. Lessons cannot be moved into replicated products.

_Full description: `cademi commands content products lessons update --json --jq '.commands[0].description'`_

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | Only `false` is accepted. Restores it from the trash; deleting is the DELETE operation. One of: `false` |
| `exam_id` | string, nullable |  | Public ID of the exam displayed by this lesson. Set to `null` to remove the exam from the lesson. |
| `image` | string, nullable |  |  |
| `module_id` | string, nullable |  |  |
| `name` | string |  |  |
| `pinned` | boolean |  |  |
| `position` | integer |  |  |
| `settings` | object |  |  |
| `status` | string |  | One of: `published`, `draft` |
| `summary` | string, nullable |  |  |
| `target_product_id` | string |  |  |

Full schema: `cademi commands content products lessons update --schema --json`

Legacy path: `cademi lessons update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products lessons update prd_42 les_7 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi content products lessons update prd_42 les_7 --data @body.json --json
```

### `cademi content products lessons attachments` — Manage content products lessons attachments

Related: `cademi content products lessons content get`, `cademi content products modules`, `cademi content products exams`

#### `cademi content products lessons attachments delete <product_id> <lesson_id> <file_id>`

Remove an attachment · `DELETE /api/v3/products/{product_id}/lessons/{lesson_id}/attachments/{file_id}` · permission `lessons.update`

Removes the attachment from the lesson body. The underlying file is not deleted.

Attachments without a `file_id` cannot be removed with this operation; remove them by replacing the lesson content instead.

Send the lesson's current `ETag` in the `If-Match` header to avoid overwriting a newer version. Replicated lessons are read-only, and requests to change them return `replica_readonly`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`
- `file_id` — Public ID of the file, prefixed with `file_`. Example: `file_9`

**Flag sets:** output, idempotency, confirm

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

Legacy path: `cademi lessons attachments delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products lessons attachments delete prd_42 les_7 file_9 --yes
```

#### `cademi content products lessons attachments list <product_id> <lesson_id>`

List lesson attachments · `GET /api/v3/products/{product_id}/lessons/{lesson_id}/attachments` · permission `lessons.read`

Returns the files and images attached to the lesson. Attachments are the file and image blocks of the lesson body, so they also appear in the lesson content.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output

Legacy path: `cademi lessons attachments list`

**Examples:**

```bash
# get
cademi content products lessons attachments list prd_42 les_7 --json
```

#### `cademi content products lessons attachments update <product_id> <lesson_id> <file_id>`

Add or update an attachment · `PUT /api/v3/products/{product_id}/lessons/{lesson_id}/attachments/{file_id}` · permission `lessons.update`

Attaches a file to the lesson body, or updates the title and kind of an existing attachment for the same file. The operation is idempotent per file ID: it returns `201` when the attachment is added and `200` when an existing attachment is updated.

Send the lesson's current `ETag` in the `If-Match` header to avoid overwriting a newer version. Replicated lessons are read-only, and requests to change them return `replica_readonly`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`
- `file_id` — Public ID of the file, prefixed with `file_`. Example: `file_9`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `kind` | string |  | One of: `file`, `image` |
| `title` | string, nullable |  |  |

Full schema: `cademi commands content products lessons attachments update --schema --json`

Legacy path: `cademi lessons attachments update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products lessons attachments update prd_42 les_7 file_9 -f kind=file --if-match '"<etag>"' --json

# full body from a file
cademi content products lessons attachments update prd_42 les_7 file_9 --data @body.json --json
```

### `cademi content products lessons connection` — Manage content products lessons connection

Related: `cademi content products lessons content get`, `cademi content products modules`, `cademi content products exams`

#### `cademi content products lessons connection delete <product_id> <lesson_id>`

Disconnect a lesson · `DELETE /api/v3/products/{product_id}/lessons/{lesson_id}/connection` · permission `lessons.connect`

Disconnects the lesson from its source lesson, so it no longer mirrors that lesson's content.

If the lesson is not connected, the request fails with `state_conflict`. Replicated lessons are read-only, and requests to change them return `replica_readonly`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi lessons connection delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products lessons connection delete prd_42 les_7 --yes
```

#### `cademi content products lessons connection get <product_id> <lesson_id>`

Retrieve a lesson connection · `GET /api/v3/products/{product_id}/lessons/{lesson_id}/connection` · permission `lessons.read`

Returns whether the lesson is connected to a source lesson and, if so, identifies the source lesson, its product, and its module. Lessons without a connection return `connected` set to `false`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output

Legacy path: `cademi lessons connection get`

**Examples:**

```bash
# get
cademi content products lessons connection get prd_42 les_7 --json
```

#### `cademi content products lessons connection update <product_id> <lesson_id>`

Connect a lesson · `PUT /api/v3/products/{product_id}/lessons/{lesson_id}/connection` · permission `lessons.connect`

Connects the lesson to the source lesson given by `source_lesson_id`. A connected lesson mirrors the content of its source lesson and takes on its format, name, summary, and thumbnail.

Requires the `lessons.connect` permission on the lesson being connected. The source lesson must be readable with the current credentials; otherwise the API returns `404 Not Found`. The source lesson must have the same format, must not be the lesson itself, and must not be connected to another lesson.

Send the lesson's current `ETag` in the `If-Match` header to avoid overwriting a newer version. Replicated lessons are read-only, and requests to change them return `replica_readonly`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `source_lesson_id` | string | yes |  |

Full schema: `cademi commands content products lessons connection update --schema --json`

Legacy path: `cademi lessons connection update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products lessons connection update prd_42 les_7 -f source_lesson_id=<source_lesson_id> --json

# full body from a file
cademi content products lessons connection update prd_42 les_7 --data @body.json --json
```

### `cademi content products lessons content` — Read and edit lesson bodies

Lesson bodies are separate from the metadata returned by lessons get.
Supply product and lesson IDs. Accepted content depends on lesson format.

Related: `cademi content products lessons attachments`

#### `cademi content products lessons content get <product_id> <lesson_id>`

Retrieve lesson content · `GET /api/v3/products/{product_id}/lessons/{lesson_id}/content` · permission `lessons.read`

Returns the lesson body in Editor.js block format, along with its video and embed settings. For connected lessons, the content comes from the source lesson and `inherited` is `true`.

The response includes an `ETag` representing the current revision of the lesson. Send this value in the `If-Match` header when changing the lesson to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output

Legacy path: `cademi lessons content get`

**Examples:**

```bash
# get
cademi content products lessons content get prd_42 les_7 --json
```

#### `cademi content products lessons content update <product_id> <lesson_id>`

Replace lesson content · `PUT /api/v3/products/{product_id}/lessons/{lesson_id}/content` · permission `lessons.update`

Replaces the lesson body and, when `video` is included, the lesson video. The `inherited` field returned by the retrieve operation is accepted and ignored, so retrieved content can be sent back as is.

`raw` blocks are not accepted, embed blocks must use a recognized provider, and quiz lessons do not accept a video. The request body must not exceed 1 MiB.

The content of a connected lesson belongs to its source lesson and cannot be changed; such requests fail with `state_conflict`. Replicated lessons are read-only, and requests to change them return `replica_readonly`.

Send the lesson's current `ETag` in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `body` | object |  |  |
| `body.blocks` | array of object |  |  |
| `video` | object, nullable |  |  |
| `video.url` | string |  |  |

Full schema: `cademi commands content products lessons content update --schema --json`

Legacy path: `cademi lessons content update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products lessons content update prd_42 les_7 -f video.url=<video.url> --if-match '"<etag>"' --json

# full body from a file
cademi content products lessons content update prd_42 les_7 --data @body.json --json
```

### `cademi content products lessons copies` — Duplicate a lesson

Related: `cademi content products lessons content get`, `cademi content products modules`, `cademi content products exams`

#### `cademi content products lessons copies create <product_id> <lesson_id>`

Duplicate a lesson · `POST /api/v3/products/{product_id}/lessons/{lesson_id}/copies` · permission `lessons.create`

Starts an asynchronous operation that copies the lesson into the product given by `target_product_id`, optionally into a specific module and under a new name. The copy is always created as a draft.

This operation requires an `Idempotency-Key` header. The response returns the operation, and the `Location` header points to it; retrieve the operation to track its progress and obtain the new lesson once it completes.

Requires the `lessons.create` permission on the destination product. Replicated lessons cannot be copied, and lessons cannot be copied into replicated products; both cases return `replica_readonly`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `module_id` | string, nullable |  |  |
| `name` | string, nullable |  |  |
| `target_product_id` | string | yes |  |

Full schema: `cademi commands content products lessons copies create --schema --json`

Legacy path: `cademi lessons copies create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products lessons copies create prd_42 les_7 -f target_product_id=<target_product_id> --wait --json

# full body from a file
cademi content products lessons copies create prd_42 les_7 --data @body.json --wait --json
```

### `cademi content products lessons taxonomy-terms` — Manage content products lessons taxonomy-terms

Related: `cademi content products lessons content get`, `cademi content products modules`, `cademi content products exams`

#### `cademi content products lessons taxonomy-terms list <product_id> <lesson_id>`

List lesson taxonomy terms · `GET /api/v3/products/{product_id}/lessons/{lesson_id}/taxonomy-terms` · permission `lessons.read`

Returns the taxonomy terms assigned to the lesson.

Only lessons in products with the `blog` format support taxonomies. For products in other formats, the API returns `format_unsupported` instead of an empty list.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output

Legacy path: `cademi lessons taxonomy-terms list`

**Examples:**

```bash
# get
cademi content products lessons taxonomy-terms list prd_42 les_7 --json
```

#### `cademi content products lessons taxonomy-terms update <product_id> <lesson_id>`

Replace lesson taxonomy terms · `PUT /api/v3/products/{product_id}/lessons/{lesson_id}/taxonomy-terms` · permission `lessons.update`

Replaces the complete set of taxonomy terms assigned to the lesson with the terms in `term_ids`. Send an empty array to remove all terms. Every term must belong to a taxonomy of the same product.

Only lessons in products with the `blog` format support taxonomies. For products in other formats, the API returns `format_unsupported` and nothing is saved. Replicated lessons are read-only, and requests to change them return `replica_readonly`.

Send the lesson's current `ETag` in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `lesson_id` — Public ID of the lesson, prefixed with `les_`. Example: `les_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `term_ids` | array of string | yes |  |

Full schema: `cademi commands content products lessons taxonomy-terms update --schema --json`

Legacy path: `cademi lessons taxonomy-terms update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products lessons taxonomy-terms update prd_42 les_7 -f 'term_ids[]=<value>' --json

# full body from a file
cademi content products lessons taxonomy-terms update prd_42 les_7 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
