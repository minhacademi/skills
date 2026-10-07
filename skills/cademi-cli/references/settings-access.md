---
name: cademi-cli-settings-access
description: "Sign-in, security, registration and student profile settings — `cademi settings` (8 commands)"
metadata:
  cademi-cli: "0.2.6"
  cademi-api: "3.12.1"
---

# Settings Access Commands

> cademi 0.2.6, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

Sign-in, security, registration and student profile settings. The rest of `cademi settings` is in the sibling files below.

**Related:** `cademi config`, `cademi users tags`, `cademi users custom-fields`

**See also in this domain:** `references/settings-definitions.md`, `references/settings-platform.md`, `references/settings-emails.md`, `references/settings-legal-terms.md`, `references/settings-menus.md`, `references/settings-support.md`

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
| `admin_two_factor_available` | boolean |  | Deprecated and ignored: two-step verification is available to every administrator on every account. Kept for compatibility until v4. |
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
