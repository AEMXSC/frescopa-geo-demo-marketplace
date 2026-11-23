---
name: frescopa-geo-demo
description: "Run the scripted SEO+GEO → governed-page live demo for the AEM XSC Showcase org (Frescopa / Profee brand). Use when the user says \"run the GEO demo\", \"frescopa demo\", \"Frescopa GEO demo\", \"Profee demo\", \"protein coffee page\", \"create a new product for the Frescopa brand\", or wants the scripted four-stage signal → draft → approve → create flow."
metadata:
  author: AEM XSC
  version: 1.1.0
---

# Frescopa / Profee GEO Demo (scripted happy path)

Fast, deterministic run of the four-stage SEO+GEO → governed-page workflow for a live demo: signals → draft → approve → create, with no mid-run discovery. Everything slow is pre-resolved below.

Presenter narration (section beats, lines to say, watch-outs) lives in `references/talk-track.md`. Read it only when the presenter asks for narration help; don't load it on a normal run.

## Connectors (resolve once, at the start)

Use the default tool name. If it isn't available, use the tool on that connector whose name ends with the suffix shown, rather than failing. If no tool matches, tell the presenter which connector needs to be added or signed in, then continue with the fallback for that stage.

| Role | Default tool name(s) | Match by suffix |
|---|---|---|
| Semrush (SEO signals) | `Semrush__execute_report` | `execute_report` |
| Adobe Brand Visibility (GEO signals) | `ABV__read-api` | `read-api` |
| Experience Context (governance) | `governance__bga_research_context_for_usecase`, `governance__bga_evaluate_text` | `bga_research_context_for_usecase`, `bga_evaluate_text` |
| Experience Workspace | `DA_MCP__da_create_source`, `DA_MCP__da_preview_content`, `DA_MCP__da_list_sources` | `da_create_source`, `da_preview_content`, `da_list_sources` (or `da_get_source`) |

Wherever this skill names a tool below, use the resolved name.

## Pre-resolved IDs (do NOT re-discover these live)

- **Org (ABV + Experience Context):** AEM XSC Showcase — imsOrgId `708E423B67F3C2050A495C27@AdobeOrg`, ABV orgId `fbd9aaba-c175-4d6f-82e8-edee06553ba3`
- **ABV brand:** Frescopa Coffee Pods — brandId `01a0f388-02ba-72aa-b6ac-00d8ec440b73`, domain `us.frescopa.coffee`
- **Experience Context brand:** `frescopa` (brand_id for `bga_evaluate_text` and `bga_research_context_for_usecase`)
- **Semrush params:** target `frescopa.coffee`, database `us`
- **Experience Workspace:** org `aemxsc`, repo `frescopa26`, preview base `https://main--frescopa26--aemxsc.preview.da.live/`, edit base `https://da.live/canvas#/aemxsc/frescopa26/`
- **Approved media (reuse, already brand-approved):**
  - coffee hero: `https://main--frescopa26--markszulc.aem.live/media_132718180b6190179a138b13595a42c32f51665ed.jpg`
  - lifestyle/secondary: `https://main--frescopa26--markszulc.aem.live/media_126153dedaaab1e3fdd811716e201bae4bcfcf30f.jpg`
  - beans: `https://main--frescopa26--markszulc.aem.live/media_14dda38183367f1ef5f700eaf66d1a13a59bdd756.jpg`
- **Site blocks the theme styles:** `hero`, `cards`, `columns`, `accordion`. Everything else stays default content.
- **Profee** is a Frescopa product line with no ABV brand of its own. Profee pages use the Frescopa Coffee Pods signals; say so if asked.

## Save location (one folder per presenter)

Every presenter writes only to their own folder: `drafts/<name>/<slug>.html` in `aemxsc/frescopa26`. This lets the whole team run the demo at once without colliding.

- Take `<name>` from the prompt (e.g. "Save the page under drafts/jane-doe"). If it's missing, ask for it in the same question as the page menu (Stage 2). Don't ask twice.
- Normalize `<name>` to lowercase letters, digits and hyphens ("Jane Doe" → `jane-doe`).
- **Never write to `guides/`**, the site root, or another presenter's folder, even if asked to use "the default". Explain that `guides/` is shared by the whole team and use `drafts/<name>/`.
- Before creating, list `drafts/<name>/`. If `<slug>.html` already exists, use `<slug>-2` (then `-3`, …). Never overwrite a page.

## Stage 1 — Signals (one batched pull, then 7 artifacts)

Load the `visual-artifacts` skill once, up front; `add_artifact` is rejected without it.

Fire every pull in ONE `execute_code` block, concurrently (ThreadPoolExecutor), and return a single payload. No `get_report_schema` and no ABV `lookup-api-spec` at run time; the calls are inlined here. Render each artifact with `source_tool_call_id` pointing at this block's result (bind with `{"path":"/toolResult/..."}`) so large payloads never round-trip through tokens.

