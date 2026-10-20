# Changelog

All notable changes to this plugin appear in this file. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.1.1] - 2026-10-19

### Changed
- `init-cvd` reads the portal URL from `get_getting_started`. A company with a verified custom domain gets that domain. The command asks for the slug only when the MCP server is not connected.
- `sbom` works on the Reporting plan. `list_products` returns the product id and name there.
- The README sets `CVD_PORTAL_API_KEY` before `claude mcp add`, because an empty variable registers an empty key.

## [0.1.0] - 2026-10-19

### Added
- The `cvd-portal` MCP server at `https://cvdportal.com/api/mcp`, with browser sign-in.
- `list-gaps`, which lists open CRA conformity gaps and ranks them by repository evidence. It only reads.
- `init-cvd`, which writes `SECURITY.md` and `.well-known/security.txt`.
- `sbom`, which generates a CycloneDX SBOM and uploads it after confirmation.
- `AGENTS.md` with the rules that every skill follows.
