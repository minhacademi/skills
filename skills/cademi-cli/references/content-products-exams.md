---
name: cademi-cli-content-products-exams
description: "Manage a product's exams, questions and answer keys — `cademi content products exams` (19 commands)"
metadata:
  cademi-cli: "0.2.6"
  cademi-api: "3.12.1"
---

# Content Products Exams Commands

> cademi 0.2.6, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi content` — Organize showcases, products and learning content.

Exams belong to a product. Questions here are assessment questions.
Student support questions are under support questions. Reading an answer
key requires a separate permission from reading the exam.

**Related:** `cademi reports exams`

**See also in this domain:** `references/content-banners.md`, `references/content-products.md`, `references/content-products-access-schedules.md`, `references/content-products-lessons.md`, `references/content-products-modules.md`, `references/content-products-taxonomies.md`, `references/content-showcases.md`

Live catalog for this file: `cademi commands content products exams --json` (offline, no credential needed).

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

### `cademi content products exams create <product_id>`

Create an exam · `POST /api/v3/products/{product_id}/exams` · permission `exams.create`

Creates an exam in the product.

A new exam is not attached to any lesson. To display it to users, set its ID in the `exam_id` field of a lesson using the lesson update operation.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `description` | string, nullable |  |  |
| `settings` | object |  |  |
| `title` | string | yes |  |

Full schema: `cademi commands content products exams create --schema --json`

Legacy path: `cademi exams create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products exams create prd_42 -f title=<title> --json

# full body from a file
cademi content products exams create prd_42 --data @body.json --json
```

### `cademi content products exams delete <product_id> <exam_id>`

Delete an exam · `DELETE /api/v3/products/{product_id}/exams/{exam_id}` · permission `exams.delete`

Moves the exam and its questions to the trash. Recorded attempts do not prevent deletion.

An exam that is displayed by a lesson, required to unlock a certificate, or required by a release rule cannot be deleted. In that case the API returns `state_conflict`, and `details[].reason` indicates the dependency: `lesson`, `certificate`, or `release_rule`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`

**Flag sets:** output, idempotency, confirm

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

Legacy path: `cademi exams delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products exams delete prd_42 exm_7 --yes
```

### `cademi content products exams get <product_id> <exam_id>`

Retrieve an exam · `GET /api/v3/products/{product_id}/exams/{exam_id}` · permission `exams.read`

Retrieves an exam by its public ID. The response never includes the correct alternatives; use the answer key operation, which requires the `exams.read_answer_key` permission.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when modifying the exam to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`

**Flag sets:** output

Legacy path: `cademi exams get`

**Examples:**

```bash
# get
cademi content products exams get prd_42 exm_7 --json
```

### `cademi content products exams list <product_id>`

List exams · `GET /api/v3/products/{product_id}/exams` · permission `exams.read`

Returns the exams of the product. Requires the `exams.read` permission.

The collection never includes the correct alternatives; use the answer key operation, which requires the `exams.read_answer_key` permission.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--lesson-id <string>` — Public ID of a lesson. Returns only the exam displayed by that lesson.
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)

Legacy path: `cademi exams list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products exams list prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products exams list prd_42 --limit 200 --raw --json

# every page, projected
cademi content products exams list prd_42 --all --jq '[.[] | {id}]'
```

### `cademi content products exams update <product_id> <exam_id>`

Update an exam · `PATCH /api/v3/products/{product_id}/exams/{exam_id}` · permission `exams.update`

Updates an exam. `settings` is applied partially: omitted keys keep their current values, and `null` in a numeric setting removes the limit.

To avoid overwriting a newer version, send the `ETag` from the retrieve operation in the `If-Match` header.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | Only `false` is accepted. Restores the exam from the trash, together with the questions that were moved to the trash when the exam was deleted. One of: `false` |
| `description` | string, nullable |  |  |
| `settings` | object |  |  |
| `title` | string |  |  |

Full schema: `cademi commands content products exams update --schema --json`

Legacy path: `cademi exams update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products exams update prd_42 exm_7 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi content products exams update prd_42 exm_7 --data @body.json --json
```

