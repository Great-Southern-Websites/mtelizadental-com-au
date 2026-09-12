# Working on this site

This is a Quarkus Roq static site. Keep it that way.

- Content lives in `content/` (front matter + Markdown or HTML), layouts and partials in
  `templates/`, styles in `public/css/main.css`, images in `public/images/`.
- Links between pages use `{=site.url('path')}`; images use `{=site.image('name.png')}`
  or `{=site.url('images/name.png')}`. Never hard-code the domain: the site is served
  under a repository path on GitHub Pages and later under a custom domain.
- `quarkus.qute.alt-expr-syntax=true` is on: Qute expressions are `{=expr}`. Wrap
  inline `<script>` and `<style>` bodies in `{| ... |}` so braces are left alone.
- Every page must render well at 360px wide. Tap targets at least 44px. Respect
  `prefers-reduced-motion`.
- When checking pages in a browser, never wait for images to load in a script:
  off-screen `loading="lazy"` images never start loading and the wait never ends.
  Check for broken images by fetching their URLs with curl instead.
- Search engines: every page has a `title` (under 60 characters) and a one-sentence
  `description` (80 to 155 characters) in its front matter; the home page also sets
  `image:` to the name of a real file inside `public/images/` (no `images/` prefix). `templates/partials/structured-data.html`
  holds the schema.org JSON-LD for the business (name, phone, address, hours, social
  links, booking action); keep it accurate and valid whenever those details change.
  `content/sitemap.xml` is generated and `public/robots.txt` is written by the elf (it
  carries the site's address); a page can opt out of the sitemap with `sitemap: false`
  in its front matter (the 404 page does). `content/404.html` also carries
  `link: /404.html`, which publishes it to the root as GitHub Pages requires;
  without it Roq writes `404/index.html`, which GitHub never serves. Keep both
  keys if you touch that page.
- Site settings are the front matter of `content/index.html` (Roq's "site data"): its
  `title` is the site title (the business name; it names the RSS feed and llms.txt too)
  and its `description` the site description. Two optional keys the layout reads:
  `analytics: { ga4: XXXXXXXXXX }` (the GA4 Measurement ID WITHOUT the `G-` prefix; the
  `{#ga4 /}` tag adds it and puts the snippet on every page) and
  `googleSiteVerification: <code>` (the Search Console HTML tag's content value). Never
  hand-write those snippets in the layout; set the key.
- `content/rss.xml` is the posts feed and `content/llms.qute.txt` plus
  `content/llms-full.qute.txt` generate `/llms.txt` for AI assistants; all three are
  generated from the site, leave them alone. A page opts out of llms.txt with
  `llmstxt: false`.
- Code blocks: fenced code in Markdown (```java, ```bash, ```xml, ```json, ```yaml,
  ```properties, ```graphql, ```markdown, ```javascript, ```css, ```sql, ```kotlin,
  ```python, ```dockerfile) is coloured by highlight.js, bundled from `web/app/main.js`
  (the mvnpm artifact `org.mvnpm:highlight.js` in `pom.xml`) and loaded by the post
  layout only through `{#bundle /}`. Colours are the `--hl-*` tokens in `main.css`. To
  support another language, import it from `highlight.js/lib/languages/<name>` and
  register it in `main.js`. Third-party front-end libraries always come from mvnpm as
  Maven dependencies, never from a CDN or a copied file.
- Contact form plugin: when `templates/partials/contact-form.html` exists, it is the
  contact form from Great Southern Websites (a web component in `web/contact/`, its own
  bundle). The contact page includes it with `{#include partials/contact-form /}`, once,
  where a visitor expects a form. Never edit the partial or `web/contact/`; the elf owns
  them. Fields and the recipient are set by the owner in their Settings, not on the site.
- Comments plugin: likewise, when `templates/partials/comments.html` exists, the post
  layout includes it with `{#include partials/comments /}` after the post body, once.
  Never edit the partial or `web/comments/`; moderation is the owner's setting.
- Table of contents: the post layout shows one (`page.tocHtml`, TOC plugin) when a post
  has three or more headings. Use `{=page.tocHtml}` in a page layout for long pages.
- Old addresses: a page's `aliases:` front matter list (paths only, for example
  `["/about-us.html", "/classes/"]`) makes the aliases plugin generate a redirect page at
  each old path. When a page moves, add its old path to its aliases; when content merges,
  the surviving page carries every old path.
- Brand colours are CSS custom properties at the top of `main.css`. Dark theme tokens
  live in the two `data-theme` blocks; if the site has no dark theme, both blocks and the
  `theme-toggle` partial are gone.
- Verify before you finish: `QUARKUS_HTTP_PORT=8765 QUARKUS_ROQ_GENERATOR_BATCH=true mvn -q -B package quarkus:run`
  must succeed and `target/roq/index.html` must exist.
- Do not add build tooling (npm, bundlers), server code, or third-party scripts beyond
  the embeds the site already relies on (maps, video, booking widgets, social feeds).
- `target/` and `node_modules/` are work folders (Maven's, and the web bundler's own
  esbuild and Deno tooling, which is over 100 MB). Both are ignored; never commit them,
  never `git add -f` them, and never commit any file over a few megabytes. The site
  only needs its pages, templates, styles and images.
- Writing style: plain Australian English, no em or en dashes, no marketing filler, no
  exclamation marks, no emoji, nothing the business did not say. The elf hands you the
  full list as STYLE.md when it asks for work.

## News posts

Posts live in `content/posts/YYYY-MM-DD-slug.md` (front matter `title`, `description`,
optional `image`; layout `post` comes from the collection config). The listing page is
`content/posts.html`, paginated ten to a page (`/posts/`, `/posts/page2/`, ...):

```
---
title: "News"
description: "News and updates from BUSINESS NAME."
layout: default
paginate:
  collection: posts
  size: 10
---
{@io.quarkiverse.roq.frontmatter.runtime.model.NormalPage page}
{@io.quarkiverse.roq.frontmatter.runtime.model.Site site}
<section class="page-banner">
  <div class="container">
    <h1>{=page.title}</h1>
    <p class="lead">{=page.description}</p>
  </div>
</section>
<section class="section">
  <div class="container prose">
    {#for post in site.collections.posts.paginated(page.paginator)}
    <article class="change">
      <p class="muted"><time datetime="{=post.date.isoDate}">{=post.date.format('d MMMM yyyy')}</time></p>
      <h3><a href="{=post.url}">{=post.title}</a></h3>
      <p>{=post.description ?: post.contentAbstract(40)}</p>
    </article>
    {/for}
    {#include partials/pagination /}
  </div>
</section>
```

The page must be declared as `NormalPage` (pagination lives there). Post addresses
default to `/posts/<slug>/`; when the previous site used dated addresses, set
`site.collections.posts.link=/posts/:year/:month/:name/` (or the matching pattern) in
`config/application.properties` so the old addresses keep working without aliases.

The newest three on the home page: the same `{#for}` over `site.collections.posts` with
`{#if post_count < 3}` or by slicing in the template, whichever Roq version supports.
