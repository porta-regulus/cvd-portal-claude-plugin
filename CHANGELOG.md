# Changelog

All notable changes to this plugin appear in this file. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.2.0] - 2026-10-20

### Changed
- The `sbom` skill no longer runs a generator. It uploads an SBOM file that already exists, or a dependency manifest such as `package.json`, after you confirm. The README shows the syft and cdxgen commands for you to run yourself. The plugin now runs no program on your machine.

## [0.1.4] - 2026-10-20

### Changed
- The CHANGELOG no longer names the icon file path, which the directory scan flags.

## [0.1.3] - 2026-10-20

### Added
- The CVD Portal icon, a 512 by 512 px PNG, for the directory listing.

## [0.1.2] - 2026-10-20

### Changed
- The `sbom` skill pins `@cyclonedx/cdxgen` to 12.8.5.
- The README lists what each command runs, sends and fetches. It no longer says that the plugin runs no code, because `sbom` runs syft or cdxgen.
- The README no longer shows a command that sends an API key from an environment variable. It links to the docs for API key setup.

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
