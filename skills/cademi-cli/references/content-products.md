---
name: cademi-cli-content-products
description: "Manage products, modules, lessons and release schedules — `cademi content products` (15 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Content Products Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi content` — Organize showcases, products and learning content.

Products belong to showcases and have a format: course, flix, blog, link or
alias. Explore modules, lessons, exams and access-schedules from this group.
publish-all publishes the product and all its content, including copies in
replicated accounts. To grant access, use a delivery and a user enrollment.

**Related:** `cademi content showcases`, `cademi sales deliveries`, `cademi users enrollments`

**See also in this domain:** `references/content-banners.md`, `references/content-products-access-schedules.md`, `references/content-products-exams.md`, `references/content-products-lessons.md`, `references/content-products-modules.md`, `references/content-products-taxonomies.md`, `references/content-showcases.md`

**Group examples:**

```bash
cademi content products list
cademi content products modules list prd_42
cademi content products lessons list prd_42 --module-id mod_7
```

Live catalog for this file: `cademi commands content products --json` (offline, no credential needed).

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

### `cademi content products create`

Create a product · `POST /api/v3/products` · permission `products.create`

Creates a product in the given showcase. New products are created with the `draft` status. `position` is 1-based within the showcase; a value greater than the number of products places the product at the end.

`external_id` must be unique among the account's products that are not in the trash; a duplicate value returns the `already_exists` error code.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `external_id` | string, nullable |  |  |
| `format` | string | yes | One of: `course`, `flix`, `blog`, `link`, `alias` |
| `name` | string | yes |  |
| `position` | integer |  |  |
| `showcase_id` | string | yes |  |

Full schema: `cademi commands content products create --schema --json`

Legacy path: `cademi products create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products create -f format=course -f name=<name> -f showcase_id=<showcase_id> --json

# full body from a file
cademi content products create --data @body.json --json
```

### `cademi content products delete <product_id>`

Delete a product · `DELETE /api/v3/products/{product_id}` · permission `products.delete`

Moves the product to the trash, together with its modules, its lessons, and its copies in replicated accounts. The product is no longer available to users in the student area until it is restored with the update operation by setting `deleted` to `false`. Users' access and reports related to the product are preserved.

Restoring the product does not restore its copies in replicated accounts.

Replicated products are read-only and cannot be deleted; these requests return the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi products delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi content products delete prd_42 --yes
```

### `cademi content products get <product_id>`

Retrieve a product · `GET /api/v3/products/{product_id}` · permission `products.read`

Retrieves a product by its public ID.

The `offer` object is included only when the credentials have the `products.read_offer` permission.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the product to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output

Legacy path: `cademi products get`

**Examples:**

```bash
# get
cademi content products get prd_42 --json
```

### `cademi content products list`

List products · `GET /api/v3/products` · permission `products.read`

Lists the products of the account. Results are paginated with a cursor and can be filtered by showcase, status, external ID, search term, and deletion state.

If the credentials are limited to specific products, only those products are returned.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `--external-id <string>` — Only products with this external ID.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--q <string>` — Text search term.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--showcase-id <string>` — Only products in the showcase with this public ID.
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only products with this status. (published, draft, unlisted, hidden)

Legacy path: `cademi products list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products list --limit 200 --raw --json

