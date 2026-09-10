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

## Homepage Layouts

Set `homepage.layout` in `params.toml`. Valid options:

| Layout | Description |
|--------|-------------|
| `profile` | Author image + name + links, centred |
| `hero` | Full-width background image with overlay text |
| `card` | Two-column: text left, image right |
| `background` | Background image with content overlay |
| `background-custom` | Background with custom `_index.md` content |
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

## Common Config Patterns

### Adding an external project showcase

1. Set `homepage.layout = "card"` in `params.toml`
2. Set `homepageImage` to a screenshot in `assets/img/`
3. Add description and button in `content/_index.md`:
   ```markdown
   ## Project Name
   {{< button href="https://example.com" target="_blank" >}}Visit Site{{< /button >}}
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

## Shortcodes

| Shortcode | Usage |
|-----------|-------|
| `{{< button href="URL" >}}` | Styled link button |
| `{{< github repo="user/repo" >}}` | GitHub repo card |
| `{{< figure src="img.jpg" >}}` | Image with caption |
| `{{< carousel images="{a.jpg,b.jpg}" >}}` | Image carousel |
| `{{< alert >}}` | Warning/info callout |
| `{{< mermaid >}}` | Mermaid diagram |
| `{{< youtubeLite id="ID" >}}` | Lazy YouTube embed |
