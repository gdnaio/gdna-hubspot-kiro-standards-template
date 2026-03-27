# Support Portal / Knowledge Base — Design

## Templates

- `templates/pages/knowledge-base.html` — KB home with category grid and search
- `templates/pages/kb-category.html` — Category listing with article cards
- `templates/pages/kb-article.html` — Article detail with TOC and sidebar
- `templates/system/search-results.html` — Search results page

## Module Composition

### KB Home
1. `kb-hero` — Search bar with heading
2. `kb-category-grid` — Category cards with icon, title, article count

### Category Page
1. `kb-breadcrumb` — Breadcrumb navigation
2. `kb-article-list` — Article cards with title, excerpt, date

### Article Page
1. `kb-breadcrumb` — Breadcrumb navigation
2. `kb-article-content` — Article body with auto-generated TOC
3. `kb-related-articles` — Related article cards
4. `kb-contact-cta` — "Still need help?" with form or chat link

## Responsive Behavior

- Desktop: Sidebar + content area
- Tablet: Collapsible sidebar
- Mobile: Full-width content, hamburger for sidebar

## Style Approach

- Readable typography: 16px+ body, 1.6 line-height
- Max content width: 720px for article body
- Code blocks with syntax highlighting (via CSS)
- Callout boxes for tips, warnings, notes