# every page, projected
cademi content products list --all --jq '[.[] | {id}]'
```

### `cademi content products publish-all <product_id>`

Publish a product · `POST /api/v3/products/{product_id}/publications` · permission `products.publish`

Starts an asynchronous publication of the product and all of its content.

Publishing also publishes the product's copies in replicated accounts. Replicated products can be published.

The response returns an operation. Track it through the URL in the `Location` header.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, idempotency, async

Legacy path: `cademi products publications create`

**Examples:**

```bash
# run
cademi content products publish-all prd_42 --wait --json
```

### `cademi content products update <product_id>`

Update a product · `PATCH /api/v3/products/{product_id}` · permission `products.update`, `products.publish (if field:status)`, `showcases.update (if field:showcase_id)`, `showcases.update (if field:position)`

Updates a product. Only the fields included in the request are changed. Setting `deleted` to `false` restores the product from the trash.

Changing `status` also requires the `products.publish` permission. Changing `showcase_id` or `position` also requires the `showcases.update` permission on the target showcase.

`position` is 1-based within the showcase, in the destination showcase when `showcase_id` changes; a value greater than the number of products places the product at the end. A product moved to another showcase without `position` is placed at the end.

Replicated products accept only `status`; any other field returns the `replica_readonly` error code.

Send the current `ETag` value in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `definitions` | object |  |  |
| `deleted` | boolean |  | Only `false` is accepted. Restores it from the trash; deleting is the DELETE operation. One of: `false` |
| `display` | object |  |  |
| `external_id` | string, nullable |  |  |
| `image` | string, nullable |  |  |
| `name` | string |  |  |
| `position` | integer |  |  |
| `showcase_id` | string |  |  |
| `status` | string |  | One of: `published`, `draft`, `unlisted`, `hidden` |

Full schema: `cademi commands content products update --schema --json`

Legacy path: `cademi products update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi content products update prd_42 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi content products update prd_42 --data @body.json --json
```

### `cademi content products certificate template` — Manage content products certificate template

Related: `cademi users certificates`

#### `cademi content products certificate template get <product_id>`

Retrieve a certificate template · `GET /api/v3/products/{product_id}/certificate-template` · permission `certificates.read`

The certificate template has no ID of its own and is addressed through its product.

The response includes an `ETag` representing the current revision of the product. Send this value in the `If-Match` header when replacing the template to avoid overwriting a newer version. Because the revision covers the whole product, other changes to the product also produce a new `ETag`.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output

Legacy path: `cademi certificates template get`

**Examples:**

```bash
# get
cademi content products certificate template get prd_42 --json
```

#### `cademi content products certificate template update <product_id>`

Replace a certificate template · `PUT /api/v3/products/{product_id}/certificate-template` · permission `certificates.update`

Replaces the product's certificate template. Fields omitted from the request body are reset to their default values.

The certificate criteria, QR code position, and font must use one of the supported values.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version. Templates of replicated products are read-only.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `criteria` | object | yes | Condition under which the certificate becomes available to the user. |
| `criteria.exam_id` | string, nullable |  | Public ID of the exam, prefixed with `exm_`. Required when `kind` is `exam`. |
| `criteria.kind` | string | yes | One of: `completion`, `one_week_after_start`, `two_weeks_after_start`, `three_weeks_after_start`, `one_month_after_start`, `two_months_after_start`, `progress_percent`, `exam` |
| `criteria.percent` | integer, nullable |  | Required when `kind` is `progress_percent`. One of `75`, `80`, `85`, `90`, or `95`. |
| `enabled` | boolean | yes | Whether the product issues certificates. |
| `fields` | array of string |  | User custom fields printed on the certificate, in display order, as public IDs prefixed with `cfd_`. |
| `layout` | object |  |  |
| `layout.back_side_enabled` | boolean |  |  |
| `layout.background` | string, nullable |  | Background image of the certificate. When empty, the default design is used. |
| `layout.fields` | object |  | Text fields printed on the certificate, each with its own font and color. |
| `layout.fields.date` | object |  | A text field printed on the certificate. Omitted values are reset to their defaults. |
| `layout.fields.document` | object |  | A text field printed on the certificate. Omitted values are reset to their defaults. |
| `layout.fields.name` | object |  | A text field printed on the certificate. Omitted values are reset to their defaults. |
| `layout.fields.sequence` | object |  | A text field printed on the certificate. Omitted values are reset to their defaults. |
| `layout.qr` | object |  |  |
| `layout.qr.background_color` | string |  | Hexadecimal color in the `#RRGGBB` format. |
| `layout.qr.color` | string |  | Hexadecimal color in the `#RRGGBB` format. |
| `layout.qr.enabled` | boolean |  |  |
| `layout.qr.position` | string |  | One of: `bottom-right`, `bottom-left`, `top-right`, `top-left` |
| `layout.show_dates` | boolean |  |  |
| `layout.show_product_name` | boolean |  |  |
| `single_issue` | boolean |  | Whether each user can receive at most one certificate for the product. |
| `texts` | object |  |  |
| `texts.content` | string, nullable |  | Body text of the certificate. |
| `texts.instructor` | string, nullable |  |  |
| `texts.workload` | string, nullable |  |  |

