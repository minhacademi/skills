---
name: cademi-cli-support-questions
description: "Answer student questions about product content — `cademi support questions` (9 commands)"
metadata:
  cademi-cli: "0.2.2"
  cademi-api: "3.10.1"
---

# Support Questions Commands

> cademi 0.2.2, API 3.10.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi support` — Handle student comments, questions and support tickets.

The account-wide queue lists questions awaiting a reply by default.
Use --status answered for answered questions. Replies belong to a question.

**Related:** `cademi content products questions`

**Related settings:**
- `cademi settings support questions` — Controls lesson questions, publication moderation and the administrator inbox. (`cademi settings support questions get|update`)

**See also in this domain:** `references/support.md`, `references/support-comments.md`, `references/support-faqs.md`, `references/support-tickets.md`

Live catalog for this file: `cademi commands support questions --json` (offline, no credential needed).

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

### `cademi support questions delete <product_id> <question_id>`

Delete a question · `DELETE /api/v3/products/{product_id}/questions/{question_id}` · permission `questions.delete`

Deletes the question together with all of its replies. The question and its replies are no longer returned by the API.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `question_id` — Public ID of the question, prefixed with `qst_`. Example: `qst_9`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi questions delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi support questions delete prd_42 qst_9 --yes
```

### `cademi support questions get <product_id> <question_id>`

Retrieve a question · `GET /api/v3/products/{product_id}/questions/{question_id}` · permission `questions.read`

Retrieves a question by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the question to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `question_id` — Public ID of the question, prefixed with `qst_`. Example: `qst_9`

**Flag sets:** output

Legacy path: `cademi questions get`

**Examples:**

```bash
# get
cademi support questions get prd_42 qst_9 --json
```

### `cademi support questions list`

List questions across all products · `GET /api/v3/support/questions` · permission `questions.read`

Returns the support queue of questions across all products in the account that are accessible with the current credentials. Results are paginated with a cursor.

By default, only questions awaiting a reply (`status=open`) are returned. Use `status=answered` to list answered questions instead.

This collection never includes reply drafts or the `has_draft` field, even when the credentials have the `questions.read_private` permission.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only questions with this status. (open, answered)

Legacy path: `cademi questions list`

**Examples:**

```bash
# list: one page, machine-readable
cademi support questions list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi support questions list --limit 200 --raw --json

# every page, projected
cademi support questions list --all --jq '[.[] | {id}]'
```

### `cademi support questions update <product_id> <question_id>`

Update a question · `PATCH /api/v3/products/{product_id}/questions/{question_id}` · permission `questions.update`, `questions.reopen (if field:status)`, `questions.pin (if field:pinned)`

Reopens a question or changes whether it is pinned. The question text belongs to the user and cannot be changed.

Setting `status` to `open` returns the question to the queue of questions awaiting a reply and requires the `questions.reopen` permission. Any published reply remains visible. Changing `pinned` requires the `questions.pin` permission.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `question_id` — Public ID of the question, prefixed with `qst_`. Example: `qst_9`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `pinned` | boolean |  |  |
| `status` | string |  | One of: `open` |

Full schema: `cademi commands support questions update --schema --json`

Legacy path: `cademi questions update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi support questions update prd_42 qst_9 -F pinned=true --if-match '"<etag>"' --json

# full body from a file
cademi support questions update prd_42 qst_9 --data @body.json --json
```

### `cademi support questions replies` — Manage support questions replies

Related: `cademi content products questions`

Related settings:
- `cademi settings support questions` — Controls lesson questions, publication moderation and the administrator inbox. (`cademi settings support questions get|update`)

#### `cademi support questions replies create <product_id> <question_id>`

Create a reply · `POST /api/v3/products/{product_id}/questions/{question_id}/replies` · permission `questions.update`

Publishes a support reply to a question or saves a draft reply. Requires an `Idempotency-Key` header.

With `status` set to `published` (the default), the reply is published with the requested `visibility` (`public` by default) and any saved draft is discarded. A question has a single support reply: if one already exists, its content is replaced. The user is notified only when the reply is first published, and only public replies generate events.

With `status` set to `draft`, the content is saved as the question's draft reply without publishing it, and `visibility` is ignored. The draft does not affect any reply already visible to the user.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `question_id` — Public ID of the question, prefixed with `qst_`. Example: `qst_9`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `body` | object |  |  |
| `body.blocks` | array of object |  |  |
| `status` | string |  | One of: `published`, `draft` |
| `visibility` | string |  | One of: `public`, `private` |

