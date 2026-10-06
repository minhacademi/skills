---
name: cademi-cli-settings-menus
description: "Configure student-area navigation menus and items — `cademi settings menus` (8 commands)"
metadata:
  cademi-cli: "0.2.5"
  cademi-api: "3.12.0"
---

# Settings Menus Commands

> cademi 0.2.5, API 3.12.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi settings` — Configure platform behavior, appearance and shared definitions.

Inspect menu containers here and manage their links through items.
Navigation settings do not grant access to linked products.

**Related:** `cademi content products`

**See also in this domain:** `references/settings-access.md`, `references/settings-definitions.md`, `references/settings-platform.md`, `references/settings-emails.md`, `references/settings-legal-terms.md`, `references/settings-support.md`

Live catalog for this file: `cademi commands settings menus --json` (offline, no credential needed).

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

### `cademi settings menus get <menu_key>`

Retrieve a menu · `GET /api/v3/menus/{menu_key}` · permission `menu_items.read`

Retrieves a menu from the fixed set of menus provided by the platform.

The `sidebar_hidden` field reflects the current account's setting for hiding the sidebar menu.

**Arguments:**
- `menu_key` — Key of the menu. Example: `student-area`

**Flag sets:** output

Legacy path: `cademi menus get`

**Examples:**

```bash
# get
cademi settings menus get student-area --json
```

### `cademi settings menus list`

List menus · `GET /api/v3/menus` · permission `menu_items.read`

Returns the fixed set of menus provided by the platform, including the capabilities and supported item kinds of each menu. Menus cannot be created or deleted.

Reading menus requires the `menu_items.read` permission; there is no separate permission for menus.

**Flag sets:** output

Legacy path: `cademi menus list`

**Examples:**

```bash
# get
cademi settings menus list --json
```

### `cademi settings menus items` — Menu item changes

Updates a navigation item label, destination and browser target.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

#### `cademi settings menus items create <menu_key>`

Create a menu item · `POST /api/v3/menus/{menu_key}/items` · permission `menu_items.create`

Creates a menu item. New items are added to the end of the menu.

URL and embed destinations must use the HTTP or HTTPS scheme. Product and showcase destinations must reference a product or showcase that exists in the current account.

**Arguments:**
- `menu_key` — Key of the menu. Example: `student-area`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `destination` | object | yes |  |
| `kind` | string | yes | One of: `shortcut`, `product`, `showcase`, `url`, `embed` |
| `label` | string | yes |  |
| `target` | string |  | One of: `_self`, `_blank` |

Full schema: `cademi commands settings menus items create --schema --json`

Legacy path: `cademi menu-items create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings menus items create student-area -F 'destination={...}' -f kind=shortcut -f label=<label> --json

# full body from a file
cademi settings menus items create student-area --data @body.json --json
```

#### `cademi settings menus items delete <menu_key> <menu_item_id>`

Delete a menu item · `DELETE /api/v3/menus/{menu_key}/items/{menu_item_id}` · permission `menu_items.delete`

Deletes a menu item and removes it from the menu.

**Arguments:**
- `menu_key` — Key of the menu. Example: `student-area`
- `menu_item_id` — Public ID of the menu item, prefixed with `mnu_`. Example: `mnu_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi menu-items delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi settings menus items delete student-area mnu_42 --yes
```

#### `cademi settings menus items get <menu_key> <menu_item_id>`

Retrieve a menu item · `GET /api/v3/menus/{menu_key}/items/{menu_item_id}` · permission `menu_items.read`

Retrieves a menu item by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the menu item to avoid overwriting a newer version.

**Arguments:**
- `menu_key` — Key of the menu. Example: `student-area`
- `menu_item_id` — Public ID of the menu item, prefixed with `mnu_`. Example: `mnu_42`

**Flag sets:** output

Legacy path: `cademi menu-items get`

**Examples:**

```bash
# get
cademi settings menus items get student-area mnu_42 --json
```

#### `cademi settings menus items list <menu_key>`

List menu items · `GET /api/v3/menus/{menu_key}/items` · permission `menu_items.read`

Returns every item in the menu, in display order, in a single response while preserving the standard collection response format.

**Arguments:**
- `menu_key` — Key of the menu. Example: `student-area`

**Flag sets:** output

Legacy path: `cademi menu-items list`

**Examples:**

```bash
# get
cademi settings menus items list student-area --json
```

#### `cademi settings menus items update <menu_key> <menu_item_id>`

Update a menu item · `PATCH /api/v3/menus/{menu_key}/items/{menu_item_id}` · permission `menu_items.update`

Updates a menu item. Fields that are omitted keep their current values.

Items cannot be nested or moved to another menu.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting changes made since the item was retrieved.

**Arguments:**
- `menu_key` — Key of the menu. Example: `student-area`
- `menu_item_id` — Public ID of the menu item, prefixed with `mnu_`. Example: `mnu_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `destination` | object |  | Destination configuration. When omitted, the current destination is used. |
| `destination.embed` | string |  | HTTP or HTTPS URL embedded when kind=embed. |
| `destination.product_id` | string |  | Product public ID prefixed with prd_, used when kind=product. |
| `destination.shortcut` | string |  | Built-in navigation shortcut key used when kind=shortcut. |
| `destination.showcase_id` | string |  | Showcase public ID prefixed with shw_, used when kind=showcase. |
| `destination.url` | string |  | HTTP or HTTPS URL used when kind=url. |
| `kind` | string |  | Destination type. Supply a matching destination when changing the type. One of: `shortcut`, `product`, `showcase`, `url`, `embed` |
| `label` | string |  | Text displayed for the menu item. |
| `target` | string |  | Whether navigation opens in the same tab or a new tab. One of: `_self`, `_blank` |

Full schema: `cademi commands settings menus items update --schema --json`

Legacy path: `cademi menu-items update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings menus items update student-area mnu_42 -f destination.embed=<destination.embed> --if-match '"<etag>"' --json

# full body from a file
cademi settings menus items update student-area mnu_42 --data @body.json --json
```

### `cademi settings menus items order` — Reorder menu items

#### `cademi settings menus items order update <menu_key>`

Reorder menu items · `PUT /api/v3/menus/{menu_key}/items/order` · permission `menu_items.update`

Sets the display order of the menu items.

The `ids` array must contain the public ID of every item in the menu, in the desired order. Partial reordering is not supported. If the supplied set does not match, the API returns the `order_set_mismatch` error code and the existing order remains unchanged.

**Arguments:**
- `menu_key` — Key of the menu. Example: `student-area`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `ids` | array of string | yes |  |

Full schema: `cademi commands settings menus items order update --schema --json`

Legacy path: `cademi menu-items order update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings menus items order update student-area -f 'ids[]=<value>' --json

# full body from a file
cademi settings menus items order update student-area --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
