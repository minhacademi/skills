---
name: cademi-cli-settings
description: "Configure platform behavior, appearance and shared definitions — `cademi settings` (40 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Settings Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

Configure the student and administrator areas, authentication and features.
Tags and custom-fields define the catalog; user-specific values live under
users. Use config for declarative manifests and plan/apply workflows.

**Related:** `cademi config`, `cademi users tags`, `cademi users custom-fields`

**See also in this domain:** `references/settings-emails.md`, `references/settings-legal-terms.md`, `references/settings-menus.md`, `references/settings-support.md`

Live catalog for this file: `cademi commands settings --json` (offline, no credential needed).

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

### `cademi settings admin-area` — Admin area settings

Controls the administrator dashboard logo, icon and accent color.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

#### `cademi settings admin-area get`

Retrieve admin area settings · `GET /api/v3/settings/admin-area` · permission `settings.read`

Returns the appearance settings of the account's admin area: logo, icon, and accent color.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings admin-area get --json
```

#### `cademi settings admin-area update`

Update admin area settings · `PATCH /api/v3/settings/admin-area` · permission `settings.update`

Updates the admin area settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged. Images are referenced by the ID of a file that belongs to the account.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `accent_color` | string, nullable |  | Changes buttons, links, active menu items, and other accent elements. |
| `icon` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. Shown when the menu is collapsed. With no upload, uses the favicon; with no favicon, the default icon. |
| `logo` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. Horizontal version, no transparent margins. Final rendering will be 35px tall. |

Full schema: `cademi commands settings admin-area update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings admin-area update -f accent_color=<accent_color> --if-match '"<etag>"' --json

# full body from a file
cademi settings admin-area update --data @body.json --json
```

### `cademi settings app` — Progressive web app settings

Controls the installed app name, icon, screenshots, theme and Android association.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

#### `cademi settings app get`

Retrieve app settings · `GET /api/v3/settings/app` · permission `settings.read`

Returns the progressive web app (PWA) settings of the account: name, icon, theme color, Android app association, and screenshots.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings app get --json
```

#### `cademi settings app update`

Update app settings · `PATCH /api/v3/settings/app` · permission `settings.update`

Updates the progressive web app (PWA) settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged. Images are referenced by the ID of a file that belongs to the account.

`android.package` and `android.sha256_fingerprint` are independent: each one changes only when it is sent, and setting `android` to `null` clears both. The Android app association is published only when both are configured.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `android` | object, nullable |  | Only supplied child fields change. Set this object to null to clear both fields. The association is published only when both are configured. Connect your Android app to this domain using Digital Asset Links. |
| `android.package` | string, nullable |  | Package name |
| `android.sha256_fingerprint` | string, nullable |  | Certificate SHA256 |
| `icon` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. Dashboard image guidance: Square PNG, 1024×1024, up to 350kb. |
| `name` | string, nullable |  | Name shown when installing the app and on the home screen. |
| `screenshots` | array of string, nullable |  | Images shown during app installation. You can add more than one. |
| `theme_color` | string, nullable |  | Used in areas of the app interface, depending on the device. |

Full schema: `cademi commands settings app update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings app update -f android.package=<android.package> --if-match '"<etag>"' --json

# full body from a file
cademi settings app update --data @body.json --json
```

### `cademi settings authentication` — Authentication settings

Controls student passwords, login protection, Google sign-in, device limits and access-screen appearance.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

#### `cademi settings authentication get`

Retrieve authentication settings · `GET /api/v3/settings/authentication` · permission `settings.read`

Returns the authentication settings of the account: password rules, reCAPTCHA, two-factor authentication for administrators, Google sign-in, the device limit, and the layout of the access screens.