### `cademi content products exams answer-key` — Retrieve an exam answer key

Related: `cademi reports exams`

#### `cademi content products exams answer-key get <product_id> <exam_id>`

Retrieve an exam answer key · `GET /api/v3/products/{product_id}/exams/{exam_id}/answer-key` · permission `exams.read_answer_key`

Returns the correct alternatives for the exam questions.

Requires the `exams.read_answer_key` permission; `exams.read` alone does not grant access to this operation.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`

**Flag sets:** output

Legacy path: `cademi exams answer-key get`

**Examples:**

```bash
# get
cademi content products exams answer-key get prd_42 exm_7 --json
```

### `cademi content products exams attempts` — Manage content products exams attempts

Related: `cademi reports exams`

#### `cademi content products exams attempts get <product_id> <exam_id> <exam_attempt_id>`

Retrieve an exam attempt · `GET /api/v3/products/{product_id}/exams/{exam_id}/attempts/{exam_attempt_id}` · permission `exams.attempts.read`

Retrieves an exam attempt by its public ID. The user who took the exam is identified by ID and name; the email address is not included.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when reopening the attempt to avoid acting on an outdated version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`
- `exam_attempt_id` — Public ID of the exam attempt, prefixed with `exa_`. Example: `exa_9`

**Flag sets:** output

Legacy path: `cademi exams attempts get`

**Examples:**

```bash
# get
cademi content products exams attempts get prd_42 exm_7 exa_9 --json
```

#### `cademi content products exams attempts list <product_id> <exam_id>`

List exam attempts · `GET /api/v3/products/{product_id}/exams/{exam_id}/attempts` · permission `exams.attempts.read`

Returns the attempts made on the exam. Requires the `exams.attempts.read` permission.

If the current credentials are restricted to specific users, only attempts made by those users are returned.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--finished-after <string>` — Only items finished after this date and time (ISO 8601).
- `--finished-before <string>` — Only items finished before this date and time (ISO 8601).
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only attempts with this status. (open, completed, discarded, timed_out)
- `--user-id <string>` — Only attempts made by the user with this public ID.

Legacy path: `cademi exams attempts list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products exams attempts list prd_42 exm_7 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products exams attempts list prd_42 exm_7 --limit 200 --raw --json

# every page, projected
cademi content products exams attempts list prd_42 exm_7 --all --jq '[.[] | {id}]'
```

#### `cademi content products exams attempts update <product_id> <exam_id> <exam_attempt_id>`

Reopen an exam attempt · `PATCH /api/v3/products/{product_id}/exams/{exam_id}/attempts/{exam_attempt_id}` · permission `exams.attempts.reopen`

Reopens an exam attempt by setting `status` to `open`. A `reason` is required and no other fields can be changed.

Reopening discards this attempt and any earlier attempts by the same user on this exam, marks the lesson that displays the exam as not completed, reverses the points awarded, and records the adjustment with its reason and author. Certificates that were already issued are not revoked.

Attempts that are already open or already discarded cannot be reopened.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`
- `exam_attempt_id` — Public ID of the exam attempt, prefixed with `exa_`. Example: `exa_9`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `reason` | string | yes |  |
| `status` | string | yes | One of: `open` |

Full schema: `cademi commands content products exams attempts update --schema --json`

Legacy path: `cademi exams attempts update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products exams attempts update prd_42 exm_7 exa_9 -f reason=<reason> -f status=open --json

# full body from a file
cademi content products exams attempts update prd_42 exm_7 exa_9 --data @body.json --json
```

### `cademi content products exams attempts answers` — Retrieve the answers for an exam attempt

Related: `cademi reports exams`

#### `cademi content products exams attempts answers get <product_id> <exam_id> <exam_attempt_id>`

Retrieve the answers for an exam attempt · `GET /api/v3/products/{product_id}/exams/{exam_id}/attempts/{exam_attempt_id}/answers` · permission `exams.attempts.read_answers`

Returns the answers the user submitted in the attempt.

