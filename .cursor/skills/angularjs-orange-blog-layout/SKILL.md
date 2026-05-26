---
name: angularjs-orange-blog-layout
description: Generates an AngularJS (1.x) blog/article page layout styled like an orange-accented blog example (orange top nav, breadcrumb row, large title header, two-column content + sidebar widgets, card-like sections). Use when building an AngularJS frontend page and the user asks for a similar modern blog layout, hero header, sidebar, or orange header styling.
disable-model-invocation: true
---

# AngularJS Orange Blog Layout

## Goal
Create a simple AngularJS (1.x) page that matches the screenshot’s structure and vibe:
- **Top nav**: brand/logo left, menu links across, small icon buttons on right, orange background
- **Breadcrumb row** under the nav
- **Hero/title area**: large article title and meta row (author, date, share icons) on the orange background
- **Main body**: 2-column layout
  - Left: main content with a large “feature” card/image area and article body
  - Right: sidebar widgets (Recent Posts, Blog Categories), each a white card with a small section header rule

## Default implementation (keep it simple)
Prefer a single route/page with:
- `index.html` (layout + AngularJS bootstrapping)
- `app.js` (module, controller, mock data)
- `styles.css` (all styling; no framework required)

If the repo already uses a CSS framework, it’s OK to adapt, but default to **vanilla CSS**.

## Layout rules (match the screenshot)
- Use a centered container with max width around **1100–1200px**
- Use a **2-column grid** for the body:
  - main content: ~70%
  - sidebar: ~30%
- Cards have:
  - white background
  - subtle border (`#eee`) and/or soft shadow
  - 10–16px padding
  - 10–12px border radius (subtle)
- Accent color: orange similar to the screenshot (default `#f05a3c`); keep it configurable via CSS variables.
- Typography:
  - headline very large (around 44–56px on desktop)
  - sidebar title smaller (16–18px) with a thin orange rule

## Responsive behavior
- Desktop: two columns (content + sidebar)
- <= 900px: collapse to one column, sidebar below content
- Nav: allow menu items to wrap or collapse into a simple “Menu” button (keep it minimal; no heavy JS)

## Data + bindings (AngularJS)
Provide mock data via controller:
- `vm.siteNavLinks[]`
- `vm.breadcrumb[]`
- `vm.post` (title, author, publishedDate, heroImageUrl, bodyParagraphs[])
- `vm.recentPosts[]`
- `vm.categories[]`

Use `ng-repeat` for lists and `ng-bind` for text. Keep controller as `controllerAs: 'vm'`.

## Accessibility and semantics
- Use semantic tags: `header`, `nav`, `main`, `aside`, `article`
- Ensure links are keyboard-focusable; provide visible focus styles
- Images include `alt`

## Quick start template (what to produce)
When asked to implement this layout, produce:
- A working AngularJS page with mock data rendered
- Styling that closely matches the screenshot’s **structure** (not exact brand assets)
- Minimal dependencies and clear file names

### Suggested CSS variables
Define at top of `styles.css`:
- `--brand-orange`
- `--text`
- `--muted`
- `--card-border`
- `--bg`

## Example prompts this skill should handle
- “Build an AngularJS page styled like this screenshot.”
- “Create a blog post page with a right sidebar, recent posts widget, and orange header.”
- “Make a hero header + breadcrumb layout in AngularJS, similar to this design.”

