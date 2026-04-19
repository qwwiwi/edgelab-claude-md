---
name: present
description: "Generate beautifully formatted standalone HTML documents with clean minimal design. Triggers: /present, 'make a presentation', 'visualize', 'format as HTML', 'create a document'. Accepts any content (lesson, report, comparison, roadmap, dashboard) and produces a single HTML file with light/dark theme toggle. Zero external dependencies."
---

# Present -- HTML Document Generator

## When to use
- User writes `/present` or asks to format something nicely
- Data visualization, lesson, report, plan, or comparison needed
- Request for an HTML file viewable in a browser

## Process

1. **Determine content type** -- lesson, report, comparison, roadmap, dashboard
2. **Gather content** -- from conversation, files, APIs, or generate
3. **Create HTML** following the design system below
4. **Save** as `./present-{topic}.html` (current directory, kebab-case topic)

## Design System (Notion-inspired)

### Color Tokens

**Light theme (default):**
```
--bg: #ffffff
--text: #37352f
--text-muted: #787774
--card-bg: #f7f6f3
--card-border: rgba(55,53,47,0.09)
--accent: #2eaadc
--accent-dim: rgba(46,170,220,0.1)
--code-bg: #f7f6f3
--divider: rgba(55,53,47,0.09)
--error: #eb5757
--error-dim: rgba(235,87,87,0.1)
--success: #0f7b6c
--shadow: 0 1px 3px rgba(0,0,0,0.04), 0 0 0 1px rgba(0,0,0,0.04)
```

**Dark theme:**
```
--bg: #191919
--text: #e6e6e5
--text-muted: #999999
--card-bg: #252525
--card-border: rgba(255,255,255,0.08)
--accent: #529cca
--accent-dim: rgba(82,156,202,0.15)
--code-bg: #252525
--divider: rgba(255,255,255,0.08)
--error: #eb5757
--error-dim: rgba(235,87,87,0.15)
--success: #4dab9a
--shadow: 0 1px 3px rgba(0,0,0,0.2), 0 0 0 1px rgba(255,255,255,0.04)
```

### Fonts (system stack, zero external dependencies)
- Headings: `-apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif` -- weight 600-700
- Body: same stack -- weight 400
- Code: `"SFMono-Regular", Menlo, Consolas, "Liberation Mono", monospace`

### Components

**Header (doc-header)**
- badge: uppercase, accent-dim bg, accent text, border-radius: 100px
- title: system font, 2rem, font-weight 700
- subtitle: 1rem, muted color

**Table of Contents (toc)**
- card-bg background, card-border, border-radius: 0.75rem
- Numbered list: accent-colored numbers, font-weight 600
- Hover: subtle background shift

**Section (section)**
- Title: 1.5rem, weight 600, numbered with accent color, bottom divider
- Body: 0.9375rem, line-height 1.7

**Card (card)**
- card-bg, card-border, border-radius: 0.75rem, padding: 1.25rem
- Light mode: subtle box-shadow
- card-title: weight 600

**Code (pre > code)**
- code-bg, card-border, border-radius: 0.5rem
- **MANDATORY:** `white-space: pre-wrap; word-wrap: break-word; overflow-wrap: break-word;`
  (prevents horizontal scroll on mobile)
- font-size: 0.8125rem, line-height: 1.6

**Tables (table)**
- th: accent color, uppercase, letter-spacing 0.05em, weight 600
- td: muted color, first td -- primary text color, weight 500
- Rows: bottom divider

**Callout (callout)**
- accent-dim bg, border-left: 3px solid accent
- Label: accent, uppercase, 0.75rem, weight 600

**Key Point (key-point)**
- card-bg, accent border, border-radius: 0.75rem
- Label: accent, uppercase, bold

**Error Box (error-box)**
- error-dim bg, error border (subtle)
- Title: error color, weight 600

**Summary Card (summary-card)**
- accent-dim bg, accent border, border-radius: 0.75rem
- Title: weight 700

