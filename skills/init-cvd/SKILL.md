---
name: init-cvd
description: Writes SECURITY.md and .well-known/security.txt (RFC 9116) in the current repository. Both files point to the company's CVD Portal disclosure portal. Use when the repository has no vulnerability disclosure policy or security contact, or when the user asks to set up coordinated vulnerability disclosure.
when_to_use: Trigger phrases include "add a SECURITY.md", "set up security.txt", "add a CVD policy", "publish a vulnerability disclosure contact". The skill edits files but does not commit.
argument-hint: "[company slug]"
disable-model-invocation: true
allowed-tools:
  - mcp__plugin_cvd-portal_cvd-portal__get_getting_started
  - Read
  - Glob
---

# Set up coordinated vulnerability disclosure files

Read `${CLAUDE_PLUGIN_ROOT}/AGENTS.md` before you start. Its rules apply to this skill.

## Legal basis

Regulation (EU) 2024/2847, Annex I, Part II, point 5, requires the manufacturer to put in place and enforce a policy on coordinated vulnerability disclosure.

Annex I, Part II, point 6, requires the manufacturer to provide a contact address for the reporting of vulnerabilities.

This is a summary, not a quote. Tell the user to verify both points on EUR-Lex at https://eur-lex.europa.eu/eli/reg/2024/2847/oj.

## Step 1. Find the company slug

1. If `$ARGUMENTS` contains a slug, use it.
2. If not, ask the user for the slug. The slug is the first part of the portal host name, for example `acme` in `acme.cvdportal.com`. The dashboard shows it in Settings. Do not guess the slug.
3. Call `get_getting_started`. Use the company name in the result to confirm with the user that the slug belongs to the same company.
   If the call fails because the user is not signed in or the plan has no MCP access, continue with the slug that the user gave.
4. Tell the user the portal URL, `https://<slug>.cvdportal.com`.
5. Ask the user to confirm that the portal is turned on. If `get_getting_started` returns the step `enable_portal`, the portal is off. Tell the user to turn it on in Settings first.

## Step 2. Check the existing files

1. Look for `SECURITY.md`, `.github/SECURITY.md`, `docs/SECURITY.md` and `.well-known/security.txt`.
2. If the repository serves static files from a folder such as `public/` or `static/`, the correct path is `<that folder>/.well-known/security.txt`. Ask the user which path the web server serves.
3. If a file exists, show it and ask the user to choose: replace it, merge into it, or stop.

## Step 3. Write security.txt

Use this content. Replace `<slug>`. Set `Expires` to a date less than one year from today, in RFC 3339 format with a time zone.

```
# Security contact for this project. RFC 9116.
# Legal basis: Regulation (EU) 2024/2847, Annex I, Part II, points 5 and 6.
Contact: https://<slug>.cvdportal.com/submit
Expires: <YYYY-MM-DD>T00:00:00.000Z
Policy: https://<slug>.cvdportal.com/policy
Preferred-Languages: en
```

Follow these rules.

- Ask the user if they want more languages in `Preferred-Languages`. Use two-letter codes, separated by commas.
- Ask the user if they want a `Contact: mailto:` line. Add it only if the user gives a shared security address. Do not use a personal address unless the user asks.
- Do not add `Canonical` unless the user gives the final public URL of the file.
- Do not add an `Encryption` line unless the user gives the key URL.
- Tell the user to set a reminder to renew the file before the `Expires` date.

## Step 4. Write SECURITY.md

Use this structure. Replace `<slug>` and `<product>`. Keep it short.

```markdown
# Security policy

<!-- Regulation (EU) 2024/2847, Annex I, Part II, points 5 and 6. -->

## Report a vulnerability

Report a vulnerability in <product> through our disclosure portal at https://<slug>.cvdportal.com.

Do not report a vulnerability in a public issue, pull request or discussion.

## Our disclosure policy

Our coordinated vulnerability disclosure policy is at https://<slug>.cvdportal.com/policy.

## What to include

- The affected product and version.
- Steps to reproduce the vulnerability.
- The impact.

You can report without an account. If you give a contact address, we can ask questions and send you updates.
```

Follow these rules.

- Do not add response times, bounty terms or safe harbour terms. The policy page on the portal holds those terms. Add them only if the user gives the exact text.
- Do not list known vulnerabilities.

## Step 5. Show the change

1. Show the diff of each file.
2. Name the legal basis for each file: Annex I, Part II, points 5 and 6.
3. Do not commit. Do not push. Commit only if the user asks.
4. If the user asks for a commit, put the legal basis in the commit message.
5. Remind the user that the file on the company's own website domain is also important. Researchers look for `https://<company-domain>/.well-known/security.txt`.
