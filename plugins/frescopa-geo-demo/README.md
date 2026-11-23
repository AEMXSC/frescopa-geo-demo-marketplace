# frescopa-geo-demo — plugin-ready package

Clean, plugin-ready copy of the `frescopa-geo-demo` skill (SEO+GEO → governed-page
live demo for the AEM XSC Showcase org, Frescopa / Profee brand). Hand this folder to
whoever owns plugin distribution for the AEM XSC Showcase org so it can be packaged into
a plugin/manifest and published org-wide.

## Contents
- `SKILL.md` — the skill body
- `references/artifacts.md` — validated artifact component specs
- `references/page-template.md` — hero+blocks HTML skeleton and approved image URLs
- `references/snapshot.json` — last-known-good signal numbers (fallback render source)

## One open config item — MUST fill before publish

The skill is written against a dedicated **Semrush MCP connector** (replacing the old
"Project Happy Path MCP" wiring). The connector's real tool names were not available when
this copy was prepared, so every Semrush call is parameterized behind a single token.

In `SKILL.md`, find the `[PLUGIN CONFIG — fill before publish]` block (Pre-resolved IDs
section) and set:

- `SEMRUSH_MCP` tool prefix = the dedicated Semrush connector's tool prefix.
  Every `SEMRUSH_MCP__<tool>` reference then resolves to `<prefix>__<tool>`
  (e.g. `<prefix>__execute_report`, `<prefix>__get_report_schema`).

The report names (`domain_rank`, `resource_organic`) and params
(`target`, `database`, `export_columns`, …) are standard Semrush report semantics and
should carry over unchanged; verify against the connector's `get_report_schema` once it is
signed in.

The ABV connector calls already use the `ABV__read-api` tool and need no change.

## Notes for the packager
- This is demo-run content (a scripted happy path), not a customer-data workflow.
- IDs in `SKILL.md` (org, brand, repo, media URLs) are specific to the AEM XSC Showcase
  org / prod sandbox — confirm they are still valid at publish time.
- The workflow is preview-only and never publishes pages; keep that guardrail intact.