Full schema: `cademi commands content products certificate template update --schema --json`

Legacy path: `cademi certificates template update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products certificate template update prd_42 -F 'criteria={...}' -f criteria.kind=completion -F enabled=true --json

# full body from a file
cademi content products certificate template update prd_42 --data @body.json --json
```

### `cademi content products certificate template previews` — Preview a certificate template

Related: `cademi users certificates`

#### `cademi content products certificate template previews create <product_id>`

Preview a certificate template · `POST /api/v3/products/{product_id}/certificate-template/previews` · permission `certificates.preview`

Renders a sample certificate from the product's current certificate template, using placeholder data.

A preview is not an issued certificate: it does not appear in certificate listings or reports and cannot be verified through public validation. The response does not include a `file_id`; use `pdf_url` to access the rendered PDF.

Templates of replicated products cannot be previewed.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, idempotency

Legacy path: `cademi certificates template previews create`

**Examples:**

```bash
# run
cademi content products certificate template previews create prd_42 --json
```

### `cademi content products comments` — List product comments

Related: `cademi support comments`

Related settings:
- `cademi settings support comments` — Controls lesson comments, publication moderation, spam limits and the administrator inbox. (`cademi settings support comments get|update`)

#### `cademi content products comments list <product_id>`

List product comments · `GET /api/v3/products/{product_id}/comments` · permission `comments.read`

Lists the top-level comments posted on a product's lessons. Replies are available through the list comment replies operation.

The collection can be filtered by lesson, user, status, and a text search term. Results are paginated with a cursor and sorted by ID, newest first by default; use `sort=id` for oldest first.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--lesson-id <string>` — Only comments on the lesson with this public ID.
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--q <string>` — Text search term.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only comments with this moderation status. (pending, approved, hidden)
- `--user-id <string>` — Only comments written by the user with this public ID.

Legacy path: `cademi products comments list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products comments list prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products comments list prd_42 --limit 200 --raw --json

# every page, projected
cademi content products comments list prd_42 --all --jq '[.[] | {id}]'
```

### `cademi content products content` — Inspect and order the product's top-level content

Lists root modules and lessons outside modules in display order. This is
one level of the tree. Use modules content list to inspect a module.

Related: `cademi content products modules content`

#### `cademi content products content list <product_id>`

List product content · `GET /api/v3/products/{product_id}/content` · permission `products.read`

Lists the top-level content of a product: its root modules and the lessons that do not belong to any module, in their display order, using cursor-based pagination.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Opaque continuation token from page.next_cursor of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of entries per page; minimum: 1; maximum: 200; API default: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--status <string>` — Only content with this status. (published, draft)

Legacy path: `cademi products content list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products content list prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products content list prd_42 --limit 200 --raw --json

# every page, projected
cademi content products content list prd_42 --all --jq '[.[] | {id}]'
```

### `cademi content products content order` — Reorder product content

Related: `cademi content products content list`, `cademi content products get`, `cademi guide reordering`

#### `cademi content products content order update <product_id>`

Reorder product content · `PUT /api/v3/products/{product_id}/content/order` · permission `products.update`

Sets the display order of the product's top-level content.

The `ids` list may mix module IDs (`mod_`) and lesson IDs (`les_`), and must contain exactly the complete set of root modules and lessons that do not belong to any module. If the supplied set does not match, the request is rejected.

Send the product's current `ETag` value in the `If-Match` header to avoid overwriting a newer version.

Discover the required set through GET /products/{product_id}/content, following every cursor page without narrowing filters such as status, kind or search. A fully paginated list is still limited by credential visibility; verify that the credential can read the entire required set before reordering. On order_set_mismatch, re-read the set and check scope and visibility instead of blindly retrying stale IDs.

_Full description: `cademi commands content products content order update --json --jq '.commands[0].description'`_

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `ids` | array of string | yes |  |

Full schema: `cademi commands content products content order update --schema --json`

Legacy path: `cademi products content order update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products content order update prd_42 -f 'ids[]=<value>' --json

