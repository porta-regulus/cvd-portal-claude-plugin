---
name: sbom
description: Uploads an existing SBOM, or a dependency manifest, from the current repository to a product in CVD Portal after the user confirms. Use when the user asks to upload or refresh an SBOM in CVD Portal, or when list-gaps reports a missing SBOM. It does not generate an SBOM and does not run any program.
when_to_use: Trigger phrases include "upload the SBOM", "refresh the SBOM in CVD Portal", "send my CycloneDX file to CVD Portal". The skill reads files only and writes to the portal only after confirmation.
argument-hint: "[path to SBOM file] [product name]"
disable-model-invocation: true
allowed-tools:
  - mcp__plugin_cvd-portal_cvd-portal__get_getting_started
  - mcp__plugin_cvd-portal_cvd-portal__list_products
  - Read
  - Glob
---

# Upload an SBOM to CVD Portal

Read `${CLAUDE_PLUGIN_ROOT}/AGENTS.md` before you start. Its rules apply to this skill.

This skill does not run a program. It does not generate an SBOM. It reads a file that already exists and uploads it after the user confirms.

## Legal basis

Regulation (EU) 2024/2847, Annex I, Part II, point 1, requires the manufacturer to identify and document the vulnerabilities and components in the product. This includes a software bill of materials in a commonly used, machine-readable format, which covers at least the top-level dependencies.

This is a summary, not a quote. Tell the user to verify the point on EUR-Lex at https://eur-lex.europa.eu/eli/reg/2024/2847/oj.

## Step 1. Find the file

1. If `$ARGUMENTS` names a file path, use that file.
2. If not, look for an SBOM in the repository: `*.cdx.json`, `bom.json`, `*.spdx.json` and `sbom*.json`. Ignore `node_modules` and other dependency folders.
3. If there is more than one, show the list and ask the user which one to upload.
4. If there is no SBOM, look for a dependency manifest that the portal accepts: `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml` or `Gemfile.lock`. Tell the user that a manifest gives less detail than a real SBOM, because it has no transitive dependencies and no hashes. Ask the user to choose: upload the manifest, or create an SBOM first.
5. If the user wants an SBOM first, tell the user to run a generator and then run this command again. Show these commands as text. Do not run them.

```
syft dir:. -o cyclonedx-json=bom.cdx.json
npx @cyclonedx/cdxgen@12.8.5 -o bom.cdx.json .
```

## Step 2. Check the file

1. For a JSON SBOM, read `bomFormat` and `specVersion`, or `spdxVersion`. Count the components.
2. Check the file size. The portal accepts at most 5 MB. If the file is larger, stop and tell the user.
3. If the SBOM has 0 components, stop. Tell the user that the file lists no components.
4. Show the user the file name, the format, the version and the component count.

## Step 3. Select the product

1. Call `list_products`.
2. If `$ARGUMENTS` names a product, select it.
3. If not, show the list and ask the user which product the SBOM belongs to. Do not guess.
4. On the Reporting plan, `list_products` returns only the id and the name, with `detail` set to `basic`. That is enough for the upload.
5. If `list_products` returns a plan error, ask the user for the product ID from the dashboard. Do not upload without a product ID unless the user asks for a company-wide SBOM.

## Step 4. Confirm and upload

1. Tell the user that the upload replaces the SBOM that the portal stores for that product. Without a product ID, it replaces the company-wide SBOM.
2. Show the user a summary: product name, product ID, format, component count and file size.
3. Ask: "Upload this SBOM to CVD Portal and replace the stored SBOM for this product?" Wait for a clear yes.
4. Call `upload_sbom` with the arguments in this list.
   - `productId` is the confirmed product ID.
   - `content` is the full SBOM file as text.
   - `filename` is the name of the file, for example `bom.cdx.json` or `package.json`. The portal selects the parser from it.
5. If the tool returns a plan, scope or role error, show it. The upload needs the Reporting plan or higher, the `sbom:write` scope, and the ADMIN role. Stop.
6. If the tool returns a size, format or rate limit error, show it. Do not retry with a changed document unless the user asks.
7. Show the result: the component count, the CRA minimum-elements check and the scan counts. Tell the user that the dashboard shows the findings at `/settings/sbom`.

## What not to do

- Do not run a generator or any other program. Show the commands and let the user run them.
- Do not edit the SBOM to remove components. An incomplete SBOM is wrong evidence.
- Do not upload an SBOM for a product that the user did not confirm.
- Do not paste the SBOM into a public file, issue or pull request.
