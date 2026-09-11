---
name: blowfish
description: Build and configure Blowfish Hugo sites — homepage layouts, config files, featured SVG diagrams, shortcodes, and theme customisation.
---

# Blowfish Hugo Theme Skill

Use this skill when working with Blowfish theme projects. Covers config, layouts, SVGs, and shortcodes.

## Project Structure

```
config/_default/
├── hugo.toml          # Main Hugo config (theme, outputs, taxonomies)
├── params.toml        # Theme parameters (layout, colours, features)
├── languages.en.toml  # Language, author info, social links
├── menus.en.toml      # Header/footer/subnavigation menus
├── markup.toml        # Markdown rendering settings
└── module.toml        # Hugo module imports

content/
├── _index.md          # Homepage content (rendered inside layout)
├── blog/              # Blog posts
├── about/             # About page
└── homelab-setup/     # Project docs with featured SVGs

assets/img/            # Images referenced by config (backgrounds, author photos)
```

## Homepage Layouts (P:4 F:4)

Set `homepage.layout` in `params.toml`. Valid options:

| Layout | Description |
|--------|-------------|
| `profile` | Author image + name + links, centred |
| `hero` | Full-width background image with overlay text |
| `card` | Two-column: text left, image right |
| `background` | Background image with content overlay |
| `background-custom` | Background with custom layout partial |
| `page` | Simple content page |

Key params:

```toml
[homepage]
  layout = "card"
  homepageImage = "img/screenshot.jpg"   # Used in hero, card, background
  showRecent = true
  showRecentItems = 5
  showMoreLink = true
  showMoreLinkDest = "/blog"
  cardView = true
```

### How index.html renders homepage (P:5 F:2)

The theme's `layouts/index.html` defines the `main` block and always renders `recent-articles/main.html` **after** the layout partial:

```html
{{ define "main" }}
  {{ partial $partial . }}          <!-- your layout partial -->
  <section>
    {{ partial "recent-articles/main.html" . }}  <!-- ALWAYS rendered -->
  </section>
{{ end }}
```

**Critical rule:** Custom homepage layouts (e.g. `background-custom.html`) must **not** include `recent-articles/main.html` — the theme already renders it. Doing so causes duplicate panels.

### Custom homepage layout overrides (P:4 F:3)

Place custom layouts at `layouts/partials/home/<layout-name>.html` to override the theme's built-in layouts. Hugo resolves them via the `homepage.layout` param.

## Remote Resources & Security (P:5 F:2)

Hugo blocks `resources.GetRemote` by default. To allow it, add to `config/_default/hugo.toml`:

```toml
[security.http]
  methods = ["(?i)GET|POST"]
  urls = [".*"]
```

Without this, `resources.GetRemote` silently returns nothing.

### resources.GetRemote gotchas (P:5 F:2)

- `resources.GetRemote` returns a `Resource` object. Call `.Content` to get the HTML string.
- Do **not** use `try` with chained `.Content` — `try` wraps the result in `template.TryValue` which doesn't expose `.Content`. Assign to a variable first, then `with` that variable.
- For OG tag extraction from HTML, the attribute order in `<meta>` tags varies. Use two regex passes: one for `property=...content=...` and one for `content=...property=...`.
- For external link screenshots without build-time fetching, use client-side services like `https://image.thum.io/get/width/800/crop/400/{url}` (no API key needed).
- `urlize` is for making URL slugs (lowercase, hyphenate) — do **not** use it for URL encoding. Thum.io takes raw URLs.

## Featured SVGs for Articles (P:4 F:1)

Place `featured.svg` in the article directory. Blowfish renders it as the article thumbnail.

> **Note:** After editing SVGs, Hugo's live reload may not update the browser. Tell the user to hard refresh (`Ctrl + F5`) or use an incognito window to bypass cache.

### Reverse-S Flowchart Pattern (Optional)

Only use when the user explicitly asks for a flowchart or process flow. For simple thumbnails, skip this and use a minimal conceptual diagram instead.

For process flows, use a reverse-S layout (3 rows, curved transitions):

```
Row 1 (L→R): Step1 → Step2 → Step3 → Step4
                                           ↓ (curve)
Row 2 (R→L): Step7 ← Step6 ← Step5
                ↓ (curve)
Row 3 (L→R): Step8 → Step9 → Step10 → Done
```

### SVG Validation (P:5 F:1)

- Canvas: `viewBox="0 0 800 300"` (800 × 300 px)
- Margins: Left 200px, Right 200px, Top 50px (10px in rare cases), Bottom 50px (10px in rare cases)
- Escape `&` as `&amp;` in text (XML requirement)
- Use `viewBox` for responsive scaling
- Flowchart and title bar are optional — only add when user requests them
- Keep node width ≤ 90px and height 30px for 3-row fits
- Curved paths: `Q` (quadratic) for smooth row transitions

## Prefer Existing Theme Features (P:5 F:1)