Secrets are never returned; the `configured` fields indicate whether they have been set.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings authentication get --json
```

#### `cademi settings authentication update`

Update authentication settings · `PATCH /api/v3/settings/authentication` · permission `settings.update_security`

Updates the authentication settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged. Images are referenced by the ID of a file that belongs to the account.

Secrets such as `recaptcha.secret` and `google_login.client_secret` are write-only. Changes take effect at the next login; active sessions are not terminated.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `access_screens` | object, nullable |  | Appearance of login, registration and password recovery screens. |
| `access_screens.accent_color` | string, nullable |  | Buttons, links and the background of the access screens come from this color, in all four models. |
| `access_screens.background` | object, nullable |  |  |
| `access_screens.background.image` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. Image shown in the panel beside the access form. |
| `access_screens.background.position` | string, nullable |  | Image position |
| `access_screens.background.size` | string, nullable |  | CSS background sizing value for the access-screen image. |
| `access_screens.form_position` | string, nullable |  | Choose which side displays the form on larger screens. |
| `access_screens.logo` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. Shown at the top of login, sign-up, and password-recovery screens. |
| `access_screens.logo_align` | string, nullable |  | Alignment |
| `access_screens.model` | string, nullable |  | Which model does this platform use? |
| `access_screens.phrase` | string, nullable |  | Shown next to the form. Leave empty to use the model's own sentence. |
| `access_screens.theme` | string, nullable |  | Applies to all four models. Following the area, it matches the mode chosen for the student area. |
| `admin_two_factor_available` | boolean |  | When enabled, each administrator can set up verification in their own Profile. This option does not require the second factor. |
| `google_login` | object, nullable |  | Google sign-in credentials for students. Active sessions are not terminated by these changes. |
| `google_login.client_id` | string, nullable |  | Client ID |
| `google_login.client_secret` | string, nullable |  | Write-only credential. null clears the stored value. |
| `google_login.enabled` | boolean |  | Show “Sign in with Google” on the student login screen. |
| `max_devices` | integer |  | Maximum concurrent devices per student. Must be an integer from 1 to 999; omitted values remain unchanged. |
| `password` | object, nullable |  | Initial password source and password-strength policy. |
| `password.default_password` | string, nullable |  | Write-only. Set null to clear the configured default password. All new students will receive this same password. Leave it blank to generate an individual password automatically. |
| `password.field` | string, nullable |  | Optional. Choose email or tax ID; to set a temporary password or generate one automatically, select “Do not use student information”. One of: `email`, `doc`, `none` |
| `password.rules` | object, nullable |  | Set the minimum requirements. Enter 0 to make a category optional. |
| `password.rules.digits` | integer |  | Numbers |
| `password.rules.letters` | integer |  | Letters in total |
| `password.rules.lowercase` | integer |  | Lowercase letters |
| `password.rules.min_length` | integer |  | Minimum length |
| `password.rules.special_chars` | integer |  | Special characters |
| `password.rules.uppercase` | integer |  | Uppercase letters |
| `password.strong` | boolean |  | Applies the configured criteria to every password that is created or changed. |
| `recaptcha` | object, nullable |  | reCAPTCHA configuration for student registration. |
| `recaptcha.enabled` | boolean |  | Enable protection against automated submissions in new registrations. |
| `recaptcha.secret` | string, nullable |  | Write-only credential. null clears the stored value. |
| `recaptcha.site_key` | string, nullable |  | Site key |

Full schema: `cademi commands settings authentication update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings authentication update -f access_screens.accent_color=<access_screens.accent_color> --if-match '"<etag>"' --json

# full body from a file
cademi settings authentication update --data @body.json --json
```

### `cademi settings branding` — Branding settings

Controls the student-area visual identity and certificate logos.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

#### `cademi settings branding get`

Retrieve branding settings · `GET /api/v3/settings/branding` · permission `settings.read`

Returns the branding settings of the account: accent color, logo, favicon, and certificate logos.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings branding get --json
```

#### `cademi settings branding update`

Update branding settings · `PATCH /api/v3/settings/branding` · permission `settings.update`

Updates the branding settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged. Images are referenced by the ID of a file that belongs to the account.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `accent_color` | string, nullable |  | Accent color used by the student area for highlighted interface elements. Send null to clear the custom color. |
| `certificate_logo` | object, nullable |  | Used when issuing certificates. Upload both versions: the theme set on the product certificate decides which one is used. |
| `certificate_logo.dark` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. |
| `certificate_logo.light` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. |
| `favicon` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. Shown in the browser tab and used as the admin dashboard's fallback icon. |
| `logo` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. Horizontal version, no transparent margins. Final height of 35px. |

Full schema: `cademi commands settings branding update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings branding update -f accent_color=<accent_color> --if-match '"<etag>"' --json

# full body from a file
cademi settings branding update --data @body.json --json
```

### `cademi settings configuration applications` — Apply a configuration plan

#### `cademi settings configuration applications create`

Apply a configuration plan · `POST /api/v3/configuration/applications` · permission `configuration.apply`

Applies a configuration plan previously returned by the plan operation. Submit the plan's `id` as `plan_id`, together with its `signature` and `steps`.

Before accepting the request, the API verifies that the plan is still valid, that its signature matches the submitted steps, and that none of the affected resources has changed since the plan was calculated. A plan can only be applied with the same credentials used to create it, and only once.

The plan is applied asynchronously. The response returns an operation, and the `Location` header points to it so you can track progress.

Steps are not applied as a single transaction: if a step fails, steps that were already applied remain in effect.

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `plan_id` | string | yes |  |
| `signature` | string | yes | The `signature` value returned with the plan. |
| `steps` | array of object | yes | The plan steps exactly as returned with the plan. Write-only values appear as `"***"` in the plan; replace them with the actual values before submitting. This replacement does not invalidate the signature. Submitting `"***"` as a value is… |

Full schema: `cademi commands settings configuration applications create --schema --json`

Legacy path: `cademi configuration applications create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings configuration applications create -f plan_id=<plan_id> -f signature=<signature> --data '{"steps":[...]}' --wait --json

# full body from a file
cademi settings configuration applications create --data @body.json --wait --json
```

### `cademi settings configuration plans` — Create a configuration plan

#### `cademi settings configuration plans create`

