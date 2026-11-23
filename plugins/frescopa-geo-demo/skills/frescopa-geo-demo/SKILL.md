---
name: frescopa-geo-demo
description: "Run the scripted SEO+GEO \u2192 governed-page live demo for the AEM\
  \ XSC Showcase org (Frescopa / Profee brand). Use when the user says \"run the GEO\
  \ demo\", \"frescopa demo\", \"profee demo\", or wants the fast scripted happy-path\
  \ run of the four-stage signal\u2192draft\u2192approve\u2192create flow."
metadata:
  author: user
---

# Frescopa / Profee GEO Demo (scripted happy path)

Fast, deterministic run of the four-stage SEO+GEO → governed-page workflow for a live demo. All slow discovery is pre-resolved here so the run is signals → draft → approve → create with no mid-run lookups. The governed workflow template `seo-geo-to-governed-pages.template.json` is the *productionization* story to narrate; THIS skill is what you actually run on stage.

## Slide 5 → demo crib ("the most complete intelligence layer")

This demo IS the payoff to the slide titled "The most complete intelligence layer in the market." That slide shows five signal layers converging into "One prioritized action list — ranked by impact, ready to execute." Map each layer to what's on screen as you run Stage 1:

| Slide 5 layer | Demo artifact that proves it |
|---|---|
| 1. AI prompts ("what customers ask") | ABV topic-coverage table — 14 high-interest questions, 11 Frescopa is absent from |
| 2. Market ("who AI recommends instead") | competitor mentions/citations chart — Nestlé/Keurig/Nespresso cited, Frescopa 0 |
| 3. SEO ("topics you rank for that AI overlooks") | ranking-keywords table + leader-scale contrast (4 vs 191K keywords) |
| 4. Agentic traffic | the live beverages page serving today / preview-reach point |
| 5. Owned data (AA + CJA) | where you'd close the loop on which pages drive revenue (narrate; not pulled live) |

Beats to hit:
- The reconciled Stage-1 gap ("mentioned in AI answers, never cited; question-led pages win") IS the slide's black-box "prioritized action list." Say that out loud.
- The payoff line: the action list is NOT a PDF — it powers page creation directly in Experience Workspace. Signals → governed draft → approval → live preview, one flow, same session. That's what a point solution can't do and what the slide's right-hand side promises.
- At the EW creation step, tie the artifact back explicitly: "this page exists because of item #1 on that action list." Closing the loop verbally is what makes the slide click.

## Pre-resolved IDs (do NOT re-discover these live)