Full schema: `cademi commands support questions replies create --schema --json`

Legacy path: `cademi questions replies create`

**Examples:**

```bash
# partial update
cademi support questions replies create prd_42 qst_9 -f status=published --json

# full body from a file
cademi support questions replies create prd_42 qst_9 --data @body.json --json
```

#### `cademi support questions replies delete <product_id> <question_id> <reply_id>`

Delete a reply · `DELETE /api/v3/products/{product_id}/questions/{question_id}/replies/{reply_id}` · permission `questions.update`

Deletes a published support reply or discards the question's draft reply.

To discard the draft, use the reply ID `qrp_draft_<question_id>`. Deleting a published reply removes it from the user's view, and the question returns to the `open` status when no other support reply remains.

Only support replies can be deleted; replies written by users are rejected with `403`, the `permission_denied` error code, and `details[].field` set to `author.kind`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `question_id` — Public ID of the question, prefixed with `qst_`. Example: `qst_9`
- `reply_id` — Public ID of the reply, prefixed with `qrp_`. Example: `qrp_1`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi questions replies delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi support questions replies delete prd_42 qst_9 qrp_1 --yes
```

#### `cademi support questions replies get <product_id> <question_id> <reply_id>`

Retrieve a reply · `GET /api/v3/products/{product_id}/questions/{question_id}/replies/{reply_id}` · permission `questions.read`

Retrieves a published reply by its public ID, or the question's draft reply using the ID `qrp_draft_<question_id>`.

Private replies and drafts are visible only to credentials with the `questions.read_private` permission. Without it, they return `404 Not Found`.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the reply to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `question_id` — Public ID of the question, prefixed with `qst_`. Example: `qst_9`
- `reply_id` — Public ID of the reply, prefixed with `qrp_`. Example: `qrp_1`

**Flag sets:** output

Legacy path: `cademi questions replies get`

**Examples:**

```bash
# get
cademi support questions replies get prd_42 qst_9 qrp_1 --json
```

#### `cademi support questions replies list <product_id> <question_id>`

List replies to a question · `GET /api/v3/products/{product_id}/questions/{question_id}/replies` · permission `questions.read`

Returns all replies to a question in a single response, in chronological order, while preserving the standard collection response format.

Private replies and the draft reply (ID `qrp_draft_<question_id>`) are included only when the credentials have the `questions.read_private` permission. When present, the draft is listed last.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `question_id` — Public ID of the question, prefixed with `qst_`. Example: `qst_9`

**Flag sets:** output

Legacy path: `cademi questions replies list`

**Examples:**

```bash
# get
cademi support questions replies list prd_42 qst_9 --json
```

#### `cademi support questions replies update <product_id> <question_id> <reply_id>`

Update a reply · `PATCH /api/v3/products/{product_id}/questions/{question_id}/replies/{reply_id}` · permission `questions.update`

Updates a published support reply or the question's draft reply (ID `qrp_draft_<question_id>`).

For a published reply, `body` corrects the content without notifying the user again, and `visibility` switches the reply between `public` and `private`. Only support replies can be updated; replies written by users are rejected with `403`, the `permission_denied` error code, and `details[].field` set to `author.kind`.

For the draft, `body` replaces the draft content. Setting `status` to `published` publishes the draft (using the supplied `body`, if any) with the supplied `visibility` (`public` by default), with the same effects as publishing through the create operation.

Send the `ETag` returned by the retrieve operation, for a published reply or for the draft, in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `question_id` — Public ID of the question, prefixed with `qst_`. Example: `qst_9`
- `reply_id` — Public ID of the reply, prefixed with `qrp_`. Example: `qrp_1`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `body` | object |  |  |
| `body.blocks` | array of object |  |  |
| `status` | string |  | One of: `published` |
| `visibility` | string |  | One of: `public`, `private` |

Full schema: `cademi commands support questions replies update --schema --json`

Legacy path: `cademi questions replies update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi support questions replies update prd_42 qst_9 qrp_1 -f status=published --if-match '"<etag>"' --json

# full body from a file
cademi support questions replies update prd_42 qst_9 qrp_1 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
