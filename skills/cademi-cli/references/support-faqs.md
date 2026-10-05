---
name: cademi-cli-support-faqs
description: "Maintain frequently asked questions — `cademi support faqs` (6 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Support Faqs Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi support` — Handle student comments, questions and support tickets.

Manage reusable answers and their display order in the student area.

**Related settings:**
- `cademi settings support faq` — Controls whether the FAQ page is available and the title and guidance shown above its answers. (`cademi settings support faq get|update`)

**See also in this domain:** `references/support.md`, `references/support-comments.md`, `references/support-questions.md`, `references/support-tickets.md`

Live catalog for this file: `cademi commands support faqs --json` (offline, no credential needed).

## Shared flag sets

Each command lists the sets it accepts. The flags of a set are:

**output**
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document)
- `--jq <string>` — Filter data (or --raw envelope); strings print unquoted, other values as JSON; overrides --output
- `--json` — Print JSON (same as --output json)
- `-o, --output <string>` — Output format: json, table or yaml (default: table for lists on a terminal, JSON otherwise)
- `--raw` — Print the full response envelope instead of data

**body**
- `-d, --data <string>` — JSON body: a literal, @file or @- for stdin
- `-f, --field <stringArray>` — Add a string field: key=value (nested: a.b=v, arrays: tags[]=v)
- `-F, --typed-field <stringArray>` — Add a typed field: key=true\|false\|null\|number\|JSON, or key=@file to read text; applied after all -f fields

**idempotency**
- `--idempotency-key <string>` — Reuse the key returned with a failed request to repeat it (default: automatic UUIDv7)

**confirm**
- `-y, --yes` — Do not ask for confirmation

## Commands

### `cademi support faqs create`

Create an FAQ entry · `POST /api/v3/support/faqs` · permission `faqs.create`, `faqs.publish (if field:status)`

Creates an FAQ entry with a question and its answer.

Entries are created as drafts unless `status` is set to `published`. Creating a published entry also requires the `faqs.publish` permission.

Groups are labels: a group exists as soon as an entry is assigned to it, so there is no separate operation to create one.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `answer` | object | yes |  |
| `answer.blocks` | array of object |  |  |
| `group` | string, nullable |  |  |
| `question` | string | yes |  |
| `status` | string |  | One of: `draft`, `published` |

Full schema: `cademi commands support faqs create --schema --json`

Legacy path: `cademi faqs create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi support faqs create -F 'answer={...}' -f question=<question> --json

# full body from a file
cademi support faqs create --data @body.json --json
```

### `cademi support faqs delete <faq_id>`

Delete an FAQ entry · `DELETE /api/v3/support/faqs/{faq_id}` · permission `faqs.delete`

Deletes an FAQ entry. Deleted entries no longer appear in the collection and can no longer be retrieved.

**Arguments:**
- `faq_id` — Public ID of the faq, prefixed with `faq_`. Example: `faq_9`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi faqs delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi support faqs delete faq_9 --yes
```

### `cademi support faqs get <faq_id>`

Retrieve an FAQ entry · `GET /api/v3/support/faqs/{faq_id}` · permission `faqs.read`

Retrieves an FAQ entry by its public ID, including the answer content in `answer.blocks`.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the entry to avoid overwriting a newer version.

**Arguments:**
- `faq_id` — Public ID of the faq, prefixed with `faq_`. Example: `faq_9`

**Flag sets:** output

Legacy path: `cademi faqs get`

**Examples:**

```bash
# get
cademi support faqs get faq_9 --json
```

### `cademi support faqs list`

List FAQ entries · `GET /api/v3/support/faqs` · permission `faqs.read`

Lists the FAQ entries of the current account. The collection can be filtered by group and status.

Entries are sorted by group name, with ungrouped entries last, and then by position within each group. All matching entries are returned in a single response.

Collection items do not include the answer. Use the retrieve operation to obtain it.

**Flag sets:** output

**Flags:**
- `--group <string>` — Only entries in the group with this name.
- `--status <string>` — Only entries with this status. (draft, published)

Legacy path: `cademi faqs list`

**Examples:**

```bash
# get
cademi support faqs list --json
```

### `cademi support faqs update <faq_id>`

Update an FAQ entry · `PATCH /api/v3/support/faqs/{faq_id}` · permission `faqs.update`, `faqs.publish (if field:status)`

Updates an FAQ entry. Only the fields present in the request are changed.

Setting `position` moves the entry within its group; positions start at 1. When `group` is changed in the same request, `position` applies within the new group. Changing `status` also requires the `faqs.publish` permission.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Arguments:**
- `faq_id` — Public ID of the faq, prefixed with `faq_`. Example: `faq_9`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `answer` | object |  |  |
| `group` | string, nullable |  |  |
| `position` | integer |  |  |
| `question` | string |  |  |
| `status` | string |  | One of: `draft`, `published` |

Full schema: `cademi commands support faqs update --schema --json`

Legacy path: `cademi faqs update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi support faqs update faq_9 -f group=<group> --if-match '"<etag>"' --json

# full body from a file
cademi support faqs update faq_9 --data @body.json --json
```

### `cademi support faqs order` — Reorder FAQ entries

Related settings:
- `cademi settings support faq` — Controls whether the FAQ page is available and the title and guidance shown above its answers. (`cademi settings support faq get|update`)

#### `cademi support faqs order update`

Reorder FAQ entries · `PUT /api/v3/support/faqs/order` · permission `faqs.update`

Replaces the order of the FAQ entries within a single group.

Identify the group in `group`; omit it or send `null` to reorder the entries without a group. `ids` must list the public IDs of every entry in that group exactly once, in the desired order.

If the supplied IDs do not match the entries in the group, the API returns `order_set_mismatch` and the existing order remains unchanged.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `group` | string, nullable |  |  |
| `ids` | array of string | yes |  |

Full schema: `cademi commands support faqs order update --schema --json`

Legacy path: `cademi faqs order update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi support faqs order update -f 'ids[]=<value>' --json

# full body from a file
cademi support faqs order update --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
