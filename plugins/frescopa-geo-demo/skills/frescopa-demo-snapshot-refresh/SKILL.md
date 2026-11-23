---
name: frescopa-demo-snapshot-refresh
description: "Refresh the fallback snapshot (last-known-good numbers) for the Frescopa / Profee GEO demo from live Semrush and Adobe Brand Visibility data. Use when the user says \"refresh the frescopa snapshot\", \"update demo fallback numbers\", or preflight reports the snapshot is stale."
metadata:
  author: AEM XSC
  version: 1.1.0
---

# Refresh the Frescopa demo snapshot

Produces a new `snapshot.json` for `frescopa-geo-demo`. Installed plugin files are read-only, so the output is a file to commit to the marketplace repo, not an in-place update. Cost: about 240 Semrush API units (four calls at ~60 each). Semrush returns semicolon-separated CSV; parse it in code.

## 1. Pull (one `execute_code` block, concurrent)

Use exactly the Stage 1 calls from `../frescopa-geo-demo/SKILL.md`: `domain_rank` for frescopa.coffee, nespresso.com and keurig.com; `resource_organic` for frescopa.coffee; and the single ABV call returning `{presence, aivis, competitors, sov}`. If ABV brand presence returns 403, stop and report; don't write a snapshot with missing ABV numbers.

## 2. Map to the existing schema

Read `../frescopa-geo-demo/references/snapshot.json` and keep its exact shape and key names. Compute every number in code, never by eye.
- `captured` = today (YYYY-MM-DD); update the date inside `_note` to match.
- `semrush.domain_rank` from frescopa.coffee; `semrush.leader_scale` = nespresso.com, keurig.com, then frescopa.coffee.
- `semrush.ranking_keywords`: keyword, position, volume, band (1–10 "Page 1", 11–30 "Page 2-3", 31+ "Page 3+").
- `abv.brand_presence`: total_executions, total_mentions, total_citations, average_visibility_score.
- `abv.ai_visibility`: cited_pages, citations.
- `abv.competitor_summary`: top 5 competitors by mentions (name, mentions, citations); `abv.frescopa_self` from Frescopa's own row.
- `abv.topic_coverage`: High-popularity topics, Absent first, then Present; leaders = top `topCompetitors[].name` joined with ", ".
- `abv.high_popularity_topics`: total / present / absent counts.

## 3. Output

1. A short table of what changed (old → new) for the headline numbers: rank, organic keywords, organic traffic, mentions, citations, absent topics.
2. The full JSON in one code block.
3. These steps for the maintainer:
   - Replace `plugins/frescopa-geo-demo/skills/frescopa-geo-demo/references/snapshot.json` in `AEMXSC/frescopa-geo-demo-marketplace`.
   - Bump the patch version in both `plugins/frescopa-geo-demo/.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` (CI fails if they differ).
   - Commit as `chore: refresh frescopa demo snapshot <date>`.
