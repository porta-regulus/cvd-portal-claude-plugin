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

Use an API key when a browser sign-in is not possible, for example on a CI runner.

1. Create an API key in the CVD Portal dashboard. Give it only the scopes that you need.
2. Store the key in an environment variable, for example `CVD_PORTAL_API_KEY`. Do not write the key into a file in the repository.
3. Add a second server at user scope with this command.

```bash
claude mcp add --transport http --scope user cvd-portal-key https://cvdportal.com/api/mcp \
  --header "Authorization: Bearer ${CVD_PORTAL_API_KEY}"
```

4. Disable the plugin server `cvd-portal` in `/mcp`, so that only one server is active.

The shell expands the variable when you run the command. Claude Code then stores the key in plain text in `~/.claude.json`, outside the repository. Give the key a short expiry and rotate it.

The skills call tools by name. They work with either server. The permission rules in the skills pre-approve the plugin server only, so Claude Code asks before each call to `cvd-portal-key`.

## Commands

| Command | What it does | Writes to |
|---|---|---|
| `/cvd-portal:list-gaps [product]` | Lists the open conformity gaps for a product and ranks them by the evidence that the repository can supply. | Nothing. Read-only. |
| `/cvd-portal:init-cvd [slug]` | Writes `SECURITY.md` and `.well-known/security.txt` that point to `https://<slug>.cvdportal.com`. Shows the diff. Does not commit. | Local files only. |
| `/cvd-portal:sbom [product]` | Generates a CycloneDX SBOM with syft or `@cyclonedx/cdxgen`, shows the component count, and uploads it after you confirm. | The portal, after confirmation. |

`init-cvd` and `sbom` run only when you type the command. Claude does not start them by itself.

## Plan requirements

| Command | Plan | Other requirements |
|---|---|---|
| `list-gaps` | Compliance or higher. The free trial runs at Compliance level. | `read` scope. Role MEMBER or higher. |
| `init-cvd` | Any plan with a public portal. The MCP check needs Reporting or higher. Without it, the skill uses the slug that you give. | No portal write. |
| `sbom` | Reporting or higher for the upload. Compliance or higher to list products. | `sbom:write` scope. Role ADMIN. Node.js for cdxgen, or syft. |

## Rules for the agent

[AGENTS.md](AGENTS.md) holds the rules that every skill follows. The main rules are these.

- Claude does not sign off Annex I requirements or record an Article 14 decision without your approval for that specific call.
- No tool files a report with ENISA. An authorized person files on the ENISA Single Reporting Platform.
- Claude does not invent or paraphrase legal citations. Verify each citation on [EUR-Lex](https://eur-lex.europa.eu/eli/reg/2024/2847/oj).
- Each repository change names the Annex I point or the Article that it serves.

## Privacy

- The plugin contains no code that runs on your machine. It contains instructions for Claude and one MCP server address.
- The skills read files in your repository to find evidence. File contents go to the Claude model that you use. They go to CVD Portal only when a skill uploads them, for example the SBOM after you confirm.
- Vulnerability reports and Article 14 drafts stay in CVD Portal. The skills do not write them into repository files.
- The CVD Portal privacy policy is at https://cvdportal.com/privacy.

## Support

Email support@cvdportal.com or go to https://cvdportal.com/support.

## License

MIT. See [LICENSE](LICENSE).