Create a configuration plan · `POST /api/v3/configuration/plans` · permission `configuration.plan`

Calculates the steps required to bring the account to the state described in a configuration manifest, and returns them as a signed plan.

This operation is synchronous and does not modify any resource. The plan expires 24 hours after it is created, as indicated by `expires_at`. To apply it, submit the plan to the apply operation before it expires.

If the manifest is invalid, the API returns `validation_failed` with the same error codes reported by the validation operation, and no plan is created.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `$schema` | string |  |  |
| `order` | object |  | Ordered sets keyed by set name (for example, `showcases` or `menu_items`). Each value must be the complete list for its scope. |
| `resources` | array of object | yes |  |

Full schema: `cademi commands settings configuration plans create --schema --json`

Legacy path: `cademi configuration plans create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings configuration plans create --data '{"resources":[...]}' --json

# full body from a file
cademi settings configuration plans create --data @body.json --json
```

### `cademi settings configuration validations` — Validate a configuration manifest

#### `cademi settings configuration validations create`

Validate a configuration manifest · `POST /api/v3/configuration/validations` · permission `configuration.validate`

Validates a configuration manifest without modifying any resource.

Content problems, such as invalid fields, unresolved references, or incomplete ordered sets (`order_set_mismatch`), are reported in the validation report, and the response status is still `200`. Check the `valid` field to determine the result.

The request is rejected with `422` only when the manifest cannot be evaluated: `too_many_items` when it exceeds the maximum number of resources, and `unsupported_resource` when it contains an unsupported resource kind.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `$schema` | string |  |  |
| `order` | object |  | Ordered sets keyed by set name (for example, `showcases` or `menu_items`). Each value must be the complete list for its scope. |
| `resources` | array of object | yes |  |

Full schema: `cademi commands settings configuration validations create --schema --json`

Legacy path: `cademi configuration validations create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings configuration validations create --data '{"resources":[...]}' --json

# full body from a file
cademi settings configuration validations create --data @body.json --json
```

### `cademi settings custom-code` — Manage custom scripts and styles

Manage custom code applied to the platform's presentation.

Related settings:
- `cademi settings branding` — Controls the student-area visual identity and certificate logos. (`cademi settings branding get|update`)

#### `cademi settings custom-code create`

Create a custom code block · `POST /api/v3/settings/custom-code` · permission `custom_code.write`, `custom_code.manage_pwa (if transition:type=pushalertco)`

Creates an active custom code block. Creation enables it immediately; no separate activation request is needed. The create operation does not accept status.

Blocks created through this API have no login-only restriction. Their integration renderer is used by student-area and sign-in page layouts; OAuth consent sign-in and two-factor pages suppress instance scripts. The integration type determines the rendered code.

Creating a block with `type` set to `pushalertco` also requires the `custom_code.manage_pwa` permission.

Code content is stored exactly as submitted and is not sanitized.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `content` | object |  | Code content. Keys that are omitted are stored empty. `sw_js` and `manifest_json` are used only by `pushalertco` blocks. |
| `content.body` | string, nullable |  | Code injected into the page `<body>`. |
| `content.header` | string, nullable |  | Code injected into the page `<head>`. |
| `content.manifest_json` | string, nullable |  | Web app manifest. |
| `content.sw_js` | string, nullable |  | Service worker script. |
| `name` | string | yes | Identifies the block in this list. Students never see this name. |
| `position` | string | yes | Required input accepted as `head` or `body`, but it does not select the injection location. Set `content.header` and/or `content.body` to populate those locations. The returned position is derived from the saved content: `head` when… |
| `type` | string | yes | Selects the integration implementation. Accepted values: `active-campaign`, `custom`, `facebook-pixel`, `analytics`, `tagmanager`, `pushalertco`, `whats`. Creating a `pushalertco` block also requires `custom_code.manage_pwa`. The API… |

Full schema: `cademi commands settings custom-code create --schema --json`

Legacy path: `cademi custom-code create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings custom-code create -f name=<name> -f position=head -f type=<type> --json

# full body from a file
cademi settings custom-code create --data @body.json --json
```

#### `cademi settings custom-code delete <custom_code_id>`

Delete a custom code block · `DELETE /api/v3/settings/custom-code/{custom_code_id}` · permission `custom_code.write`

Moves the custom code block to the trash instead of permanently deleting it.

A block in the trash can be restored with the update operation by sending `deleted: false`.

**Arguments:**
- `custom_code_id` — Public ID of the custom code, prefixed with `cc_`. Example: `cc_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi custom-code delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi settings custom-code delete cc_42 --yes
```

#### `cademi settings custom-code get <custom_code_id>`

Retrieve a custom code block · `GET /api/v3/settings/custom-code/{custom_code_id}` · permission `custom_code.read`

Retrieves a custom code block by its public ID, including blocks in the trash.

The block metadata is always returned. The `content` object (`header`, `body`, `sw_js`, and `manifest_json`) is included only when the credentials also have the `custom_code.read_content` permission, because code content often contains third-party tokens.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the block to avoid overwriting a newer version.

