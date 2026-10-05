---
name: cademi-cli-settings-support
description: "Support ticket settings — `cademi settings support` (8 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Settings Support Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi settings` — Configure platform behavior, appearance and shared definitions.

Controls availability of the student support channel and how administrators open conversations.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

**Related:** `cademi support tickets`, `cademi support departments`

**See also in this domain:** `references/settings.md`, `references/settings-emails.md`, `references/settings-legal-terms.md`, `references/settings-menus.md`

Live catalog for this file: `cademi commands settings support --json` (offline, no credential needed).

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

## Commands

### `cademi settings support get`

Retrieve support settings · `GET /api/v3/settings/support` · permission `settings.read`

Returns the settings of the support ticket channel in the student area.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings support get --json
```

### `cademi settings support update`

Update support settings · `PATCH /api/v3/settings/support` · permission `settings.update_support`

Updates the support ticket channel settings. Only the fields included in the request are changed; omitted fields are left unchanged.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `enabled` | boolean |  | Enable or disable the support channel for students. |
| `open_in_new_tab` | boolean |  | Clicking an item in the queue opens the conversation in a new browser tab, and the queue stays where it is. |

Full schema: `cademi commands settings support update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings support update -F enabled=true --if-match '"<etag>"' --json

# full body from a file
cademi settings support update --data @body.json --json
```

### `cademi settings support comments` — Comment settings

Controls lesson comments, publication moderation, spam limits and the administrator inbox.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

Related: `cademi support comments`, `cademi content products comments`

#### `cademi settings support comments get`

Retrieve comment settings · `GET /api/v3/settings/support/comments` · permission `settings.read`

Returns the lesson comment settings of the account: availability, moderation, tags, texts displayed to users, sort order, and posting limits.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings support comments get --json
```

#### `cademi settings support comments update`

Update comment settings · `PATCH /api/v3/settings/support/comments` · permission `settings.update_support`

Updates the lesson comment settings. Only the fields included in the request are changed; omitted fields are left unchanged.

Setting `moderation_enabled` to `false` publishes every comment in the account that is pending moderation. The number of comments published is returned in `effects.approved_pending_comments`.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `cooldown_minutes` | integer |  | Waiting period in minutes; 0 disables the waiting period. Define how long students must wait before sending another comment. |
| `enabled` | boolean |  | Enable or disable comments on lessons. |
| `min_length` | integer |  | Minimum number of characters required in each comment. |
| `moderation_enabled` | boolean |  | When false, publishes every pending comment in the account. The response reports the number in effects.approved_pending_comments. |
| `open_in_new_tab` | boolean |  | Clicking an item in the queue opens the conversation in a new browser tab, and the queue stays where it is. |
| `placeholder` | string |  | Shown inside the field where students write a comment. |
| `sort` | string |  | Sets the order comments arrive in your dashboard inbox. One of: `newest`, `oldest` |
| `tags_enabled` | boolean |  | Whether student tags are displayed to administrators in the comments inbox. |
| `title` | string |  | Shown in the student area menu. |

Full schema: `cademi commands settings support comments update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings support comments update -F cooldown_minutes=1 --if-match '"<etag>"' --json

# full body from a file
cademi settings support comments update --data @body.json --json
```

### `cademi settings support faq` — FAQ settings

Controls whether the FAQ page is available and the title and guidance shown above its answers.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

Related: `cademi support faqs`

#### `cademi settings support faq get`

Retrieve FAQ settings · `GET /api/v3/settings/support/faq` · permission `settings.read`

Returns the settings of the FAQ section in the student area: availability, title, and text.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings support faq get --json
```

#### `cademi settings support faq update`

Update FAQ settings · `PATCH /api/v3/settings/support/faq` · permission `settings.update_support`

Updates the FAQ section settings. Only the fields included in the request are changed; omitted fields are left unchanged.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `enabled` | boolean |  | Enable or disable the frequently asked questions page. |
| `text` | string |  | Shown above the frequently asked questions. |
| `title` | string |  | Shown at the top of the frequently asked questions page. |

Full schema: `cademi commands settings support faq update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings support faq update -F enabled=true --if-match '"<etag>"' --json

# full body from a file
cademi settings support faq update --data @body.json --json
```

### `cademi settings support questions` — Student question settings

Controls lesson questions, publication moderation and the administrator inbox.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

Related: `cademi support questions`, `cademi content products questions`

#### `cademi settings support questions get`

Retrieve question settings · `GET /api/v3/settings/support/questions` · permission `settings.read`

Returns the lesson question settings of the account: availability, moderation, texts displayed to users, sort order, and minimum length.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings support questions get --json
```

#### `cademi settings support questions update`

Update question settings · `PATCH /api/v3/settings/support/questions` · permission `settings.update_support`

Updates the lesson question settings. Only the fields included in the request are changed; omitted fields are left unchanged.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `enabled` | boolean |  | Enable or disable the question channel on lessons. |
| `min_length` | integer |  | Minimum number of characters required for each question. |
| `moderation_enabled` | boolean |  | When false, student-authored questions become visible in their lesson regardless of moderation status, including existing pending or hidden questions. This changes the visibility policy without rewriting message statuses. Viewers still… |
| `open_in_new_tab` | boolean |  | Clicking an item in the queue opens the conversation in a new browser tab, and the queue stays where it is. |
| `placeholder` | string |  | Shown inside the field where students write a question. |
| `sort` | string |  | Sets the order questions arrive in your dashboard inbox. One of: `newest`, `oldest` |
| `title` | string |  | Shown in the student area menu. |

Full schema: `cademi commands settings support questions update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings support questions update -F enabled=true --if-match '"<etag>"' --json

# full body from a file
cademi settings support questions update --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
