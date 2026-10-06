---
name: cademi-cli-settings-platform
description: "Platform-wide settings: platform, app, branding, admin area, embedded pages, sharing and gamification — `cademi settings` (14 commands)"
metadata:
  cademi-cli: "0.2.3"
  cademi-api: "3.10.1"
---

# Settings Platform Commands

> cademi 0.2.3, API 3.10.1. The live catalog is always `cademi commands <prefix> --json`.

Platform-wide settings: platform, app, branding, admin area, embedded pages, sharing and gamification. The rest of `cademi settings` is in the sibling files below.

**Related:** `cademi config`, `cademi users tags`, `cademi users custom-fields`

**See also in this domain:** `references/settings-access.md`, `references/settings-definitions.md`, `references/settings-emails.md`, `references/settings-legal-terms.md`, `references/settings-menus.md`, `references/settings-support.md`

Live catalog for this file: `cademi commands settings --json` (offline, no credential needed).

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

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
