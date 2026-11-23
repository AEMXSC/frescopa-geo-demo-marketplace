# Stage 1 artifact specs (copy these component shapes)

Fill `value`/`data` from the live pulls. One `add_artifact` call per artifact; the workspace tabs them.

## 1. KPI snapshot — Grid of MetricCards
```
components:
- {id: root, component: Grid, columns: 3, children: [c1,c2,c3,c4,c5,c6]}
- {id: c1, component: MetricCard, label: "Semrush Rank", value: "~22M", trend: down}
- {id: c2, component: MetricCard, label: "Organic keywords", value: 4}
- {id: c3, component: MetricCard, label: "Organic traffic / mo", value: 0, trend: flat}
- {id: c4, component: MetricCard, label: "AI-answer mentions (ABV)", value: 15}
- {id: c5, component: MetricCard, label: "Citations / cited pages", value: "0 / 0", trend: flat}
- {id: c6, component: MetricCard, label: "Site health", value: "80 / 100"}
```

## 2. Keyword table — TableView
columns: keyword (isRowHeader), position (presentation number), status (presentation status with variants Page 1=positive, "Page 2-3"=notice, "Page 3+"=negative). items from resource_organic. Band rule (compute in code): position 1–10 → "Page 1", 11–30 → "Page 2-3", 31+ → "Page 3+".

## 3. Keyword positions — CartesianChart
```
{id: root, component: CartesianChart, variant: "bar", x: "keyword", y: "position",
 data: [{keyword, position}, ...]}
```
IMPORTANT: variant MUST be `bar`. `horizontal_bar` renders blank in this client.

## 4. GEO gap — CartesianChart
```
{id: root, component: CartesianChart, variant: "bar", x: "stage", y: "count",
 data: [{stage:"AI-answer mentions", count:15}, {stage:"Citations", count:0}, {stage:"Cited pages", count:0}]}
```
This is the money shot: tall mentions bar next to two flat zeros.

Compute any totals/derived numbers in code from the pulled rows; don't eyeball them.

## 5. Competitor mentions & citations — CartesianChart (grouped)
From ABV `get-competitor-summary` `competitors[]` plus Frescopa's own mentions/citations. Grouped bars via `series`.
```
{id: root, component: CartesianChart, variant: "bar", x: "brand", y: "mentions", series: "metric",
 data: [
   {brand:"Nestlé", metric:"Mentions", mentions:250}, {brand:"Nestlé", metric:"Citations", mentions:46},
   {brand:"Keurig Dr Pepper", metric:"Mentions", mentions:214}, {brand:"Keurig Dr Pepper", metric:"Citations", mentions:34},
   ... top ~4 competitors ...,
   {brand:"Frescopa", metric:"Mentions", mentions:15}, {brand:"Frescopa", metric:"Citations", mentions:0}
 ]}
```
The `y` key stays "mentions" for both metrics (it's just the value); `series:"metric"` splits the pair. Money shot: competitors have tall citation bars, Frescopa's is 0.

## 6. Category topic coverage — TableView
From `get-share-of-voice` `topics[]`. Columns (each needs `id` + `name`; type lives under `presentation`):
- topic (isRowHeader: true)
- status, presentation {type:"status", variants:{Present:"positive", Absent:"negative"}} — "Present" if brandMentions>0 else "Absent"
- leaders (plain text) — top `topCompetitors[].name` for that topic ("who AI cites instead")
Lead with the High-popularity topics where Frescopa is Absent; include its few Present topics for contrast.

## 7. Organic SEO scale — TableView
From `domain_rank` on frescopa.coffee + the two leaders. Columns:
- domain (isRowHeader: true)
- rank, presentation {type:"number", formatOptions:{notation:"compact"}}
- keywords, presentation {type:"number", formatOptions:{notation:"standard"}}
- traffic, presentation {type:"number", formatOptions:{notation:"compact"}}
Rows: the two leaders (hundreds of thousands of keywords, 6-7 figure traffic) then frescopa.coffee (4 keywords, 0 traffic, rank ~22M).

## Validated gotchas (from live runs)
- TableView columns require `id` and `name` (NOT `key`/`title`); column type goes inside `presentation`, and `status`/`number` are presentation types, not top-level `type` (top-level `type` only allows `text`/`thumbnail`).
- CartesianChart does NOT accept `aria-label`; only id/component/data/x/y/variant/series etc.
- `horizontal_bar` renders blank in this client — always use `bar`.
