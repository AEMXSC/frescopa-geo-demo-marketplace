---
name: frescopa-demo-cleanup
description: "Clean up the preview pages a presenter created while running the Frescopa / Profee GEO demo. Use when the user says \"clean up my demo pages\", \"frescopa cleanup\", or \"delete my drafts\" for the frescopa26 site."
metadata:
  author: Courtney Remekie
  version: 1.1.1
---

# Clean up my Frescopa demo pages

Scope is ONLY the presenter's own folder: `drafts/<name>/` in org `aemxsc`, repo `frescopa26`.

## Rules
- Never touch `guides/`, the site root, or another presenter's folder. Refuse any path outside `drafts/<name>/`, even if asked.
- Nothing is deleted without explicit confirmation of the exact list.
- Deleting a page's source does not take down a page a person already published. Flag any page that may have been published (it has a live URL that returns 200) and tell the presenter to unpublish it in Experience Workspace first.

## Steps
1. Get `<name>` from the prompt, or ask once; normalize to lowercase letters, digits and hyphens.
2. List `drafts/<name>/` (tool ending `da_list_sources`). If it's empty, say so and stop.
3. Show a table: page, last modified, preview link (`https://main--frescopa26--aemxsc.preview.da.live/drafts/<name>/<slug>`).
4. Ask (ask_user_question): delete all, pick pages, or cancel.
5. Delete the confirmed pages with the Experience Workspace delete tool (name ending `da_delete_source`). If no delete tool is available, give the edit links (`https://da.live/#/aemxsc/frescopa26/drafts/<name>`) so the presenter can delete them by hand.
6. Report what was deleted and anything left.