Requires the `exams.attempts.read_answers` permission; `exams.attempts.read` alone does not grant access to this operation.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`
- `exam_attempt_id` — Public ID of the exam attempt, prefixed with `exa_`. Example: `exa_9`

**Flag sets:** output

Legacy path: `cademi exams attempts answers get`

**Examples:**

```bash
# get
cademi content products exams attempts answers get prd_42 exm_7 exa_9 --json
```

### `cademi content products exams questions` — Manage content products exams questions

Related: `cademi reports exams`

#### `cademi content products exams questions create <product_id> <exam_id>`

Create an exam question · `POST /api/v3/products/{product_id}/exams/{exam_id}/questions` · permission `exams.update`

Adds a question to the exam.

The question must have at least two alternatives, and at least one of them must be marked as correct. Questions are single-choice: if more than one alternative is marked as correct, only the first one is treated as correct.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `alternatives` | array of object | yes |  |
| `format` | string | yes | One of: `multiple_choice` |
| `image` | string, nullable |  |  |
| `text` | string | yes |  |

Full schema: `cademi commands content products exams questions create --schema --json`

Legacy path: `cademi exams questions create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products exams questions create prd_42 exm_7 --data '{"alternatives":[...]}' -f format=multiple_choice -f text=<text> --json

# full body from a file
cademi content products exams questions create prd_42 exm_7 --data @body.json --json
```

#### `cademi content products exams questions delete <product_id> <exam_id> <exam_question_id>`

Delete an exam question · `DELETE /api/v3/products/{product_id}/exams/{exam_id}/questions/{exam_question_id}` · permission `exams.update`

Moves the question to the trash. Attempts that already answered the question remain unchanged.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`
- `exam_question_id` — Public ID of the exam question, prefixed with `exq_`. Example: `exq_3`

**Flag sets:** output, idempotency, confirm

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

Legacy path: `cademi exams questions delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products exams questions delete prd_42 exm_7 exq_3 --yes
```

#### `cademi content products exams questions get <product_id> <exam_id> <exam_question_id>`

Retrieve an exam question · `GET /api/v3/products/{product_id}/exams/{exam_id}/questions/{exam_question_id}` · permission `exams.read`

The `correct` field of each alternative is included only when the current credentials have the `exams.read_answer_key` permission; otherwise, the field is omitted.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`
- `exam_question_id` — Public ID of the exam question, prefixed with `exq_`. Example: `exq_3`

**Flag sets:** output

Legacy path: `cademi exams questions get`

**Examples:**

```bash
# get
cademi content products exams questions get prd_42 exm_7 exq_3 --json
```

#### `cademi content products exams questions list <product_id> <exam_id>`

List exam questions · `GET /api/v3/products/{product_id}/exams/{exam_id}/questions` · permission `exams.read`

The `correct` field of each alternative is included only when the current credentials have the `exams.read_answer_key` permission; otherwise, the field is omitted.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200; API default: 50
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)

Legacy path: `cademi exams questions list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products exams questions list prd_42 exm_7 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products exams questions list prd_42 exm_7 --limit 200 --raw --json

# every page, projected
cademi content products exams questions list prd_42 exm_7 --all --jq '[.[] | {id}]'
```

#### `cademi content products exams questions update <product_id> <exam_id> <exam_question_id>`

Update an exam question · `PATCH /api/v3/products/{product_id}/exams/{exam_id}/questions/{exam_question_id}` · permission `exams.update`

Updates an exam question. When `alternatives` is supplied, it replaces the entire set of alternatives. Answers already recorded in completed attempts are not regraded.

To avoid overwriting a newer version, send the `ETag` from the retrieve operation in the `If-Match` header.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`
- `exam_question_id` — Public ID of the exam question, prefixed with `exq_`. Example: `exq_3`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `alternatives` | array of object |  |  |
| `deleted` | boolean |  | Only `false` is accepted. Restores the question from the trash. One of: `false` |
| `image` | string, nullable |  |  |
| `text` | string |  |  |

Full schema: `cademi commands content products exams questions update --schema --json`

