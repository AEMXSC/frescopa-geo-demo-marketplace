---
name: frescopa-demo-preflight
description: "Pre-demo readiness check and connection pre-warm for the Frescopa / Profee GEO demo. Use when the user says \"frescopa preflight\", \"pre-warm the demo\", \"GEO demo check\", \"am I ready for the Frescopa demo\", ideally in a throwaway chat two minutes before going live."
metadata:
  author: Courtney Remekie
  version: 1.1.1
---

# Frescopa demo preflight

Run in a throwaway Coworker chat about two minutes before going live. Read-only: never create, edit, delete or publish anything.

If the presenter's drafts name isn't in the prompt, ask once ("Which drafts folder is yours, e.g. jane-doe?") and normalize it to lowercase letters, digits and hyphens.

## Checks (run the tool calls concurrently in one `execute_code` block where possible)

Resolve tools the same way as `frescopa-geo-demo` (default name, else the connector tool with the matching suffix).

| # | Check | Call | Pass when |
|---|---|---|---|
| 1 | Semrush signed in + warm | `Semrush__execute_report(report="domain_rank", params={"target":"frescopa.coffee","database":"us","export_columns":["domain","rank"]})` | Returns a row (costs ~60 Semrush units) |
| 2 | ABV signed in + brand access + warm | `ABV__read-api` → `aem.post('/adobe/brand-visibility/v1/get-brand-presence-stats', { orgId:'fbd9aaba-c175-4d6f-82e8-edee06553ba3', brandId:'01a0f388-02ba-72aa-b6ac-00d8ec440b73' })` | 200 with stats. 403 = signed in but no Frescopa Coffee Pods brand access (the demo still runs on the cross-web fallback) |
| 3 | Experience Context brand | `bga_research_context_for_usecase(brand="frescopa", usecase="preflight", include_metadata=false)` | Returns brand context |
| 4 | Experience Workspace access | `da_list_sources` on org `aemxsc`, repo `frescopa26`, path `drafts/` | Lists the folder. Note whether `drafts/<name>/` exists; if not, that's fine (the first page creates it) |
| 5 | Visual artifacts | Load the `visual-artifacts` skill | Loads |
| 6 | Snapshot age | Read `captured` in `../frescopa-geo-demo/references/snapshot.json` | Within 14 days. Older: warn and suggest `frescopa-demo-snapshot-refresh` after the demo |

## Output

A short table: check, ✅/⚠️/❌, and the fix for anything not green:
- Semrush ❌ → sign in to the Semrush connector in Coworker settings; on stage, the demo renders from the snapshot and you say "live refresh pending".
- ABV 403 → request access to the Frescopa Coffee Pods brand in ABV.
- Experience Context ❌ → confirm you are in the AEM XSC Showcase org.
- Experience Workspace ❌ → open https://da.live/#/aemxsc/frescopa26/drafts and check access.

End with: "Start a new Coworker chat for the live run, and end your prompt with: Save the page under drafts/<name>."
