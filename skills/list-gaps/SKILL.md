---
name: list-gaps
description: Read-only. Lists the open EU Cyber Resilience Act conformity gaps for a product in CVD Portal and ranks them by the evidence that the current repository can supply. Use when the user asks which CRA gaps are open, what evidence the repository has, or where to start on Annex I.
when_to_use: Trigger phrases include "list my CRA gaps", "what is missing for the CRA", "which gaps can this repo close", "Annex I gaps". Do not use to change data in the portal.
argument-hint: "[product name]"
allowed-tools:
  - mcp__plugin_cvd-portal_cvd-portal__get_getting_started
  - mcp__plugin_cvd-portal_cvd-portal__list_products
  - mcp__plugin_cvd-portal_cvd-portal__get_conformity_gaps
  - mcp__plugin_cvd-portal_cvd-portal__get_clause_mapping
  - mcp__plugin_cvd-portal_cvd-portal__lookup_regulatory_reference
  - Read
  - Glob
  - Grep
---

# List CRA gaps and rank them by repository evidence

Read `${CLAUDE_PLUGIN_ROOT}/AGENTS.md` before you start. Its rules apply to this skill.

This skill is read-only. Do not call a tool that writes to the portal. Do not edit, create or delete a file in the repository.

## Step 1. Get the account context

1. Call `get_getting_started`.
2. Show the user the summary line and the plan name.
3. If the tool returns an authentication error, tell the user to run `/mcp` and sign in to `cvd-portal`. Stop.

## Step 2. Select the product

1. Call `list_products`.
2. If the user gave a product name in `$ARGUMENTS`, select the product with that name.
3. If there is more than one product and no name, ask the user which product this repository builds. Stop until the user answers.
4. If there are no products, tell the user to register the product in the dashboard first. Stop.

## Step 3. Get the gaps

1. Call `get_conformity_gaps` for the selected product.
2. If the tool returns a plan error, tell the user that conformity gaps need the Compliance plan. Stop.
3. Keep the gap ID, the requirement, the Annex I point and the evidence that each gap names. Use the values that the tool returns. Do not add a citation that the tool did not return.

## Step 4. Inspect the repository

Look for these files. Read only what you need to decide if a file is evidence for a gap.

| Area | Files to look for |
|---|---|
| Disclosure policy and contact | `SECURITY.md`, `.github/SECURITY.md`, `docs/SECURITY.md`, `.well-known/security.txt`, `public/.well-known/security.txt` |
| Components and SBOM | `package.json` and lockfiles, `requirements*.txt`, `pyproject.toml`, `poetry.lock`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle*`, `*.csproj`, `Gemfile.lock`, `composer.lock`, existing `*.cdx.json`, `*.spdx.json`, `bom.json` |
| Dependency updates | `.github/dependabot.yml`, `renovate.json`, `.renovaterc*` |
| Security testing | CI files in `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`, `azure-pipelines.yml`. Look for SAST, dependency scanning, secret scanning, fuzzing and tests. |
| Secure updates and signing | Release workflows, signing steps, checksum files, update manifests |
| User documentation | `README*`, `docs/`, `CHANGELOG*`, support period statements |

## Step 5. Rank the gaps

Give each gap one evidence level.

| Level | Meaning |
|---|---|
| Strong | A file in the repository directly supports the gap. Name the file. |
| Partial | A file supports part of the gap. Name the file and the missing part. |
| Repo can create | The repository has no evidence, but a skill or a small change can create it. Name the skill or the change. |
| Outside repo | The evidence is a process, a contract or a decision. The repository cannot supply it. |

Sort the gaps in this order: Strong, Partial, Repo can create, Outside repo.

## Step 6. Show the result

Show one table.

| Rank | Gap | Annex I point | Evidence level | Repository evidence | Next action |
|---|---|---|---|---|---|

Follow these rules for the table.

- Use the Annex I point exactly as the tool returned it.
- In "Next action", suggest `/cvd-portal:init-cvd` for a missing disclosure policy or security.txt.
- In "Next action", suggest `/cvd-portal:sbom` for a missing SBOM.
- Do not record evidence in the portal. Tell the user that they can attach evidence in the dashboard.

After the table, write one line. It tells the user to verify each citation on EUR-Lex.