Legacy path: `cademi exams questions update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products exams questions update prd_42 exm_7 exq_3 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi content products exams questions update prd_42 exm_7 exq_3 --data @body.json --json
```

### `cademi content products exams questions order` — Reorder exam questions

Related: `cademi content products exams questions list`, `cademi content products exams get`, `cademi guide reordering`

#### `cademi content products exams questions order update <product_id> <exam_id>`

Reorder exam questions · `PUT /api/v3/products/{product_id}/exams/{exam_id}/questions/order` · permission `exams.update`

Sets the order in which the exam questions are presented.

The request must contain exactly the set of questions currently in the exam, excluding questions in the trash. If the supplied set does not match, the request is rejected with `order_set_mismatch` and the existing order remains unchanged.

To avoid overwriting a newer version, send the exam `ETag` from the exam retrieve operation in the `If-Match` header.

Discover the required set through GET /products/{product_id}/exams/{exam_id}/questions, following every cursor page without narrowing filters such as status, kind or search. A fully paginated list is still limited by credential visibility; verify that the credential can read the entire required set before reordering. On order_set_mismatch, re-read the set and check scope and visibility instead of blindly retrying stale IDs.

_Full description: `cademi commands content products exams questions order update --json --jq '.commands[0].description'`_

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `ids` | array of string | yes |  |

Full schema: `cademi commands content products exams questions order update --schema --json`

Legacy path: `cademi exams questions order update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products exams questions order update prd_42 exm_7 -f 'ids[]=<value>' --json

# full body from a file
cademi content products exams questions order update prd_42 exm_7 --data @body.json --json
```

### `cademi content products exams results` — Manage content products exams results

Related: `cademi reports exams`

#### `cademi content products exams results delete <product_id> <exam_id> <exam_result_id>`

Delete an exam result · `DELETE /api/v3/products/{product_id}/exams/{exam_id}/results/{exam_result_id}` · permission `exams.results.delete`

Permanently deletes an exam result. A `reason` is required in the request body, and the deletion is recorded with its reason and author.

The lesson that displays the exam is marked as not completed, the points awarded are reversed, and the user's certificate eligibility is recalculated. Certificates that were already issued are not revoked.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`
- `exam_result_id` — Public ID of the exam result, prefixed with `exa_`. Example: `exa_9`

**Flag sets:** output, body, idempotency, confirm

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `reason` | string | yes |  |

Full schema: `cademi commands content products exams results delete --schema --json`

Legacy path: `cademi exams results delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products exams results delete prd_42 exm_7 exa_9 --yes
```

#### `cademi content products exams results get <product_id> <exam_id> <exam_result_id>`

Retrieve an exam result · `GET /api/v3/products/{product_id}/exams/{exam_id}/results/{exam_result_id}` · permission `exams.attempts.read`

A result shares the public ID of the attempt it comes from. Attempts that are still open or were discarded are not results.

The response includes an `ETag` representing the current revision, which can be sent in the `If-Match` header when deleting the result.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`
- `exam_result_id` — Public ID of the exam result, prefixed with `exa_`. Example: `exa_9`

**Flag sets:** output

Legacy path: `cademi exams results get`

**Examples:**

```bash
# get
cademi content products exams results get prd_42 exm_7 exa_9 --json
```

#### `cademi content products exams results list <product_id> <exam_id>`

List exam results · `GET /api/v3/products/{product_id}/exams/{exam_id}/results` · permission `exams.attempts.read`

Returns the results of the exam. A result is an attempt that was completed and not discarded; attempts that ended because the time limit expired are also results.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`
- `exam_id` — Public ID of the exam, prefixed with `exm_`. Example: `exm_7`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--finished-after <string>` — Only items finished after this date and time (ISO 8601).
- `--finished-before <string>` — Only items finished before this date and time (ISO 8601).
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--user-id <string>` — Only results of the user with this public ID.

Legacy path: `cademi exams results list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products exams results list prd_42 exm_7 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products exams results list prd_42 exm_7 --limit 200 --raw --json

# every page, projected
cademi content products exams results list prd_42 exm_7 --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