**Arguments:**
- `custom_code_id` — Public ID of the custom code, prefixed with `cc_`. Example: `cc_42`

**Flag sets:** output

Legacy path: `cademi custom-code get`

**Examples:**

```bash
# get
cademi settings custom-code get cc_42 --json
```

#### `cademi settings custom-code list`

List custom code blocks · `GET /api/v3/settings/custom-code` · permission `custom_code.read`

Returns the metadata of the account's custom code blocks, using cursor-based pagination.

By default, only blocks that are not in the trash are returned. Set `deleted=true` to list only blocks in the trash.

The `content` object is never included in this collection. To read a block's code, use the retrieve operation with the `custom_code.read_content` permission.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

Legacy path: `cademi custom-code list`

**Examples:**

```bash
# list: one page, machine-readable
cademi settings custom-code list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi settings custom-code list --limit 200 --raw --json

# every page, projected
cademi settings custom-code list --all --jq '[.[] | {id}]'
```

#### `cademi settings custom-code update <custom_code_id>`

Update a custom code block · `PATCH /api/v3/settings/custom-code/{custom_code_id}` · permission `custom_code.write`, `custom_code.activate (if field:status)`

Updates an existing custom code block. Only the fields present in the request are changed.

Changing `status` also requires the `custom_code.activate` permission. Changing the `name` or `content` of a `pushalertco` block also requires the `custom_code.manage_pwa` permission.

Send `deleted: false` to restore a block from the trash.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting changes made since the block was retrieved.

**Arguments:**
- `custom_code_id` — Public ID of the custom code, prefixed with `cc_`. Example: `cc_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `content` | object |  | Code content. Only the keys included are changed. |
| `content.body` | string, nullable |  | Code injected into the page `<body>`. |
| `content.header` | string, nullable |  | Code injected into the page `<head>`. |
| `content.manifest_json` | string, nullable |  | Web app manifest. |
| `content.sw_js` | string, nullable |  | Service worker script. |
| `deleted` | boolean |  | Send `false` to restore a block from the trash. One of: `false` |
| `name` | string |  | Identifies the block in this list. Students never see this name. |
| `status` | string |  | `active` enables the block; `inactive` keeps it as a draft. Changing this field also requires `custom_code.activate`. While in draft, the code is not injected into the pages. One of: `active`, `inactive` |

Full schema: `cademi commands settings custom-code update --schema --json`

**Additional permission checks:**
- `custom_code.manage_pwa` when `stored:type=pushalertco AND any-field:name,content`

Legacy path: `cademi custom-code update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings custom-code update cc_42 -f content.body=<content.body> --if-match '"<etag>"' --json

# full body from a file
cademi settings custom-code update cc_42 --data @body.json --json
```

### `cademi settings custom-fields` — Define custom fields for student profiles

Manage field definitions and types. Read or update a student's values
through users custom-fields.

Related: `cademi users custom-fields`

#### `cademi settings custom-fields create`

Create a custom field · `POST /api/v3/settings/user-profile/custom-fields` · permission `custom_fields.create`

Creates a custom field definition for user profiles in the current account.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `label` | string | yes | Field name shown when viewing or editing a student profile. |
| `position` | integer |  | Nonnegative ordering value for the custom field definition list. Lower values appear first, with the field ID breaking ties. Omit to place the new definition after the existing definitions. |
| `required` | boolean |  | Stores whether the field is marked as required. Current user creation, custom-field value updates and student profile forms do not enforce this flag: omitted or empty values remain allowed. Omit to create the definition with this flag set… |
| `type` | string | yes | Profile input type. When updating values through `users.custom_fields.update`, `number` requires a numeric value and `date` requires `YYYY-MM-DD`; `text` accepts text. One of: `text`, `number`, `date` |

Full schema: `cademi commands settings custom-fields create --schema --json`

Legacy path: `cademi custom-fields create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings custom-fields create -f label=<label> -f type=text --json

# full body from a file
cademi settings custom-fields create --data @body.json --json
```

#### `cademi settings custom-fields delete <custom_field_id>`

Delete a custom field · `DELETE /api/v3/settings/user-profile/custom-fields/{custom_field_id}` · permission `custom_fields.delete`

Deletes a custom field definition.

Only the definition is removed. Values that users have already filled in for this field are retained but are no longer returned with the user's custom field values.

**Arguments:**
- `custom_field_id` — Public ID of the custom field, prefixed with `cfd_`. Example: `cfd_7`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi custom-fields delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi settings custom-fields delete cfd_7 --yes
```

#### `cademi settings custom-fields get <custom_field_id>`

Retrieve a custom field · `GET /api/v3/settings/user-profile/custom-fields/{custom_field_id}` · permission `custom_fields.read`

Retrieves a custom field definition by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the custom field to avoid overwriting a newer version.

**Arguments:**
- `custom_field_id` — Public ID of the custom field, prefixed with `cfd_`. Example: `cfd_7`

**Flag sets:** output

