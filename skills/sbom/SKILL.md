---
name: sbom
description: Generates a CycloneDX SBOM for the current repository and, after the user confirms, uploads it to a product in CVD Portal. Use when the user asks to create, refresh or upload an SBOM, or when list-gaps reports a missing SBOM.
when_to_use: Trigger phrases include "generate an SBOM", "upload the SBOM", "refresh the SBOM in CVD Portal", "CycloneDX". The skill runs a generator and writes to the portal only after confirmation.
argument-hint: "[product name]"
disable-model-invocation: true
allowed-tools:
  - mcp__plugin_cvd-portal_cvd-portal__get_getting_started
  - mcp__plugin_cvd-portal_cvd-portal__list_products
  - Read
  - Glob
  - Bash(command -v *)
  - Bash(syft --version)
  - Bash(jq *)
---

# Generate an SBOM and upload it to CVD Portal

Read `${CLAUDE_PLUGIN_ROOT}/AGENTS.md` before you start. Its rules apply to this skill.

## Legal basis

Regulation (EU) 2024/2847, Annex I, Part II, point 1, requires the manufacturer to identify and document the vulnerabilities and components in the product. This includes a software bill of materials in a commonly used, machine-readable format, which covers at least the top-level dependencies.

This is a summary, not a quote. Tell the user to verify the point on EUR-Lex at https://eur-lex.europa.eu/eli/reg/2024/2847/oj.

## Step 1. Choose the generator

1. Run `command -v syft`.
2. If syft is present, use syft. If not, use cdxgen through npx.
3. Tell the user which generator you will use and the command. `npx` downloads the package if it is not in the npm cache. Ask the user before the first run.

## Step 2. Generate the SBOM

Write the output to a file outside the repository, or to a path that the repository ignores. Do not commit the SBOM unless the user asks.

Use this command with syft.

```bash
syft dir:. -o cyclonedx-json=<out>/bom.cdx.json
```

Use this command with cdxgen.

```bash
npx --yes @cyclonedx/cdxgen -o <out>/bom.cdx.json .
```

Follow these rules.

- Use a temporary directory for `<out>`, for example from `mktemp -d`.
- If the generator fails, show the error. Do not upload a partial file.
- If the repository has more than one language, cdxgen scans all of them by default. Tell the user if a language had no lockfile, because the result can then be incomplete.

## Step 3. Check the result

1. Read `bomFormat` and `specVersion`. Confirm that `bomFormat` is `CycloneDX`.
2. Count the components. Use `jq '.components | length'` if jq is present. If not, read the file.
3. Show the user the generator, the format version and the component count.
4. Check the file size. The portal accepts at most 5 MB. If the file is larger, stop. Tell the user to generate the SBOM for one product or one sub-folder, or to remove the `properties` and `evidence` fields with the generator options.
5. If the count is 0, stop. Tell the user that the generator found no components. Ask the user to install the dependencies or to name the lockfile.

## Step 4. Select the product

1. Call `list_products`.
2. If `$ARGUMENTS` names a product, select it.
3. If not, show the list and ask the user which product the SBOM belongs to. Do not guess.
4. If `list_products` returns a plan error, ask the user for the product ID from the dashboard. Do not upload without a product ID unless the user asks for a company-wide SBOM.

## Step 5. Confirm and upload

1. Tell the user that the upload replaces the SBOM that the portal stores for that product. Without a product ID, it replaces the company-wide SBOM.
2. Show the user a summary: product name, product ID, format, component count and file size.
3. Ask: "Upload this SBOM to CVD Portal and replace the stored SBOM for this product?" Wait for a clear yes.
4. Call `upload_sbom` with the arguments in this list.
   - `productId` is the confirmed product ID.
   - `content` is the full SBOM file as text.
   - `filename` is `bom.cdx.json`.
5. If the tool returns a plan, scope or role error, show it. The upload needs the Reporting plan or higher, the `sbom:write` scope, and the ADMIN role. Stop.
6. If the tool returns a size, format or rate limit error, show it. Do not retry with a changed document unless the user asks.
7. Show the result: the component count, the CRA minimum-elements check and the scan counts. Tell the user that the dashboard shows the findings at `/settings/sbom`.

## What not to do

- Do not edit the SBOM to remove components. An incomplete SBOM is wrong evidence.
- Do not upload an SBOM for a product that the user did not confirm.
- Do not paste the SBOM into a public file, issue or pull request.
