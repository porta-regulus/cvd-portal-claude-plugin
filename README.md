# CVD Portal plugin for Claude Code

This plugin connects Claude Code to [CVD Portal](https://cvdportal.com). It helps a manufacturer prepare a software repository for the EU Cyber Resilience Act, Regulation (EU) 2024/2847.

The plugin adds three commands and registers the CVD Portal MCP server.

Full documentation is at https://docs.cvdportal.com/cvd/claude-plugin.

## Install

Run these commands in Claude Code.

```
/plugin marketplace add porta-regulus/cvd-portal-claude-plugin
/plugin install cvd-portal@porta-regulus
```

Run `/reload-plugins` or start a new session after the install.

## Sign in

The plugin registers the MCP server `cvd-portal` at `https://cvdportal.com/api/mcp`. The server uses OAuth 2.1.

1. Run `/mcp`.
2. Select `cvd-portal`.
3. Sign in to CVD Portal in the browser window that opens.
4. Approve the access request.

Claude Code stores the token. You do not need an API key.

## Use an API key instead

Use an API key when a browser sign-in is not possible, for example on a CI runner. The plugin itself never reads an API key. Add the key as a separate MCP server, as described at https://docs.cvdportal.com/cvd/claude-plugin, and disable the plugin server `cvd-portal` in `/mcp` so that only one server is active.

## Commands

| Command | What it does | Writes to |
|---|---|---|
| `/cvd-portal:list-gaps [product]` | Lists the open conformity gaps for a product and ranks them by the evidence that the repository can supply. | Nothing. Read-only. |
| `/cvd-portal:init-cvd [slug]` | Writes `SECURITY.md` and `.well-known/security.txt` that point to `https://<slug>.cvdportal.com`. Shows the diff. Does not commit. | Local files only. |
| `/cvd-portal:sbom [file] [product]` | Finds an SBOM, or a dependency manifest, in the repository, shows the component count, and uploads it after you confirm. | The portal, after confirmation. |

`init-cvd` and `sbom` run only when you type the command. Claude does not start them by itself.

## Plan requirements

| Command | Plan | Other requirements |
|---|---|---|
| `list-gaps` | Compliance or higher. The free trial runs at Compliance level. | `read` scope. Role MEMBER or higher. |
| `init-cvd` | Any plan with a public portal. The MCP check needs Reporting or higher. Without it, the skill uses the slug that you give. | No portal write. |
| `sbom` | Reporting or higher. | `sbom:write` scope. Role ADMIN. An SBOM file or a dependency manifest in the repository. |

## Rules for the agent

[AGENTS.md](AGENTS.md) holds the rules that every skill follows. The main rules are these.

- Claude does not sign off Annex I requirements or record an Article 14 decision without your approval for that specific call.
- No tool files a report with ENISA. An authorized person files on the ENISA Single Reporting Platform.
- Claude does not invent or paraphrase legal citations. Verify each citation on [EUR-Lex](https://eur-lex.europa.eu/eli/reg/2024/2847/oj).
- Each repository change names the Annex I point or the Article that it serves.

## What the plugin runs, sends and fetches

| Command | Runs on your machine | Sends | Fetches |
|---|---|---|---|
| `list-gaps` | Nothing. Claude reads files in the repository. | MCP read calls to `https://cvdportal.com/api/mcp` | Your products and conformity gaps |
| `init-cvd` | Nothing. Claude writes `SECURITY.md` and `security.txt` after it shows you the diff. | One MCP read call | Your portal URL |
| `sbom` | Nothing. Claude reads the SBOM or manifest file. | The file to `upload_sbom`, only after you confirm | Your product list |

The plugin has no hooks, no scripts and no bundled binaries. The plugin runs no program on your machine. All network traffic from the plugin goes to `https://cvdportal.com/api/mcp`.

To create an SBOM first, run a generator yourself, for example `syft dir:. -o cyclonedx-json=bom.cdx.json` or `npx @cyclonedx/cdxgen@12.8.5 -o bom.cdx.json .`, then run `/cvd-portal:sbom`.

## Privacy

- The plugin contains instructions for Claude and one MCP server address. It runs no program on your machine.
- The skills read files in your repository to find evidence. File contents go to the Claude model that you use. They go to CVD Portal only when a skill uploads them, for example the SBOM after you confirm.
- Vulnerability reports and Article 14 drafts stay in CVD Portal. The skills do not write them into repository files.
- The CVD Portal privacy policy is at https://cvdportal.com/privacy.

## Support

Email support@cvdportal.com or go to https://cvdportal.com/support.

## License

MIT. See [LICENSE](LICENSE).