# full body from a file
cademi content products content order update prd_42 --data @body.json --json
```

### `cademi content products copies` — Copy a product

Related: `cademi content showcases`, `cademi sales deliveries`, `cademi users enrollments`

#### `cademi content products copies create <product_id>`

Copy a product · `POST /api/v3/products/{product_id}/copies` · permission `products.create`

Starts an asynchronous copy of the product, with the given `name`, into the target showcase in the same account.

The response returns an operation. Track it through the URL in the `Location` header; when the operation completes, its result includes the `copy_id` of the new product.

Replicated products cannot be copied and replicated showcases cannot receive copies; these requests return the `replica_readonly` error code.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes |  |
| `showcase_id` | string | yes |  |

Full schema: `cademi commands content products copies create --schema --json`

Legacy path: `cademi products copies create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products copies create prd_42 -f name=<name> -f showcase_id=<showcase_id> --wait --json

# full body from a file
cademi content products copies create prd_42 --data @body.json --wait --json
```

### `cademi content products order` — Reorder showcase products

Related: `cademi content showcases products list`, `cademi content showcases get`, `cademi guide reordering`

#### `cademi content products order update <showcase_id>`

Reorder showcase products · `PUT /api/v3/showcases/{showcase_id}/products/order` · permission `products.update`, `showcases.update`

Sets the display order of the products in a showcase. Requires both the `products.update` and `showcases.update` permissions.

The request must contain the complete set of product IDs currently in the showcase. If the supplied set does not match, the request is rejected with `order_set_mismatch` and the existing order remains unchanged. This operation does not move products between showcases; to change a product's showcase, update the product's `showcase_id`.

Send the showcase's current `ETag` value in the `If-Match` header to avoid overwriting a newer version.

_Full description: `cademi commands content products order update --json --jq '.commands[0].description'`_

**Arguments:**
- `showcase_id` — Public ID of the showcase, prefixed with `shw_`. Example: `shw_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `ids` | array of string | yes |  |

Full schema: `cademi commands content products order update --schema --json`

Legacy path: `cademi products order update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi content products order update shw_42 -f 'ids[]=<value>' --json

# full body from a file
cademi content products order update shw_42 --data @body.json --json
```

### `cademi content products questions` — List questions for a product

Related: `cademi support questions`

Related settings:
- `cademi settings support questions` — Controls lesson questions, publication moderation and the administrator inbox. (`cademi settings support questions get|update`)

#### `cademi content products questions list <product_id>`

List questions for a product · `GET /api/v3/products/{product_id}/questions` · permission `questions.read`

Returns the questions users have asked in the lessons of a product. Results are paginated with a cursor and can be filtered by lesson, user, answered state, and search text.

The `has_draft` field is included only when the credentials also have the `questions.read_private` permission.

**Arguments:**
- `product_id` — Public ID of the product, prefixed with `prd_`. Example: `prd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--answered <string>` — When 'true', only answered questions; when 'false', only questions without an answer. (true, false)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--lesson-id <string>` — Only questions asked in the lesson with this public ID.
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--q <string>` — Text search term.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--user-id <string>` — Only questions asked by the user with this public ID.

Legacy path: `cademi products questions list`

**Examples:**

```bash
# list: one page, machine-readable
cademi content products questions list prd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi content products questions list prd_42 --limit 200 --raw --json

# every page, projected
cademi content products questions list prd_42 --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
