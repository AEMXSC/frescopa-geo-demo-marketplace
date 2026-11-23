# frescopa-geo-demo

Scripted SEO+GEO → governed-page live demo for the AEM XSC Showcase org (Frescopa / Profee), run from Coworker.

## Skills

| Skill | Use it to | Say |
|---|---|---|
| `frescopa-geo-demo` | Run the demo: signals → draft → approve → preview page | "Run the GEO demo", "Profee demo", "protein coffee page"… then "Save the page under drafts/<your-name>" |
| `frescopa-demo-preflight` | Check connectors and pre-warm them, 2 minutes before going live | "Frescopa preflight" |
| `frescopa-demo-snapshot-refresh` | Pull fresh fallback numbers into a new `snapshot.json` to commit | "Refresh the frescopa snapshot" |
| `frescopa-demo-cleanup` | Delete your own demo pages from `drafts/<your-name>/` | "Clean up my demo pages" |

## Team rules built in
- Each presenter writes only to `drafts/<name>/` in `aemxsc/frescopa26`; never `guides/`. Existing pages are never overwritten.
- Preview only. A person publishes in Experience Workspace.
- If Semrush or ABV is unavailable, the demo renders from `references/snapshot.json`; say "live refresh pending".

## Connectors (Coworker, AEM XSC Showcase org)
Semrush, Adobe Brand Visibility (ABV), Experience Context governance, Experience Workspace (DA), plus the `visual-artifacts` skill. The skills use the default tool names (`Semrush__`, `ABV__`, `governance__`, `DA_MCP__`) and fall back to matching by tool-name suffix.

## Maintaining
- Keep `version` the same in `.claude-plugin/plugin.json` and the root `.claude-plugin/marketplace.json`; CI fails if they differ.
- Approved images are still served from `main--frescopa26--markszulc.aem.live`; move them to the aemxsc site if that one changes.