Before adding custom layout or styling logic, inspect the theme’s existing templates, partials, parameters, and JavaScript behavior. Prefer configuring or reusing those features over duplicating them in project layouts. When an override is necessary, keep it narrow and preserve the theme’s established behavior across related page types and appearance modes.

For background and blur changes in particular, compare the homepage, list, taxonomy, and term implementations before editing. Change the layer that owns the behavior—gradient overlays in the page background partial versus scroll blur in the separate blur layer—rather than substituting one for the other.

## Taxonomy Layouts (P:4 F:1)

Taxonomy indexes such as `/tags/` and `/categories/` use the `[taxonomy]` settings, while individual terms such as `/tags/homelab/` use `[term]` in `params.toml`. To reuse a custom light/dark background hero, set both to the custom hero partial:

```toml
[taxonomy]
  heroStyle = "thumbAndBackground-custom"
  layoutBackgroundBlur = true
  layoutBackgroundBlurDark = true

[term]
  heroStyle = "thumbAndBackground-custom"
  layoutBackgroundBlur = true
  layoutBackgroundBlurDark = true
```

The shared `layouts/partials/hero/thumbAndBackground-custom.html` partial handles the `terms` and `term` scopes and switches between `defaultBackgroundImage` and `defaultBackgroundImageDark`.

## Common Config Patterns (P:3 F:1)

### Adding an external project showcase (P:5 F:3)

Use the `external-card` partial for reusable external link cards:

```go
{{ partial "external-card.html" (dict
  "url" "https://example.com"
  "title" "Project Name"
  "description" "A brief description."
  "tags" "tag1, tag2"
  "image" ""  /* optional: manual image URL, otherwise thum.io screenshot */
) }}
```

Or use the shortcode in markdown content:
```markdown
{{</* external-card
  url="https://example.com"
  title="Project Name"
  description="A brief description."
  tags="tag1, tag2"
*/>}}
```

Files:
- `layouts/partials/external-card.html` — the card partial
- `layouts/shortcodes/external-card.html` — shortcode wrapper

The partial supports `layout="vertical"` (default) and `layout="horizontal"`:

```go
{{ partial "external-card.html" (dict
  "url" "https://example.com"
  "title" "Project Name"
  "layout" "horizontal"
) }}
```

For custom horizontal card sizing, prefer explicit CSS dimensions and an explicit row/column media-query fallback. Responsive Tailwind utilities may not be present in the theme's compiled CSS, so classes such as `sm:flex-row` or `sm:w-1/3` should not be the only source of layout behavior.

### Conditional section rendering (P:3 F:2)

Wrap optional homepage sections in conditionals to avoid empty `<section>` wrappers:

```html
{{ if .Site.Params.homepage.showResearch | default false }}
<section>
  {{ partial "research-projects/main.html" . }}
</section>
{{ end }}
```

### Menu structure

```toml
[[main]]
  name = "Blog"
  pageRef = "blog"
  weight = 10

[[main]]
  name = "Projects"
  weight = 20
[[main]]
  name = "Sub Item"
  parent = "Projects"
  pageRef = "section"
  weight = 10

[[subnavigation]]
  name = "GitHub"
  pre = "github"
  url = "https://github.com/user"
  weight = 20
```

## Shortcodes (P:3 F:1)

### PDF Viewer

Renders a PDF with zoom, scroll, and page navigation controls using PDF.js.

Files:
- `layouts/partials/pdf-viewer.html` — the viewer partial
- `layouts/shortcodes/pdf-viewer.html` — shortcode wrapper

**Usage:**
```markdown
{{< pdf-viewer src="/resume/resume.pdf" >}}
{{< pdf-viewer src="/resume/resume.pdf" width="1200px" height="600px" >}}
```

**Parameters:**
| Param | Default | Description |
|-------|---------|-------------|
| `src` | (required) | Path to PDF file (place in `static/`) |
| `width` | `960px` | Viewer width |
| `height` | `800px` | Viewer height |

**Features:**
- Page navigation (Prev/Next buttons, page input)
- Zoom in/out with +/- buttons
- Fit Width / Fit Page buttons
- Scroll through all pages
- High-DPI rendering via devicePixelRatio
- Dark mode support

**Important:** Hugo escapes HTML entities in templates — use plain text labels (Prev, Next) not unicode arrows.

| Shortcode | Usage |
|-----------|-------|
| `{{< button href="URL" >}}` | Styled link button |
| `{{< github repo="user/repo" >}}` | GitHub repo card |
| `{{< figure src="img.jpg" >}}` | Image with caption |
| `{{< carousel images="{a.jpg,b.jpg}" >}}` | Image carousel |
| `{{< alert >}}` | Warning/info callout |
| `{{< mermaid >}}` | Mermaid diagram |
| `{{< youtubeLite id="ID" >}}` | Lazy YouTube embed |
| `{{< pdf-viewer src="URL" width="960px" height="800px" >}}` | PDF viewer with zoom/scroll controls |
