# Page template — homepage hero + blocks skeleton

Build every demo page on this pattern (matches the frescopa26 homepage). Lead with a `hero` block, NOT a bare H1 (a bare H1 renders oversized with a big empty top band). Keep one H1, inside the hero. Use `cards`/`columns`/`accordion` for structure; leave everything else as default content.

Approved images (reuse; already brand-approved):
- HERO: `https://main--frescopa26--markszulc.aem.live/media_132718180b6190179a138b13595a42c32f51665ed.jpg`
- SECONDARY: `https://main--frescopa26--markszulc.aem.live/media_126153dedaaab1e3fdd811716e201bae4bcfcf30f.jpg`
- BEANS: `https://main--frescopa26--markszulc.aem.live/media_14dda38183367f1ef5f700eaf66d1a13a59bdd756.jpg`

Skeleton (DA_MCP__da_create_source, contentType text/html):
```html
<body><header></header><main>
<div>
  <div class="hero">
    <div><div><picture><source srcset="HERO"><source srcset="HERO" media="(min-width: 600px)"><img src="HERO" alt="..." loading="lazy"></picture></div></div>
    <div><div><p>Eyebrow label</p><h1>Page title</h1><p>Intro — opens with "Fall in love with coffee. Every single day."</p></div></div>
  </div>
</div>
<div>
  <h2>Section with cards</h2>
  <div class="cards">
    <div><div><picture>...<img src="BEANS" alt="..."></picture></div><div><p>Label</p><h3>Card heading</h3><p>Card body.</p></div></div>
    <!-- repeat card rows -->
  </div>
</div>
<div>
  <div class="columns"><div><div><h2>Heading</h2><p>Copy...</p></div><div><picture>...<img src="SECONDARY" alt="..."></picture></div></div></div>
</div>
<div>
  <h2>Frequently asked questions</h2>
  <div class="accordion">
    <div><div>Question?</div><div>Answer.</div></div>
    <!-- repeat -->
  </div>
</div>
<div>
  <h2>CTA heading</h2><p>Closing line.</p>
  <p><a href="#">Shop ...</a> <a href="#">Discover ...</a></p>
</div>
</main><footer></footer></body>
```

After create: `DA_MCP__da_preview_content` (org aemxsc, repo frescopa26, path guides/<slug>.html). Return edit URL `https://da.live/canvas#/aemxsc/frescopa26/guides/<slug>` and preview `https://main--frescopa26--aemxsc.preview.da.live/guides/<slug>`. NEVER da_publish_content.