**Metric Card (metric-card) -- for reports/dashboards**
- card-bg, border, border-radius: 0.75rem
- metric-value: 2rem, weight 700, accent color
- metric-label: muted, uppercase, 0.75rem

**Theme Toggle (theme-toggle)**
- Fixed top-right corner, 40x40px button
- Sun icon (light mode) / Moon icon (dark mode) -- pure CSS/SVG, no deps
- Respects `prefers-color-scheme` on first load
- Persists choice in `localStorage`
- Smooth transition: `transition: background 0.3s, color 0.3s, border-color 0.3s`

### Responsive
```css
@media (max-width: 640px) {
  .container { padding: 1rem 1rem 3rem; }
  .doc-title { font-size: 1.5rem; }
  .section-title { font-size: 1.25rem; }
  pre { padding: 0.75rem; font-size: 0.75rem; }
  table { font-size: 0.8125rem; }
  th, td { padding: 0.5rem; }
  .metric-grid { grid-template-columns: 1fr; }
}
```

### HTML Skeleton
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{title}</title>
<!-- CSS variables for both themes, styles, theme toggle JS -->
</head>
<body>
<button class="theme-toggle" aria-label="Toggle theme">...</button>
<div class="container">
  <div class="doc-header">
    <div class="doc-badge">{badge}</div>
    <h1 class="doc-title">{title}</h1>
    <p class="doc-subtitle">{subtitle}</p>
  </div>
  <div class="toc">...</div>
  <!-- sections -->
</div>
<script>/* theme toggle logic */</script>
</body>
</html>
```

### Theme Toggle JS Pattern
```javascript
(function() {
  const toggle = document.querySelector('.theme-toggle');
  const html = document.documentElement;
  const stored = localStorage.getItem('present-theme');
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
  const theme = stored || (prefersDark ? 'dark' : 'light');
  html.setAttribute('data-theme', theme);

  toggle.addEventListener('click', function() {
    const current = html.getAttribute('data-theme');
    const next = current === 'dark' ? 'light' : 'dark';
    html.setAttribute('data-theme', next);
    localStorage.setItem('present-theme', next);
  });
})();
```

## Content Types

### Lesson
- Badge: "Lesson -- {topic}"
- TOC with numbered sections
- Sections with code blocks, tables, callouts, key-points
- Practice/exercises near the end, summary table

### Report
- Badge: "Report -- {date}"
- Key metrics in metric-card grid (2-4 columns)
- Data tables
- Summary in summary-card

### Comparison
- Badge: "Comparison"
- Comparison table (th: parameter, columns: options)
- Pros/cons in cards
- Verdict in summary-card

### Roadmap
- Badge: "Roadmap" or "Plan"
- Phases as numbered sections
- Timeline visualization (CSS)
- Statuses: done (success), in progress (accent), planned (muted)

### Dashboard
- Badge: "Dashboard -- {topic}"
- Metrics in grid cards
- Data tables
- Trends as text descriptions or inline charts

## Rules

- **Standalone HTML:** single file, zero external dependencies (no Google Fonts, no CDN)
- **HTML escaping:** all user-supplied text MUST be HTML-escaped before insertion (`<` -> `&lt;`, `>` -> `&gt;`, `&` -> `&amp;`, `"` -> `&quot;`). Never insert raw user strings into HTML attributes or script contexts
- **Mobile-first:** responsive, max-width container
- **pre/code blocks:** ALWAYS `white-space: pre-wrap; word-wrap: break-word; overflow-wrap: break-word;`
- **Content language:** match user's language (the template/skill is in English, content adapts)
- **Theme toggle:** always present, top-right corner
- **Default theme:** respect `prefers-color-scheme`, fallback to light
- **Transitions:** smooth 0.3s on theme change for bg, color, border
- **Save path:** `./present-{topic}.html` -- relative, agent decides full path
- **Reference templates:** `assets/template-lesson.html`, `assets/template-report.html`