Calls (all independent):
- `Semrush__execute_report(report="domain_rank", params={"target":"frescopa.coffee","database":"us","export_columns":["domain","rank","organic_keywords","organic_traffic"]})`
- `Semrush__execute_report(report="resource_organic", params={"target":"frescopa.coffee","database":"us","display_limit":30,"display_sort":"position_asc","export_columns":["keyword","position","volume","cpc","url","traffic","intent"]})`
- Semrush returns `data` as a **semicolon-separated CSV string** with display headers (`Organic Keywords`, `Search Volume`, …). Parse it into rows with snake_case keys inside the block before binding artifacts. Each call costs about 60 Semrush units.
- Leader scale: the same `domain_rank` call for `nespresso.com` and `keurig.com`. **Never** run `domain_organic_organic` for frescopa.coffee: with ~4 keywords it errors, and that emptiness IS the talking point.
- One `ABV__read-api` call whose `code` does all four POSTs and returns `{presence, aivis, competitors, sov}`:
  - `await aem.post('/adobe/brand-visibility/v1/get-brand-presence-stats', { orgId, brandId })`
  - `await aem.post('/adobe/brand-visibility/v1/get-ai-visibility-stats', { orgId, domain:'us.frescopa.coffee', market:'us', months:4 })`
  - `await aem.post('/adobe/brand-visibility/v1/get-competitor-summary', { orgId, brandId })`
  - `await aem.post('/adobe/brand-visibility/v1/get-share-of-voice', { orgId, brandId, verbose:true })`
  - Read `res.body`. SoV `null` on a topic means the brand is absent (not 0%). High-popularity topics with brandMentions 0 are the money list.
  - **If brand presence returns 403:** don't retry. Use the cross-web `get-ai-visibility-stats` result for mentions and citations, and tell the presenter their ABV user needs access to the Frescopa Coffee Pods brand.

**Fallback (Semrush or ABV slow, down or signed out):** render from `references/snapshot.json` and say "live refresh pending" so nobody mistakes it for live data. Mention the snapshot's `captured` date once.

Render these seven artifacts, one `add_artifact` each (exact component specs in `references/artifacts.md`):
1. KPI snapshot: Grid of MetricCards.
2. Ranking keywords table with a ranking-band status column.
3. Keyword positions: CartesianChart variant `bar` (NOT `horizontal_bar`, which renders blank).
4. GEO gap: mentions vs citations vs cited pages.
5. Frescopa vs top competitors, mentions and citations grouped via `series:"metric"`.
6. Category topic coverage: Present/Absent plus who AI cites instead.
7. Organic SEO scale: Frescopa vs the two leaders.

Reconciled headline: both lenses point at the SAME competitors (Keurig, Nespresso) and the same root cause. The leaders have deep, question-level content, so they win rankings AND AI citations; Frescopa has neither. Fix: fast, structured, single-intent question pages. Present this as the prioritized action list.

## Stage 2 — Pick the page, then draft with brand context

Ask one question (ask_user_question) covering the page choice and, if not already given, the presenter's drafts name. Rotate the default so back-to-back runs differ.

Profee (protein coffee):
1. **Does protein coffee actually work?** FAQ-heavy, accordion.
2. **Protein coffee vs. your morning protein shake.** Comparison + cards.
3. **How much protein is in a cup of Profee?** Spec/nutrition, single intent.
4. **Pre-workout, post-workout, or anytime?** Ritual/persona angle.
5. **Protein coffee buying guide: what to look for.** Cards + accordion.
6. **How to make protein coffee at home.** How-to steps + FAQ.

Frescopa (pods): pod machine-compatibility guide; coffee-pod buying guide.

Then:
1. `bga_research_context_for_usecase(brand="frescopa", usecase=<chosen page>, include_metadata=true)`. Apply the brand voice (warm, artisanal-editorial; signature line "Fall in love with coffee. Every single day."), design tokens and claim guardrails.
2. Draft in the warm, sensory register. Guardrails: no negative competitor comparisons (name competitors neutrally), no prices or discounts, approved CTA verbs only (Shop/Learn/Discover/Start/Take/Claim/Register/Pre-order), and treat it as a regulated food category (claims review).
3. `bga_evaluate_text(brand_id="frescopa", text=...)`. Revise once for voice, competitor, price or CTA issues. Known config gaps that copy can't fix (report them as brand-owner follow-ups; don't loop): no approved claims for the Profee line; two conflicting style checks (formal-business vs evocative); sustainability-comparison checks firing on buying guides. Governance is advisory at the gate.

## Stage 3 — Approval gate

Present the draft, the opportunity rationale (which action-list item it answers) and the governance status. In production this routes through a Workfront approval with a reviewer and an audit trail; in the demo, pause and take inline approval via ask_user_question. Nothing is written before approval.

## Stage 4 — Create in Experience Workspace (preview only, NEVER publish)

Build from the homepage pattern in `references/page-template.md`: lead with a `hero` block (label + H1 + intro + hero image), not a bare H1. Use `cards`/`columns`/`accordion` for structured sections; keep everything else as default content.

1. Resolve the path per **Save location**: `drafts/<name>/<slug>.html` (slug suffixed if it already exists).
2. `da_create_source` (contentType text/html), then `da_preview_content`. Do NOT call any publish tool.
3. Return the preview URL and the edit URL, then say: "Open it in Experience Workspace to edit, check and publish. The agent never publishes; that call stays with a person." Tie it back: "This page exists because of item one on the action list."

## Guardrails
- Preview only; never publish.
- Write only under `drafts/<name>/`; never `guides/` or another presenter's folder; never overwrite.
- Reuse the approved media above; don't generate new hero images unless asked.
- Report governance config gaps honestly as brand-owner follow-ups; don't loop to force a 100% pass.
- Don't say "most complete intelligence layer in the market"; that claim is pending claims approval.
- Frescopa analytics and outcome numbers are demo data; say so if asked.
- The IDs above are authoritative for the AEM XSC Showcase org / prod sandbox.