Legacy path: `cademi custom-fields get`

**Examples:**

```bash
# get
cademi settings custom-fields get cfd_7 --json
```

#### `cademi settings custom-fields list`

List custom fields · `GET /api/v3/settings/user-profile/custom-fields` · permission `custom_fields.read`

Lists the custom field definitions available for user profiles in the current account.

Returns all definitions in a single response, ordered by `position`.

**Flag sets:** output

Legacy path: `cademi custom-fields list`

**Examples:**

```bash
# get
cademi settings custom-fields list --json
```

#### `cademi settings custom-fields update <custom_field_id>`

Update a custom field · `PATCH /api/v3/settings/user-profile/custom-fields/{custom_field_id}` · permission `custom_fields.update`

Updates a custom field definition. Fields omitted from the request keep their current values.

The `type` of a field cannot be changed once users have filled in values for it. Such requests return the `state_conflict` error code.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header.

**Arguments:**
- `custom_field_id` — Public ID of the custom field, prefixed with `cfd_`. Example: `cfd_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `label` | string |  | Field name shown when viewing or editing a student profile. |
| `position` | integer |  | Nonnegative ordering value for the custom field definition list. Lower values appear first, with the field ID breaking ties. Omit to preserve the current position. |
| `required` | boolean |  | Stores whether the field is marked as required. Current user creation, custom-field value updates and student profile forms do not enforce this flag: omitted or empty values remain allowed. Omit to preserve the current flag. |
| `type` | string |  | Profile input type. When updating values through `users.custom_fields.update`, `number` requires a numeric value and `date` requires `YYYY-MM-DD`; `text` accepts text. Changing the type is rejected with `state_conflict` if any user value… |

Full schema: `cademi commands settings custom-fields update --schema --json`

Legacy path: `cademi custom-fields update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings custom-fields update cfd_7 -f label=<label> --if-match '"<etag>"' --json

# full body from a file
cademi settings custom-fields update cfd_7 --data @body.json --json
```

### `cademi settings embedded-pages` — Embedded page settings

Controls external pages shown at home or first access and whether dynamic content is enabled.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

#### `cademi settings embedded-pages get`

Retrieve embedded page settings · `GET /api/v3/settings/embedded-pages` · permission `settings.read`

Returns the settings for pages embedded in the student area: the home page URL, the first-access page URL, and whether dynamic content is enabled.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings embedded-pages get --json
```

#### `cademi settings embedded-pages update`

Update embedded page settings · `PATCH /api/v3/settings/embedded-pages` · permission `settings.update`

Updates the embedded page settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged.

`home_url` and `first_access_url` must use the HTTP or HTTPS scheme.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `dynamic_content_enabled` | boolean, nullable |  | When enabled, you can add an embedded page to each lesson. |
| `first_access_url` | string, nullable |  | Must use HTTP or HTTPS. Shown only the first time the student accesses the platform. |
| `home_url` | string, nullable |  | Must use HTTP or HTTPS. Shown when the student opens the student area. |

Full schema: `cademi commands settings embedded-pages update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings embedded-pages update -F dynamic_content_enabled=true --if-match '"<etag>"' --json

# full body from a file
cademi settings embedded-pages update --data @body.json --json
```

### `cademi settings gamification` — Gamification settings

Controls ranking presentation and the activities that award points. Changes affect scoring when subsequent activity events are processed; saving settings does not recalculate previously awarded points or replay past activities.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

Related: `cademi gamification`

#### `cademi settings gamification get`

Retrieve gamification settings · `GET /api/v3/settings/gamification` · permission `gamification.read`

Returns the gamification settings of the account: ranking visibility, the scoring rule for comments, and the triggers that award points.

