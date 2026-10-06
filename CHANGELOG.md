# Changelog

All notable changes to the skills in this repository. Each skill carries its own version in the `metadata` block of its `SKILL.md`.

## cademi-cli 0.1.4 (2026-10-06)

- References follow the `cademi` CLI 0.2.3 (API 3.10.1): `files exports create` and the three `processing-attempts create` commands (sales events, users imports, automations diamonds memberships) accept `--wait` and `--wait-timeout`, and required query flags (`account usage --period`, `reports activity list --from/--to`, `reports lessons list --product-id`, `reports support get --from/--to`) are marked `(required)`.
- The guide checks item-level failures with `cademi operations items <operation_id>`.
- Corrected command descriptions: `auth logout` points to `cademi integrations credentials update`, and `config apply` documents `--plan` as a file.
- Version examples in the guide point to 0.2.3.

## cademi-cli 0.1.3 (2026-10-06)

- References follow the `cademi` CLI 0.2.2 (API 3.10.1): updated descriptions of account usage, where `storage.used_bytes` is recalculated daily, and of uploads, with the 50 GiB storage quota and the attachment limit set by the account.
- Version examples in the guide point to 0.2.2.

## cademi-cli 0.1.2 (2026-10-05)

- The installation section explains `cademi skills install`, which keeps this skill at the version of the installed CLI, and `cademi mcp install <client>`, with a note on when to prefer the MCP over the CLI.

## cademi-cli 0.1.1 (2026-10-05)

- References follow the `cademi` CLI 0.2.1 (API 3.10.0): new `cademi mcp` (connect AI assistants to the Cademí MCP server) and `cademi skills` (install this skill) commands.
- `users` and `settings` references split by theme: `users-learning`, `users-profile`, `settings-platform`, `settings-access` and `settings-definitions`.

## cademi-cli 0.1.0 (2026-10-04)

- First release: agent guidance for the `cademi` CLI 0.2.0 (API 3.10.0) and one reference file per command domain.
