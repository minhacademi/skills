# Cademí skills

The collection of [Agent Skills](https://agentskills.io/specification) published by [Cademí](https://cademi.dev): portable `SKILL.md` instruction sets that teach coding agents (Claude Code, Cursor, Codex, OpenCode, Gemini CLI, GitHub Copilot, and any other Agent Skills client) how to work with Cademí products. One copy installs everywhere.

## Skills

| Skill | Install name | What it covers |
|---|---|---|
| [`skills/cademi-cli`](skills/cademi-cli/SKILL.md) | `cademi-cli` | The `cademi` CLI for API v3: discovery, authentication, input and output contracts, exit codes, safe writes, async operations, events, sandbox, config as code, plus one reference per command domain (`references/`). |

## Install

With the [`skills`](https://github.com/vercel-labs/skills) CLI, which installs into every agent it detects (`-g` for the user-wide directory):

```sh
npx skills add minhacademi/skills
npx skills add minhacademi/skills --skill cademi-cli -g
```

In Claude Code, as a plugin from this marketplace:

```text
/plugin marketplace add minhacademi/skills
/plugin install cademi-cli@minhacademi
```

By hand: copy or symlink a skill directory such as `skills/cademi-cli` into your agent's skills directory (for example `~/.claude/skills/`, `~/.agents/skills/` or `.agents/skills/` in a project), or paste its `SKILL.md` into a chat.

### The `cademi-cli` skill

Already have the CLI? Run `cademi skills install`. It installs the skill version that matches your CLI into `~/.claude/skills` and `~/.agents/skills` (`--dir .claude/skills` for a project-level copy, `cademi skills status` to check it, `cademi skills uninstall` to remove it). Updates of the CLI refresh the copies already installed.

The skill assumes the `cademi` CLI is installed and that a credential is available.

macOS and Linux:

```sh
curl -fsSL https://cli.cademi.dev/install.sh | bash
```

Windows PowerShell:

```powershell
irm https://cli.cademi.dev/install.ps1 | iex
```

Then `cademi auth login`, or export `CADEMI_API_KEY`. Docs at [cademi.dev/cli](https://cademi.dev/cli), with an index for LLMs at [cademi.dev/llms.txt](https://cademi.dev/llms.txt).

## Versions

- The `metadata` block of each `SKILL.md` records the product release the skill describes; for `cademi-cli` that is the `cademi` release and the API release (`cademi version` prints yours).
- Each skill has its own `metadata.version`; the plugin version in `.claude-plugin/` is bumped whenever any skill changes.
- `CHANGELOG.md` lists what changed in each version and which product release it matches.

## Reporting problems

Problems with Cademí products (the CLI, the API, webhooks, MCP) go to [minhacademi/developers](https://github.com/minhacademi/developers); `cademi bug --print` prefills a CLI issue. Problems with a skill, such as wrong guidance or a stale reference, are issues in this repository.

## License

See [LICENSE](LICENSE).