Requires the `gamification.read` permission; `settings.read` does not grant access to this group.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings gamification get --json
```

#### `cademi settings gamification update`

Update gamification settings · `PATCH /api/v3/settings/gamification` · permission `gamification.update`

Updates the gamification settings. Only the fields included in the request are changed, and triggers omitted from `triggers` are left unchanged.

A trigger's `points` value must be at least 1. To stop a trigger from awarding points, set its `enabled` field to `false`.

Requires the `gamification.update` permission. To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `comment_scoring` | string |  | Comments score when a student submits them, without waiting for moderation approval. once awards points once per student and lesson; unlimited allows each distinct comment submission to score. Changing this rule does not recalculate… |
| `enabled` | boolean |  | Shows the points ranking in the student area. |
| `show_full_name` | boolean |  | Shows students' full names in the ranking. If disabled, shows only the first name. |
| `show_to_students` | boolean |  | Shows how many students are taking part in the ranking. |
| `triggers` | object |  | Only supplied triggers and child fields change. manual is not a configurable trigger. Points must be at least 1; disable a trigger with enabled=false. Choose which student actions earn points and how much each one is worth. |
| `triggers.certificate_issued` | object |  | Awards points for certificate issuance, once per student and product even if another certificate is issued. |
| `triggers.certificate_issued.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.certificate_issued.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |
| `triggers.comment` | object |  | Awards points when a student submits a comment, without waiting for moderation approval. Repetition follows comment_scoring. |
| `triggers.comment.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.comment.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |
| `triggers.course_completed` | object |  | Awards points when a course is completed, once per student and product. |
| `triggers.course_completed.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.course_completed.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |
| `triggers.course_progress_50` | object |  | Awards points when the 50% course progress milestone is reached, once per student and product. |
| `triggers.course_progress_50.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.course_progress_50.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |
| `triggers.course_progress_75` | object |  | Awards points when the 75% course progress milestone is reached, once per student and product. |
| `triggers.course_progress_75.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.course_progress_75.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |
| `triggers.course_progress_90` | object |  | Awards points when the 90% course progress milestone is reached, once per student and product. |
| `triggers.course_progress_90.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.course_progress_90.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |
| `triggers.course_started` | object |  | Awards points when a course is started, once per student and product. |
| `triggers.course_started.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.course_started.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |
| `triggers.exam_passed` | object |  | Awards points for a finalized passing exam attempt, once per student and exam. An attempt already deleted does not award points. |
| `triggers.exam_passed.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.exam_passed.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |
| `triggers.lesson_completed` | object |  | Awards points for a lesson completion event, once per student and lesson. |
| `triggers.lesson_completed.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.lesson_completed.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |
| `triggers.question` | object |  | Awards points when a student submits a lesson question, without waiting for moderation or an answer. Once per student and lesson. |
| `triggers.question.enabled` | boolean |  | Whether this activity awards points. |
| `triggers.question.points` | integer |  | Points awarded for the activity. Use enabled=false to disable scoring. |

Full schema: `cademi commands settings gamification update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings gamification update -f comment_scoring=once --if-match '"<etag>"' --json

# full body from a file
cademi settings gamification update --data @body.json --json
```

### `cademi settings platform` — Platform settings

Controls the student-area language, navigation, appearance and footer.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

#### `cademi settings platform get`

Retrieve platform settings · `GET /api/v3/settings/platform` · permission `settings.read`

Returns the platform settings of the account, such as language, title, showcase layout, dark mode, font size, sidebar visibility, and footer options.

The `defaults` object contains the default values, which lets you distinguish configured values from defaults. The `revision` field identifies the current revision of the settings.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings platform get --json
```

#### `cademi settings platform update`

Update platform settings · `PATCH /api/v3/settings/platform` · permission `settings.update`

Updates the platform settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `dark_mode` | boolean, nullable |  | Display mode for the student area: true enables dark mode; false or null selects light mode. Administrator display preferences are configured separately. |
| `font_size` | string, nullable |  | Larger text across the whole student area, for those who need more legibility. One of: `padrao`, `ampliado` |
| `footer` | object, nullable |  | Set the footer for the student area and access screens. |
| `footer.disclaimer` | string, nullable |  | Legal text displayed persistently in the footer. |
| `footer.hide_privacy` | boolean, nullable |  | Whether the privacy-policy link is hidden in the footer. |
| `footer.hide_support` | boolean, nullable |  | Whether the support link is hidden in the footer. |
| `footer.hide_terms` | boolean, nullable |  | Whether the terms-of-use link is hidden in the footer. |
| `language` | string, nullable |  | Default interface language shown to students. |
| `showcase_layout` | string, nullable |  | Student-area navigation model: `default` selects the educational layout and `netflix` selects the streaming-style showcase. Send null to clear the configured model. One of: `default`, `netflix` |
| `sidebar_hidden` | boolean, nullable |  | Whether the student-area sidebar is hidden. |
| `title` | string, nullable |  | Text shown in the browser tab on every page. |

Full schema: `cademi commands settings platform update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings platform update -F dark_mode=true --if-match '"<etag>"' --json

# full body from a file
cademi settings platform update --data @body.json --json
```

### `cademi settings registration` — Registration settings

Controls free sign-up channels, the delivery granted to new students, allowed email domains and document requirements.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

Related: `cademi users`, `cademi sales deliveries`

#### `cademi settings registration get`

Retrieve registration settings · `GET /api/v3/settings/registration` · permission `settings.read`

Returns the registration settings of the account: public sign-up, domain restrictions, whether a document is required, and the highlight applied to users with free access.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings registration get --json
```

#### `cademi settings registration update`

Update registration settings · `PATCH /api/v3/settings/registration` · permission `settings.update_registration`

