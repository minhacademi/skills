---
name: cademi-cli-support-comments
description: "Moderate comments and manage their replies — `cademi support comments` (8 commands)"
metadata:
  cademi-cli: "0.2.1"
  cademi-api: "3.10.0"
---

# Support Comments Commands

> cademi 0.2.1, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi support` — Handle student comments, questions and support tickets.

Inspect and moderate individual comments here. Use content products comments
to list a product's comments. Replies belong to their parent comment.

**Related:** `cademi content products comments`

**Related settings:**
- `cademi settings support comments` — Controls lesson comments, publication moderation, spam limits and the administrator inbox. (`cademi settings support comments get|update`)

**See also in this domain:** `references/support.md`, `references/support-faqs.md`, `references/support-questions.md`, `references/support-tickets.md`

Live catalog for this file: `cademi commands support comments --json` (offline, no credential needed).

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

### `cademi support comments delete <product_id> <comment_id>`

Delete a comment · `DELETE /api/v3/products/{product_id}/comments/{comment_id}` · permission `comments.delete`

Deletes a top-level comment together with all of its replies.

To remove a single reply while keeping the rest of the conversation, use the delete reply operation.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `comment_id` — Public ID of the comment, prefixed with `cmt_`. Example: `cmt_9`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi comments delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi support comments delete prd_42 cmt_9 --yes
```

### `cademi support comments get <product_id> <comment_id>`

Retrieve a comment · `GET /api/v3/products/{product_id}/comments/{comment_id}` · permission `comments.read`

Retrieves a top-level comment by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the comment to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `comment_id` — Public ID of the comment, prefixed with `cmt_`. Example: `cmt_9`

**Flag sets:** output

Legacy path: `cademi comments get`

**Examples:**

```bash
# get
cademi support comments get prd_42 cmt_9 --json
```

### `cademi support comments list`

List comments for moderation · `GET /api/v3/support/comments` · permission `comments.read`

Lists top-level comments across all products accessible with the current credentials, for use as a moderation queue.

When `status` is omitted, only comments with the `pending` status are returned. Results are paginated with a cursor and sorted by ID, newest first by default; use `sort=id` for oldest first.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only comments with this moderation status. (pending, approved, hidden)

Legacy path: `cademi comments list`

**Examples:**

```bash
# list: one page, machine-readable
cademi support comments list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi support comments list --limit 200 --raw --json

# every page, projected
cademi support comments list --all --jq '[.[] | {id}]'
```

### `cademi support comments update <product_id> <comment_id>`

Update a comment · `PATCH /api/v3/products/{product_id}/comments/{comment_id}` · permission `comments.update`, `comments.moderate (if field:status)`, `comments.pin (if field:pinned)`

Moderates or pins a top-level comment. Only `status` and `pinned` can be changed; the comment text cannot be edited.

Changing `status` requires the `comments.moderate` permission, and changing `pinned` requires the `comments.pin` permission.

The retrieve operation returns an `ETag` representing the current revision. Send this value in the optional `If-Match` header to avoid overwriting a newer version; if the comment has changed since that revision, the update is rejected.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `comment_id` — Public ID of the comment, prefixed with `cmt_`. Example: `cmt_9`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `pinned` | boolean |  |  |
| `status` | string |  | One of: `approved`, `hidden` |

Full schema: `cademi commands support comments update --schema --json`

Legacy path: `cademi comments update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi support comments update prd_42 cmt_9 -F pinned=true --if-match '"<etag>"' --json

# full body from a file
cademi support comments update prd_42 cmt_9 --data @body.json --json
```

### `cademi support comments replies` — Manage support comments replies

Related: `cademi content products comments`

Related settings:
- `cademi settings support comments` — Controls lesson comments, publication moderation, spam limits and the administrator inbox. (`cademi settings support comments get|update`)

#### `cademi support comments replies create <product_id> <comment_id>`

Create a comment reply · `POST /api/v3/products/{product_id}/comments/{comment_id}/replies` · permission `comments.update`

Adds a reply to a top-level comment.

Replies created through the API are attributed to the credential that made the request (`author.kind` is `credential`).

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `comment_id` — Public ID of the comment, prefixed with `cmt_`. Example: `cmt_9`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `text` | string | yes |  |
| `visibility` | string |  | One of: `public` |

Full schema: `cademi commands support comments replies create --schema --json`

Legacy path: `cademi comments replies create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi support comments replies create prd_42 cmt_9 -f text=<text> --json

# full body from a file
cademi support comments replies create prd_42 cmt_9 --data @body.json --json
```

#### `cademi support comments replies delete <product_id> <comment_id> <reply_id>`

Delete a comment reply · `DELETE /api/v3/products/{product_id}/comments/{comment_id}/replies/{reply_id}` · permission `comments.delete`

Deletes a single reply from a comment. The top-level comment and its other replies are not affected.

Only replies written by an administrator or an API credential (`author.kind` is `admin` or `credential`) can be deleted. Attempts to delete any other reply return `403` with the `permission_denied` error code and `details[].field` set to `author.kind`. To remove the entire conversation, delete the top-level comment instead.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `comment_id` — Public ID of the comment, prefixed with `cmt_`. Example: `cmt_9`
- `reply_id` — Public ID of the reply, prefixed with `crp_`. Example: `crp_1`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi comments replies delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi support comments replies delete prd_42 cmt_9 crp_1 --yes
```

#### `cademi support comments replies list <product_id> <comment_id>`

List comment replies · `GET /api/v3/products/{product_id}/comments/{comment_id}/replies` · permission `comments.read`

Lists the replies to a top-level comment in chronological order.

Returns all replies in a single response while preserving the standard collection response format.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `comment_id` — Public ID of the comment, prefixed with `cmt_`. Example: `cmt_9`

**Flag sets:** output

Legacy path: `cademi comments replies list`

**Examples:**

```bash
# get
cademi support comments replies list prd_42 cmt_9 --json
```

#### `cademi support comments replies update <product_id> <comment_id> <reply_id>`

Update a comment reply · `PATCH /api/v3/products/{product_id}/comments/{comment_id}/replies/{reply_id}` · permission `comments.update`, `comments.moderate (if field:status)`

Updates the text or the moderation status of a comment reply.

Only the text of replies written by an administrator or an API credential (`author.kind` is `admin` or `credential`) can be changed; attempts to edit the text of any other reply return `403` with the `permission_denied` error code and `details[].field` set to `author.kind`. Changing `status` requires the `comments.moderate` permission.

To avoid overwriting a newer version, send the optional `If-Match` header built from the reply's `revision` field, in the format `W/"comment_reply:<reply_id>:<revision>"`. If the reply has changed since that revision, the update is rejected.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `comment_id` — Public ID of the comment, prefixed with `cmt_`. Example: `cmt_9`
- `reply_id` — Public ID of the reply, prefixed with `crp_`. Example: `crp_1`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `status` | string |  | One of: `approved`, `hidden` |
| `text` | string |  |  |

Full schema: `cademi commands support comments replies update --schema --json`

Legacy path: `cademi comments replies update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi support comments replies update prd_42 cmt_9 crp_1 -f status=approved --if-match '"<etag>"' --json

# full body from a file
cademi support comments replies update prd_42 cmt_9 crp_1 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
