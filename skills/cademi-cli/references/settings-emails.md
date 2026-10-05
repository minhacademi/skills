---
name: cademi-cli-settings-emails
description: "Email settings — `cademi settings emails` (7 commands)"
metadata:
  cademi-cli: "0.2.1"
  cademi-api: "3.10.0"
---

# Settings Emails Commands

> cademi 0.2.1, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi settings` — Configure platform behavior, appearance and shared definitions.

Controls transactional email sender identity, signature and appearance.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

**See also in this domain:** `references/settings-access.md`, `references/settings-definitions.md`, `references/settings-platform.md`, `references/settings-legal-terms.md`, `references/settings-menus.md`, `references/settings-support.md`

Live catalog for this file: `cademi commands settings emails --json` (offline, no credential needed).

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

**async**
- `--wait` — When the API answers 202, wait for the operation to finish
- `--wait-timeout <duration>` — Maximum time for --wait, e.g. 30s or 2m (default: no limit; does not cancel the operation)

## Commands

### `cademi settings emails get`

Retrieve email settings · `GET /api/v3/settings/emails` · permission `settings.read`

Returns the email settings of the account: sender, signature, reply-to address, email branding, and sending status, including the sending limit and the number of emails sent in the current period.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings emails get --json
```

### `cademi settings emails update`

Update email settings · `PATCH /api/v3/settings/emails` · permission `settings.update_emails`

Updates the email settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `branding` | object, nullable |  | Email appearance. Only supplied child fields change. |
| `branding.background_color` | string, nullable |  | Page background |
| `branding.card_color` | string, nullable |  | Content background |
| `branding.link_color` | string, nullable |  | Link |
| `branding.logo` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. |
| `branding.text_color` | string, nullable |  | Text |
| `sender` | object, nullable |  | Sender identity and signature. Only supplied child fields change. |
| `sender.address` | string, nullable |  | Address to which students reply when they receive an email from the platform. |
| `sender.name` | string, nullable |  | Name displayed as the sender in the emails sent. |
| `sender.signature` | string, nullable |  | Text displayed at the end of all emails sent. |

Full schema: `cademi commands settings emails update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings emails update -f branding.background_color=<branding.background_color> --if-match '"<etag>"' --json

# full body from a file
cademi settings emails update --data @body.json --json
```

### `cademi settings emails templates` — Transactional email template settings

Controls whether a transactional email is sent, its access credentials and its editable message body.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

Related: `cademi users access-emails`, `cademi users password-reset-emails`

#### `cademi settings emails templates get <email_template_id>`

Retrieve an email template · `GET /api/v3/settings/emails/templates/{email_template_id}` · permission `settings.read`

Retrieves an email template by its key. The template catalog is fixed and contains `sale`, `import`, `free`, and `new_access`.

The response includes an `ETag` representing the current revision of this template; changes to other templates or settings groups do not affect it. Send this value in the `If-Match` header when updating the template to avoid overwriting a newer version.

**Arguments:**
- `email_template_id` — Public ID of the email template. Example: `sale`

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings emails templates get sale --json
```

#### `cademi settings emails templates list`

List email templates · `GET /api/v3/settings/emails/templates` · permission `settings.read`

Returns the complete email template catalog in a single response. The catalog is fixed: templates cannot be created or deleted.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings emails templates list --json
```

#### `cademi settings emails templates update <email_template_id>`

Update an email template · `PATCH /api/v3/settings/emails/templates/{email_template_id}` · permission `settings.update_emails`

Updates an email template. Only the fields included in the request are changed.

`body` can be changed only on editable templates. Sending `body` for the `new_access` template is rejected as invalid data.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Arguments:**
- `email_template_id` — Public ID of the email template. Example: `sale`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `body` | string, nullable |  | Custom message body for sale, import and free templates. null restores the default. The new_access template does not accept a custom body. |
| `enabled` | boolean |  | Whether this transactional email is sent. |
| `include_credentials` | boolean |  | Whether the email includes automatic access credentials. |

Full schema: `cademi commands settings emails templates update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings emails templates update sale -f body=<body> --if-match '"<etag>"' --json

# full body from a file
cademi settings emails templates update sale --data @body.json --json
```

### `cademi settings emails templates previews` — Preview an email template

#### `cademi settings emails templates previews create <email_template_id>`

Preview an email template · `POST /api/v3/settings/emails/templates/{email_template_id}/previews` · permission `settings.read`

Renders an email template with an optional draft `body` and returns the resulting subject and HTML. Nothing is saved, so this operation can be used to review changes before updating the template.

Requires only the `settings.read` permission. The `new_access` template does not support previews.

**Arguments:**
- `email_template_id` — Public ID of the email template. Example: `sale`

**Flag sets:** output, body, idempotency

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `body` | string, nullable |  | Draft body applied only while rendering this preview; nothing is saved or sent. Omit to preview the saved body. Null or empty text previews the standard body. Available for `sale`, `import` and `free`; `new_access` does not support… |

Full schema: `cademi commands settings emails templates previews create --schema --json`

**Examples:**

```bash
# partial update
cademi settings emails templates previews create sale -f body=<body> --json

# full body from a file
cademi settings emails templates previews create sale --data @body.json --json
```

### `cademi settings emails templates test-messages` — Send a test email

#### `cademi settings emails templates test-messages create <email_template_id>`

Send a test email · `POST /api/v3/settings/emails/templates/{email_template_id}/test-messages` · permission `settings.send_test_email`

Sends a test email using the template. The recipient in `to` must be the email address of an active administrator of the account or the configured sender address (`sender.address`).

The email is sent asynchronously. The response contains the operation that tracks the delivery, and the `Location` header contains its URL.

**Arguments:**
- `email_template_id` — Public ID of the email template. Example: `sale`

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `to` | string | yes | Recipient of the test email. Must match an administrator email in this account whose administrator record has not been deleted, or the configured `sender.address`. Other addresses are rejected with `validation_failed`. |

Full schema: `cademi commands settings emails templates test-messages create --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings emails templates test-messages create sale -f to=<to> --wait --json

# full body from a file
cademi settings emails templates test-messages create sale --data @body.json --wait --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