Updates the registration settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `document` | object, nullable |  | Student document requirements for free registration. |
| `document.required` | boolean |  | Whether a valid student document is required for free registration. |
| `domain_restriction` | object, nullable |  | Allowed email domains. Enabling restriction requires a nonempty list in the merged saved configuration. |
| `domain_restriction.allowed` | array of string, nullable |  | Authorized domains |
| `domain_restriction.enabled` | boolean |  | Allow registration only with email addresses from authorized domains. This rule also applies to new students who sign in with Google. |
| `highlight_free` | object, nullable |  | How free-registration students appear in the administrator student list. |
| `highlight_free.enabled` | boolean |  | In the student list, anyone who joined through free sign-up gets an icon next to their name. |
| `highlight_free.icon` | string, nullable |  | The symbol shown next to the name in the student list. |
| `public_signup` | object, nullable |  | Free student registration settings. |
| `public_signup.by_url` | boolean |  | Allow sign-up via a form on an external site |
| `public_signup.enabled` | boolean |  | Allow sign-up on the login page |
| `public_signup.free_delivery_id` | string, nullable |  | Delivery public ID prefixed with dlv_, belonging to this account. null removes the free-registration delivery. Students who register for free will have access to this delivery. Select “None” to create the account without granting a… |

Full schema: `cademi commands settings registration update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings registration update -F document.required=true --if-match '"<etag>"' --json

# full body from a file
cademi settings registration update --data @body.json --json
```

### `cademi settings security` — Content protection settings

Controls document and video identification and credentials for video-provider integrations.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

#### `cademi settings security get`

Retrieve security settings · `GET /api/v3/settings/security` · permission `settings.read`

Returns the content protection settings of the account: social DRM watermark, YouTube player protection, and video provider integrations.

Provider credentials are never returned; the `configured` fields indicate whether they have been set.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings security get --json
```

#### `cademi settings security update`

Update security settings · `PATCH /api/v3/settings/security` · permission `settings.update_security`

Updates the content protection settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged.

Provider credentials are write-only. Enabling a video provider requires its credentials, either already stored or sent in the same request. This check applies only to the providers included in the request, so updating other settings never fails because of a provider that is not part of the request.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `drm` | object, nullable |  | Document and YouTube identification settings. |
| `drm.enabled` | boolean |  | Adds the student's name and document (Tax ID) to protected PDFs; uses email when no document is available. |
| `drm.social` | object, nullable |  | Adjust the watermark colors, opacity, and position. |
| `drm.social.background_color` | string, nullable |  | Background color |
| `drm.social.opacity` | string, nullable |  | Opacity |
| `drm.social.position` | string, nullable |  | Position One of: `top`, `bottom`, `right`, `left` |
| `drm.social.text_color` | string, nullable |  | Text color |
| `drm.youtube` | object, nullable |  | Student identification displayed by the custom YouTube player. |
| `drm.youtube.enabled` | boolean |  | Uses the custom player to show a watermark with student data. |
| `drm.youtube.position` | string, nullable |  | Choose where identification appears in the player. One of: `top-left`, `top-right` |
| `providers` | object, nullable |  | Video-provider integration settings. Credentials are write-only. |
| `providers.panda` | object, nullable |  | Enabling Panda requires group_id and token in the merged saved configuration. Only a provider included in the request is checked. |
| `providers.panda.enabled` | boolean |  | Identifies the student by name and document. |
| `providers.panda.group_id` | string, nullable |  | Group ID |
| `providers.panda.token` | string, nullable |  | Write-only credential. null clears the stored value. |
| `providers.vdocipher` | object, nullable |  | Enabling VdoCipher requires api_secret in the merged saved configuration. Only a provider included in the request is checked. |
| `providers.vdocipher.api_secret` | string, nullable |  | Write-only credential. null clears the stored value. |
| `providers.vdocipher.enabled` | boolean |  | Without the integration the videos keep working through the link or the embed code, but with no student identification. |
| `providers.vdocipher.watermark` | boolean |  | Identifies the student with name and document over the video. |
| `providers.vdocipher.whitelist` | boolean |  | Blocks playback when the video is embedded outside your members area. |
| `providers.videofront` | object, nullable |  | Enabling Videofront requires token in the merged saved configuration. Only a provider included in the request is checked. |
| `providers.videofront.enabled` | boolean |  | Identifies the student with data sent to the integration. |
| `providers.videofront.token` | string, nullable |  | Write-only credential. null clears the stored value. |

Full schema: `cademi commands settings security update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings security update -F drm.enabled=true --if-match '"<etag>"' --json

# full body from a file
cademi settings security update --data @body.json --json
```

### `cademi settings sharing` — Sharing settings

Controls the title, description and image used when platform links are shared.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

#### `cademi settings sharing get`

Retrieve sharing settings · `GET /api/v3/settings/sharing` · permission `settings.read`

Returns the social sharing settings of the account: the title, description, and image used in Open Graph tags when platform links are shared.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings sharing get --json
```

#### `cademi settings sharing update`

Update sharing settings · `PATCH /api/v3/settings/sharing` · permission `settings.update`

Updates the social sharing settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged. The image is referenced by the ID of a file that belongs to the account.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `description` | string, nullable |  | Text displayed below the title. |
| `image` | string, nullable |  | Account file public ID prefixed with file_. null clears the configured asset. Image displayed with the title and description. |
| `title` | string, nullable |  | Text displayed prominently on social media. |

Full schema: `cademi commands settings sharing update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings sharing update -f description=<description> --if-match '"<etag>"' --json

# full body from a file
cademi settings sharing update --data @body.json --json
```