- **Org (ABV + Experience Context):** AEM XSC Showcase — imsOrgId `708E423B67F3C2050A495C27@AdobeOrg`, ABV orgId `fbd9aaba-c175-4d6f-82e8-edee06553ba3`
- **ABV brand:** Frescopa Coffee Pods — brandId `01a0f388-02ba-72aa-b6ac-00d8ec440b73`, domain `us.frescopa.coffee`
- **Experience Context / governance brand:** `frescopa` (use brand_id `frescopa` for bga_evaluate_text / research_context_for_usecase)
- **Semrush MCP** — the GEO/SEO signal connector. Whenever this skill says "Semrush," it means this connector. Params: target `frescopa.coffee`, database `us`.
  - **[PLUGIN CONFIG — fill before publish]** This skill is written against a dedicated **Semrush MCP connector**. Set the connector tool prefix once here and use it everywhere below:
    - `SEMRUSH_MCP` tool prefix = `Semrush` (e.g. the connector's `execute_report` tool is `<prefix>__execute_report`, schema tool `<prefix>__get_report_schema`).
    - Sign-in lives on that Semrush connector. Every `Semrush__<tool>` reference in this skill resolves to `<prefix>__<tool>`.
  - **ABV MCP** — brand-visibility connector; ABV calls below use the `ABV__read-api` tool.
- **Experience Workspace (da.live):** org `aemxsc`, repo `frescopa26`, preview base `https://main--frescopa26--aemxsc.preview.da.live/`, edit base `https://da.live/canvas#/aemxsc/frescopa26/`
- **Approved media (reuse, already brand-approved; serving on the live beverages page):**
  - coffee hero: `https://main--frescopa26--markszulc.aem.live/media_132718180b6190179a138b13595a42c32f51665ed.jpg`
  - lifestyle/secondary: `https://main--frescopa26--markszulc.aem.live/media_126153dedaaab1e3fdd811716e201bae4bcfcf30f.jpg`
  - beans: `https://main--frescopa26--markszulc.aem.live/media_14dda38183367f1ef5f700eaf66d1a13a59bdd756.jpg`
- **Site blocks that the theme styles:** `hero`, `cards`, `columns`, `accordion`. Everything else stays default content.

Pre-demo checklist: confirm the **Semrush MCP** connector is signed in BEFORE going on stage — this is the connector that needs reconnecting if signals fail. **Pre-warm the connections** — fire one throwaway `Semrush__execute_report` (`domain_rank`) and one ABV `get-brand-presence-stats` a minute before stage so the first real pull isn't paying cold-start. If the Semrush MCP is signed out at run time, say the GEO-signal capability needs its sign-in and offer to reconnect; meanwhile render from `references/snapshot.json` and say "live refresh pending."

## Stage 1 — Signals (one batched pull, then render 7 artifacts)

**First, load `visual-artifacts`** (once, up front) — `add_artifact` is rejected without it.

**Fire every pull in ONE `execute_code` block** — the Semrush and ABV tools are callable as Python functions, so run them concurrently (ThreadPoolExecutor) and return a single payload. No `get_report_schema`, no ABV `lookup-api-spec`/`detail` discovery at runtime — the exact invocations are inlined below. Then render each artifact with `source_tool_call_id` pointing at this block's result (bind with `{"path":"/toolResult/..."}`) so big payloads never round-trip through tokens.

Exact calls to make inside the block (all independent, run together):
- `Semrush__execute_report(report="domain_rank", params={"target":"frescopa.coffee","database":"us","export_columns":["domain","rank","organic_keywords","organic_traffic"]})`
- `Semrush__execute_report(report="resource_organic", params={"target":"frescopa.coffee","database":"us","display_limit":30,"display_sort":"position_asc","export_columns":["keyword","position","volume","cpc","url","traffic","intent"]})`
- Leader scale: `Semrush__execute_report(report="domain_rank", params={"target":"nespresso.com"...})` and same for `keurig.com` (export_columns domain,rank,organic_keywords,organic_traffic). **Never** `domain_organic_organic` for frescopa.coffee — ~4 keywords, it errors (emptiness IS the talking point).
- One `ABV__read-api` call whose `code` does all four POSTs and returns `{presence, aivis, competitors, sov}`:
  - `await aem.post('/adobe/brand-visibility/v1/get-brand-presence-stats', { orgId, brandId })`
  - `await aem.post('/adobe/brand-visibility/v1/get-ai-visibility-stats', { orgId, domain:'us.frescopa.coffee', market:'us', months:4 })`
  - `await aem.post('/adobe/brand-visibility/v1/get-competitor-summary', { orgId, brandId })`
  - `await aem.post('/adobe/brand-visibility/v1/get-share-of-voice', { orgId, brandId, verbose:true })`
  - Read `res.body`. SoV `null` on a topic = brand absent (not 0%); High-popularity topics with brandMentions 0 are the money list. (Honor abv-visibility-dual-read: if brand presence 403s, use the cross-web ai-visibility stats.)

**Fallback (if Semrush or ABV is slow/down on stage):** render straight from `references/snapshot.json` (last-known-good numbers) via `data_file`, and say "live refresh pending." It's a fallback, not the primary path — numbers go stale, so refresh the snapshot before a real demo.

Then render these seven artifacts (one add_artifact each; see `references/artifacts.md` for exact, validated component specs):
1. KPI snapshot Grid of MetricCards (Semrush rank, organic keywords, organic traffic, ABV mentions, citations/cited pages, avg AI visibility score).
2. TableView of ranking keywords with a "ranking band" status column.
3. CartesianChart **variant `bar`** (NOT `horizontal_bar` — it renders blank here), x=keyword y=position.
4. CartesianChart variant `bar`, mentions vs citations vs cited pages — the GEO gap money shot.
5. CartesianChart variant `bar`, **grouped via `series:"metric"`** — Frescopa vs top competitors, Mentions and Citations side by side (competitors get cited, Frescopa 0).
6. TableView — high-interest category topics, Frescopa presence status (Present/Absent) + who AI cites instead. Absent on the educational/decision topics = the opening the pages fill.
7. TableView — organic SEO scale, Frescopa vs the two leaders (Semrush rank, organic keywords, organic traffic).

Reconciled headline: both lenses point at the SAME competitors (Keurig, Nespresso) and the same root cause — leaders have deep question-level content, so they win rankings AND AI citations; Frescopa has neither. Fix = fast, structured, single-intent question pages.

## Stage 2 — Context-grounded drafting

1. Resolve Experience Context: `governance__bga_research_context_for_usecase(brand="frescopa", usecase=<the chosen page>, include_metadata=true)`. Apply the returned brand voice (warm, artisanal-editorial; signature line "Fall in love with coffee. Every single day."), design tokens, and claim guardrails.
2. Draft the page in the warm, sensory register. Honor guardrails: no negative competitor comparisons (name competitors neutrally/with context), no prices/discounts, approved CTA verbs (Shop/Learn/Discover/Start/Take/Claim/Register/Pre-order), and treat it as a regulated food category (claims review).
3. Governance check: `governance__bga_evaluate_text(brand_id="frescopa", text=...)`. Revise once for voice/competitor/price/CTA issues. KNOWN residual config gaps that copy cannot fix (report, don't loop): no approved claims for the pod/Profee product line; two conflicting style checks (formal-business vs evocative); sustainability-comparison checks firing on buying guides. Governance is advisory at the gate.

## Page menu (vary the demo)

Ask which page to build (default rotate 1/2/5 so back-to-back runs differ). Brand can be Frescopa (pods) or **Profee** (protein coffee).

Profee (protein coffee) options:
1. **Does protein coffee actually work?** — FAQ-heavy, accordion.
2. **Protein coffee vs. your morning protein shake** — comparison table + cards.
3. **How much protein is in a cup of Profee?** — spec/nutrition single-intent.
4. **Pre-workout, post-workout, or anytime?** — ritual/persona angle.
5. **Protein coffee buying guide: what to look for** — cards + accordion (mirrors the pod guide).
6. **How to make protein coffee at home** — how-to steps + FAQ.

Frescopa (pods) options: pod machine-compatibility guide; coffee-pod buying guide.

## Stage 3 — Approval gate

Present the draft + the opportunity rationale + governance status. Narrate: in production this routes through a **Workfront approval** (reviewer, audit trail); in the demo, pause and take inline approval via ask_user_question. Nothing is written before approval.

## Stage 4 — Create in Experience Workspace (preview-only, NEVER publish)

Build the page from the **homepage pattern**: lead with a `hero` block (label + H1 + intro + hero image) — not a bare H1, which the theme renders oversized with a big top gap. Use `cards`/`columns`/`accordion` for structured sections; keep everything else as default content (EDS best practice — do not wrap plain prose in blocks).

- Default path `guides/<slug>.html` under repo `frescopa26`, org `aemxsc`.
- `DA_MCP__da_create_source` (contentType text/html), then `DA_MCP__da_preview_content` for the preview link. Do NOT call da_publish_content.
- Return edit + preview URLs. See `references/page-template.md` for the exact hero+blocks HTML skeleton and the approved image URLs to slot in.

## Guardrails
- Preview-only; never publish.
- Reuse approved media above; don't generate new hero images unless asked.
- Report governance config-gap failures honestly as brand-owner follow-ups; don't loop trying to force a 100% pass.
- Keep IDs above authoritative; this is the AEM XSC Showcase org / prod sandbox.