### `cademi settings tags` — Define tags used to classify students

Create and edit tag definitions here. Assign tags to a student through
users tags; deliveries may also apply tags when granting access.

Related: `cademi users tags`, `cademi sales deliveries tags`

#### `cademi settings tags create`

Create a tag · `POST /api/v3/tags` · permission `tags.create`

Creates a tag that can be assigned to users.

Leading and trailing whitespace is removed from the name. Tag names must be unique within the account; if the name is already in use, the API returns `409 Conflict` with the `already_exists` error code.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes |  |

Full schema: `cademi commands settings tags create --schema --json`

Legacy path: `cademi tags create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings tags create -f name=<name> --json

# full body from a file
cademi settings tags create --data @body.json --json
```

#### `cademi settings tags delete <tag_id>`

Delete a tag · `DELETE /api/v3/tags/{tag_id}` · permission `tags.delete`

Moves the tag to the trash and removes it from every user it is assigned to.

A tag used by an access rule cannot be deleted; the API returns `409 Conflict` with the `tag_in_use` error code.

Replicated tags are read-only and cannot be deleted. Attempts to delete them return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `tag_id` — Public ID of the tag, prefixed with `tag_`. Example: `tag_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi tags delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi settings tags delete tag_42 --yes
```

#### `cademi settings tags get <tag_id>`

Retrieve a tag · `GET /api/v3/tags/{tag_id}` · permission `tags.read`

Retrieves a tag by its public ID, including the number of users it is assigned to.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the tag to avoid overwriting a newer version.

**Arguments:**
- `tag_id` — Public ID of the tag, prefixed with `tag_`. Example: `tag_42`

**Flag sets:** output

Legacy path: `cademi tags get`

**Examples:**

```bash
# get
cademi settings tags get tag_42 --json
```

#### `cademi settings tags list`

List tags · `GET /api/v3/tags` · permission `tags.read`

Lists the user tags of the account, sorted alphabetically by name by default. Each tag includes `users_count`, the number of users it is assigned to. Results are paginated with `limit` and `cursor`.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (name, -name); API default: name

Legacy path: `cademi tags list`

**Examples:**

```bash
# list: one page, machine-readable
cademi settings tags list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi settings tags list --limit 200 --raw --json

# every page, projected
cademi settings tags list --all --jq '[.[] | {id}]'
```

#### `cademi settings tags update <tag_id>`

Update a tag · `PATCH /api/v3/tags/{tag_id}` · permission `tags.update`

Renames a tag. The new name must be unique within the account.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. If the header is omitted, the update is applied to the current revision.

Replicated tags are read-only. Attempts to update them return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `tag_id` — Public ID of the tag, prefixed with `tag_`. Example: `tag_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string |  |  |

Full schema: `cademi commands settings tags update --schema --json`

Legacy path: `cademi tags update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings tags update tag_42 -f name=<name> --if-match '"<etag>"' --json

# full body from a file
cademi settings tags update tag_42 --data @body.json --json
```

### `cademi settings user-profile` — Student profile settings

Controls which profile fields students can see or edit and how student names appear in support tools.

Omitted fields remain unchanged. Only fields explicitly documented as nullable accept null.

Related: `cademi users`

#### `cademi settings user-profile get`

Retrieve user profile settings · `GET /api/v3/settings/user-profile` · permission `settings.read`

Returns the user profile settings of the account: which profile fields users can view or edit, and whether their full name is displayed in support conversations.

The response includes an `ETag` representing the current revision of these settings; changes to other settings groups do not affect it. Send this value in the `If-Match` header when updating these settings to avoid overwriting a newer version.

**Flag sets:** output

**Examples:**

```bash
# get
cademi settings user-profile get --json
```

#### `cademi settings user-profile update`

Update user profile settings · `PATCH /api/v3/settings/user-profile` · permission `settings.update_registration`

Updates the user profile settings. Only the fields included in the request are changed: setting a field to `null` clears its configured value and restores the default, and omitted fields are left unchanged.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. The header is optional; when it is omitted, the update is applied to the current revision.

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `display` | object, nullable |  | Student-name presentation in support tools. |
| `display.full_name_in_support` | boolean |  | How the student name appears in the comments, questions and support inboxes, and in the support report. |
| `profile` | object, nullable |  | Choose which information students can change in their own profile. Blocked fields remain visible. |
| `profile.hide_address` | boolean |  | Whether the address field is hidden from the student profile. |
| `profile.hide_document` | boolean |  | Whether the document field is hidden from the student profile. |
| `profile.hide_phone` | boolean |  | Whether the phone field is hidden from the student profile. |
| `profile.lock_editing` | boolean |  | Whether students are prevented from editing their name and document. |
| `profile.lock_email` | boolean |  | Whether students are prevented from editing their email address. |

Full schema: `cademi commands settings user-profile update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings user-profile update -F display.full_name_in_support=true --if-match '"<etag>"' --json

# full body from a file
cademi settings user-profile update --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
