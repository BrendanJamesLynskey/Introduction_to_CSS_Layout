# Introduction to CSS Layout

---

## Slide 01 — Title

# Introduction to CSS Layout

**Box Model, Flexbox, Grid & Modern Responsive Design**

box model · flexbox · grid · responsive · container queries · custom properties

---

## Slide 02 — Agenda

### Foundations
- The CSS Box Model
- Display property & formatting contexts
- Positioning & stacking contexts

### Flexbox
- Flex container & direction
- Item properties: grow, shrink, basis
- Common flexbox patterns

### CSS Grid
- Grid fundamentals & the fr unit
- Grid placement & named areas
- Dashboard & magazine layouts

### Modern Techniques
- Responsive design & media queries
- Container queries, clamp(), fluid type
- Custom properties, logical properties
- Layout patterns, debugging & performance

---

## Slide 03 — The Box Model

Every element in CSS is a rectangular box. The box model defines how **content**, **padding**, **border**, and **margin** interact to determine the element's total size.

### content-box (default)

`width` and `height` set the content area only. Padding and border are added on top.

```css
.box {
  width: 200px;
  padding: 20px;
  border: 2px solid;
  /* Total width: 200 + 40 + 4 = 244px */
}
```

### border-box (recommended)

`width` and `height` include padding and border. Much easier to reason about.

```css
*, *::before, *::after {
  box-sizing: border-box;
}
.box {
  width: 200px;
  padding: 20px;
  border: 2px solid;
  /* Total width: 200px (content shrinks) */
}
```

### Margin Collapsing

Adjacent vertical margins collapse to the larger value. A 20px bottom margin meeting a 30px top margin produces a 30px gap, not 50px. This only happens in normal flow — flexbox and grid do *not* collapse margins.

---

## Slide 04 — The Display Property

The `display` property controls how an element participates in layout. It sets both the **outer display type** (how it flows with siblings) and the **inner display type** (how children are laid out).

| Value | Outer | Inner | Behaviour |
|-------|-------|-------|-----------|
| `block` | Block | Flow | Full width, starts on new line. `<div>`, `<p>`, `<h1>` |
| `inline` | Inline | Flow | Sits in text flow, ignores width/height. `<span>`, `<a>` |
| `inline-block` | Inline | Flow | Inline flow but respects width/height/padding/margin |
| `flex` | Block | Flex | Block-level flex container — children become flex items |
| `inline-flex` | Inline | Flex | Inline-level flex container |
| `grid` | Block | Grid | Block-level grid container |
| `none` | — | — | Removed from layout entirely (not rendered, not accessible) |
| `contents` | — | — | Box itself disappears; children promoted to parent's layout |

### Two-Value Syntax (modern)

```css
/* Equivalent to display: flex */
display: block flex;

/* Equivalent to display: inline-flex */
display: inline flex;
```

### display: contents

```css
.wrapper { display: grid; grid-template-columns: 1fr 1fr; }
/* Children of .inner become grid items directly */
.inner   { display: contents; }
```

Useful for semantic wrappers that shouldn't affect grid/flex layout.

---

## Slide 05 — Positioning & Stacking Contexts

| Position | Behaviour |
|----------|-----------|
| `static` | Default. Normal flow. `top`/`left` have no effect. |
| `relative` | Offset from its normal position. Space is preserved. |
| `absolute` | Removed from flow. Positioned relative to nearest positioned ancestor. |
| `fixed` | Removed from flow. Positioned relative to the viewport. |
| `sticky` | Toggles between relative and fixed at a scroll threshold. |

```css
/* Sticky header */
.header {
  position: sticky;
  top: 0;
  z-index: 100;
  background: var(--bg);
}

/* Absolute badge on a card */
.card { position: relative; }
.card .badge {
  position: absolute;
  top: -8px; right: -8px;
}
```

### Stacking Contexts

A stacking context is a 3D conceptual layer. Elements within one context are painted together. `z-index` only works within the same stacking context.

**What Creates a Stacking Context?**
- Root `<html>` element
- `position: absolute/relative/fixed/sticky` + `z-index` other than `auto`
- `opacity` less than 1
- `transform`, `filter`, `perspective`, `clip-path`
- `isolation: isolate`
- `will-change` with certain values
- Flex/grid children with `z-index` other than `auto`

**Common Gotcha:** A `z-index: 9999` inside one stacking context can never appear above an element in a *higher* sibling stacking context with `z-index: 1`.

---

## Slide 06 — Flexbox Fundamentals

Flexbox is a **one-dimensional** layout model for distributing space along a single axis. Set `display: flex` on the container; children become flex items automatically.

```css
.container {
  display: flex;
  flex-direction: row;        /* row | column */
  flex-wrap: wrap;            /* nowrap | wrap */
  justify-content: center;    /* main axis */
  align-items: stretch;       /* cross axis */
  gap: 1rem;                  /* between items */
}
```

| Property | Values |
|----------|--------|
| `justify-content` | `flex-start` \| `center` \| `flex-end` \| `space-between` \| `space-around` \| `space-evenly` |
| `align-items` | `stretch` \| `flex-start` \| `center` \| `flex-end` \| `baseline` |
| `align-content` | Same as justify-content (multi-line only) |
| `flex-wrap` | `nowrap` \| `wrap` \| `wrap-reverse` |
| `gap` | Shorthand for `row-gap` `column-gap` |

### The flex Shorthand

`flex: <grow> <shrink> <basis>` — e.g. `flex: 1 1 0%` means grow equally, shrink equally, start from zero width. The shorthand `flex: 1` expands to `flex: 1 1 0%`. Default is `flex: 0 1 auto`.

---

## Slide 07 — Flexbox Item Properties

### flex-grow

How much an item should grow relative to siblings when there is *extra* space.

```css
.sidebar  { flex-grow: 0; } /* fixed */
.content  { flex-grow: 1; } /* takes all space */
```

### flex-shrink

How much an item should shrink when there is *not enough* space. Default is 1.

```css
.logo { flex-shrink: 0; } /* never shrink */
.nav  { flex-shrink: 1; } /* can shrink */
```

### flex-basis

The initial size before growing/shrinking. Can be a length, percentage, or `auto`.

```css
.col { flex-basis: 300px; }
/* Starts at 300px, then grows/shrinks */
```

### order

Controls visual order without changing DOM order. Default is 0. Lower numbers come first.

```css
.first  { order: -1; } /* moves to start */
.last   { order: 99; } /* moves to end */
```

### align-self

Overrides the container's `align-items` for a single item.

```css
.container { align-items: flex-start; }
.special   { align-self: center; }
```

### How flex: 1 Distribution Works

With three children each set to `flex: 1`, available space is divided **equally** (each gets 1/3). Set one child to `flex: 2` and it receives 2/4 of the space while the others get 1/4 each. The ratio is `flex-grow / sum(all flex-grow values)`.

---

## Slide 08 — Flexbox Patterns

### Navbar

```css
.nav {
  display: flex;
  align-items: center;
  gap: 1rem;
}
.nav .logo { margin-right: auto; }
/* Logo left, links pushed right */
```

### Card Row (Equal Height)

```css
.cards {
  display: flex;
  gap: 1.5rem;
}
.card {
  flex: 1;
  display: flex;
  flex-direction: column;
}
.card .body { flex: 1; }
/* Footer aligns to bottom of all cards */
```

### Perfect Centering

```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
/* Or the one-liner: */
.center { display: grid; place-items: center; }
```

### Sticky Footer

```css
body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
main { flex: 1; }
/* Footer sticks to bottom even if content is short */
```

### Holy Grail Layout

```css
.page    { display: flex; flex-direction: column; min-height: 100vh; }
.middle  { display: flex; flex: 1; }
.sidebar { flex: 0 0 250px; }
.content { flex: 1; }
.aside   { flex: 0 0 200px; }
/* header / [sidebar | content | aside] / footer */
```

---

## Slide 09 — CSS Grid Fundamentals

CSS Grid is a **two-dimensional** layout system. It controls rows and columns simultaneously, making it ideal for page-level layouts and complex component arrangements.

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
}

/* repeat() shorthand */
.grid-4 {
  grid-template-columns: repeat(4, 1fr);
}

/* Responsive without media queries */
.auto-grid {
  grid-template-columns:
    repeat(auto-fill, minmax(250px, 1fr));
}
```

| Concept | Description |
|---------|-------------|
| `fr` | Fraction unit — distributes remaining space proportionally |
| `repeat()` | Shorthand for repeating track definitions |
| `minmax()` | Sets a min and max size for a track: `minmax(200px, 1fr)` |
| `auto-fill` | Creates as many tracks as fit, collapsing empty ones |
| `auto-fit` | Like auto-fill but stretches items to fill remaining space |
| `gap` | Shorthand for `row-gap` and `column-gap` |

### auto-fill vs auto-fit

```css
/* auto-fill: keeps empty tracks */
repeat(auto-fill, minmax(200px, 1fr));

/* auto-fit: collapses empty tracks, items stretch */
repeat(auto-fit, minmax(200px, 1fr));
```

### Implicit vs Explicit Tracks

**Explicit** tracks are defined in `grid-template-*`. **Implicit** tracks are auto-created for overflow items. Control their size with `grid-auto-rows` and `grid-auto-columns`.

---

## Slide 10 — Grid Placement & Named Areas

### Line-Based Placement

```css
.header {
  grid-column: 1 / -1;     /* span full width */
  grid-row: 1;
}
.sidebar {
  grid-column: 1 / 2;
  grid-row: 2 / 4;         /* span 2 rows */
}
.feature {
  grid-column: 2 / span 2; /* start at 2, span 2 */
}
```

### Named Grid Areas

```css
.layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header"
    "sidebar content"
    "footer  footer";
  gap: 1rem;
  min-height: 100vh;
}
.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.footer  { grid-area: footer; }
```

### Named Lines

```css
grid-template-columns:
  [sidebar-start] 250px
  [sidebar-end content-start] 1fr
  [content-end];

.item { grid-column: content-start / content-end; }
```

### Auto Flow

```css
.grid {
  grid-auto-flow: dense;
  /* Fills gaps by reordering items */
}
/* Great for image galleries and masonry-like layouts */
```

---

## Slide 11 — Grid Patterns

### Dashboard Layout

```css
.dashboard {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: auto auto 1fr;
  gap: 1rem;
}
.stat-card  { /* auto-placed, 1x1 each */ }
.chart-wide { grid-column: span 2; }
.chart-tall { grid-row: span 2; }
.full-width { grid-column: 1 / -1; }
```

### Magazine Layout

```css
.magazine {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  grid-auto-rows: 200px;
  grid-auto-flow: dense;
  gap: .5rem;
}
.feature { grid-column: span 3; grid-row: span 2; }
.medium  { grid-column: span 2; }
.small   { /* 1x1 default */ }
```

### Responsive Grid (No Media Queries)

```css
.auto-grid {
  display: grid;
  grid-template-columns:
    repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
}
/* Automatically adjusts columns:
   1 col on mobile, 2 on tablet, 3-4 on desktop */
```

### Subgrid

```css
.grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem; }

.card {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;
}
/* Card children align to the parent grid's row tracks */
```

Subgrid lets nested elements align to the parent grid. Supported in all modern browsers since late 2023.

---

## Slide 12 — Responsive Design

Responsive design ensures layouts adapt to any screen size. The **mobile-first** approach starts with the smallest screen and layers on complexity with `min-width` media queries.

```css
/* Mobile-first base styles */
.grid { display: grid; gap: 1rem; }

/* Tablet: 768px+ */
@media (min-width: 48em) {
  .grid { grid-template-columns: 1fr 1fr; }
}

/* Desktop: 1024px+ */
@media (min-width: 64em) {
  .grid { grid-template-columns: repeat(3, 1fr); }
}

/* Large: 1280px+ */
@media (min-width: 80em) {
  .grid { grid-template-columns: repeat(4, 1fr); }
}
```

### Common Breakpoints

| Name | em | px |
|------|----|----|
| Small | `30em` | 480px |
| Medium | `48em` | 768px |
| Large | `64em` | 1024px |
| XL | `80em` | 1280px |

### Viewport Units

- `vw / vh` — 1% of viewport width/height
- `dvh` — dynamic viewport height (accounts for mobile address bar)
- `svh / lvh` — smallest / largest viewport height
- `vi / vb` — 1% of viewport inline/block size

### Use em in Media Queries

Using `em` instead of `px` in media queries respects user font-size preferences. If a user has zoomed text to 150%, `48em` fires earlier than `768px` would, giving them the stacked layout they need.

---

## Slide 13 — Modern Responsive Techniques

### Container Queries

Style elements based on their *container's* size, not the viewport. True component-level responsiveness.

```css
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card { display: flex; gap: 1rem; }
  .card img { width: 40%; }
}

@container card (max-width: 399px) {
  .card img { width: 100%; }
}
```

### clamp() & Fluid Typography

Set a responsive value with a minimum, preferred, and maximum.

```css
h1 {
  /* Min 1.5rem, scales with viewport, max 3rem */
  font-size: clamp(1.5rem, 4vw + 0.5rem, 3rem);
}

.container {
  /* Fluid width: 300px to 1200px */
  width: clamp(300px, 90%, 1200px);
  margin-inline: auto;
}

.gap {
  gap: clamp(0.5rem, 2vw, 2rem);
}
```

### aspect-ratio

```css
.video { aspect-ratio: 16 / 9; width: 100%; }
.square { aspect-ratio: 1; }
/* No more padding-bottom hacks! */
```

### Container Query Units

- `cqw / cqh` — 1% of container width/height
- `cqi / cqb` — 1% of container inline/block size
- `cqmin / cqmax` — smaller/larger of cqi and cqb

```css
@container (min-width: 500px) {
  .title { font-size: 3cqi; }
}
```

---

## Slide 14 — CSS Custom Properties (Variables)

Custom properties are **live, cascading variables** defined with `--name` and accessed with `var(--name)`. Unlike preprocessor variables, they cascade, inherit, and can be changed at runtime with JavaScript.

```css
:root {
  --color-primary: #3b82f6;
  --color-surface: #ffffff;
  --color-text:    #1a1a2e;
  --radius:        8px;
  --spacing:       1rem;
}

.card {
  background: var(--color-surface);
  color: var(--color-text);
  border-radius: var(--radius);
  padding: var(--spacing);
}

/* Fallback value */
.alt { color: var(--accent, #f59e0b); }
```

### Scoped Overrides

```css
.sidebar { --spacing: 0.5rem; }
/* All children of .sidebar that use var(--spacing) get 0.5rem instead */
```

### Dark Mode Theming

```css
:root {
  --bg: #ffffff;
  --text: #1a1a2e;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #0a0a0f;
    --text: #e8e8f0;
  }
}

/* Or toggle via class */
.dark {
  --bg: #0a0a0f;
  --text: #e8e8f0;
}
```

### JavaScript Integration

```javascript
// Read a custom property
getComputedStyle(el)
  .getPropertyValue('--color-primary');

// Set a custom property
el.style.setProperty('--color-primary', '#ef4444');

// Theme toggle
document.documentElement.classList.toggle('dark');
```

---

## Slide 15 — Logical Properties

Logical properties replace physical directions (`left`, `right`, `top`, `bottom`) with **inline** (text direction) and **block** (perpendicular) axes. Essential for **internationalisation** and writing-mode independence.

| Physical | Logical |
|----------|---------|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `margin-top` | `margin-block-start` |
| `margin-bottom` | `margin-block-end` |
| `padding-left/right` | `padding-inline` |
| `width` | `inline-size` |
| `height` | `block-size` |
| `text-align: left` | `text-align: start` |
| `border-left` | `border-inline-start` |
| `top / bottom` | `inset-block` |
| `left / right` | `inset-inline` |

```css
/* Physical (LTR-only) */
.card {
  margin-left: 1rem;
  padding-left: 1.5rem;
  border-left: 3px solid blue;
  text-align: left;
}

/* Logical (works in any direction) */
.card {
  margin-inline-start: 1rem;
  padding-inline-start: 1.5rem;
  border-inline-start: 3px solid blue;
  text-align: start;
}

/* Shorthand */
.box {
  margin-inline: 1rem 2rem; /* start end */
  padding-block: 0.5rem;    /* both */
  inset-inline: 0;          /* left & right: 0 */
}
```

### Why Bother?

In RTL languages (Arabic, Hebrew), `margin-left` is on the *wrong side*. With logical properties, `margin-inline-start` is always the "beginning" side regardless of direction. Set `dir="rtl"` and the layout flips automatically.

---

## Slide 16 — Common Layout Patterns

### Sidebar + Content

```css
.layout {
  display: grid;
  grid-template-columns: minmax(200px, 25%) 1fr;
  gap: 1.5rem;
}

/* Collapses on small screens */
@media (max-width: 48em) {
  .layout { grid-template-columns: 1fr; }
}
```

### Split Screen

```css
.split {
  display: grid;
  grid-template-columns: 1fr 1fr;
  min-height: 100vh;
}
.split > * {
  display: grid;
  place-items: center;
  padding: 2rem;
}
```

### Card Grid

```css
.cards {
  display: grid;
  grid-template-columns:
    repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
}
.card {
  display: flex;
  flex-direction: column;
}
.card-body { flex: 1; }
/* Cards stretch to equal height, footer aligns to bottom */
```

### Full-Bleed Layout

```css
.content {
  display: grid;
  grid-template-columns:
    1fr min(65ch, 100%) 1fr;
}
.content > * { grid-column: 2; }
.content > .full-bleed {
  grid-column: 1 / -1;
  /* Breaks out to full viewport width */
}
```

---

## Slide 17 — Layout Debugging

### DevTools Grid & Flex Overlays

- Chrome/Firefox: Elements panel → click the `grid` or `flex` badge
- Visualises track lines, gaps, areas, and item placement
- Shows named grid areas and line numbers
- Toggle multiple grid overlays simultaneously

### The Outline Hack

```css
/* Debug: see every element's box */
* {
  outline: 1px solid red !important;
}

/* More refined version */
* { outline: 1px solid rgba(255,0,0,0.2); }
* * { outline: 1px solid rgba(0,255,0,0.2); }
* * * { outline: 1px solid rgba(0,0,255,0.2); }
```

Use `outline` not `border` — outlines don't affect layout.

### Common Gotchas

- **Overflow** — horizontal scrollbar? Check for elements wider than viewport: `width: 100vw` includes scrollbar width, use `width: 100%` instead
- **Collapsing margins** — unexpected spacing? Switch to flex/grid which don't collapse
- **Flex item won't shrink** — add `min-width: 0` to override default `min-width: auto`
- **Grid blowout** — long words overflow? Add `min-width: 0` or `overflow-wrap: anywhere`
- **100vh on mobile** — use `100dvh` to account for the dynamic address bar

### Useful DevTools Tricks

- Computed tab shows resolved values and box model diagram
- Force element state (`:hover`, `:focus`) for layout testing
- Device mode for responsive testing with touch simulation
- Layout Shift Regions (Rendering tab) shows CLS in real time

---

## Slide 18 — Layout Performance

Layout is one of the most expensive steps in the rendering pipeline: **Style** → **Layout** → **Paint** → **Composite**. Minimising layout recalculations is critical for smooth 60fps rendering.

### Layout Thrashing

Reading a layout property (e.g. `offsetHeight`) then writing a style forces a **synchronous reflow**. In a loop, this is devastating.

```javascript
// BAD — layout thrashing
for (const el of elements) {
  const h = el.offsetHeight;       // read (forces layout)
  el.style.height = h * 2 + 'px'; // write (invalidates)
}

// GOOD — batch reads, then writes
const heights = elements.map(el => el.offsetHeight);
elements.forEach((el, i) => {
  el.style.height = heights[i] * 2 + 'px';
});
```

### CSS Containment

```css
.card {
  contain: layout style paint;
  /* Or the shorthand: */
  contain: strict;  /* layout + style + paint + size */
  content-visibility: auto;
  contain-intrinsic-size: 0 300px;
}
/* Browser can skip layout/paint for off-screen cards entirely */
```

### will-change

```css
/* Hint to browser: promote to own layer */
.animated { will-change: transform, opacity; }

/* Remove when animation is done!
   Each layer costs GPU memory */
```

### Composite-Only Animations

Animate only `transform` and `opacity` — they skip layout and paint entirely, running on the GPU compositor thread.

### content-visibility: auto

Skips rendering of off-screen elements. Can cut initial render time by 50%+ on long pages. Pair with `contain-intrinsic-size` to prevent layout shifts.

### Avoid Forced Reflow Triggers

Reading `offsetTop`, `scrollHeight`, `getBoundingClientRect()`, or `getComputedStyle()` after a write forces synchronous layout. Use `requestAnimationFrame` to batch.

---

## Slide 19 — Summary & Next Steps

### Key Takeaways

- Always use `box-sizing: border-box`
- Flexbox for 1D layouts (navbars, card rows, centering)
- Grid for 2D layouts (page structure, dashboards)
- `auto-fit` + `minmax()` for responsive grids without media queries
- Container queries for truly reusable components
- `clamp()` for fluid typography and spacing
- Logical properties for i18n-ready layouts
- Custom properties for theming and dark mode
- Animate only `transform` and `opacity`

### Recommended Reading

- **web.dev/learn/css** — Google's CSS course
- **Every Layout** — Heydon & Andy Bell
- **CSS: The Definitive Guide** — Eric Meyer
- **MDN Web Docs** — Layout reference

### Try These

- Rebuild a favourite site's layout using only Grid
- Create a responsive card component with container queries
- Implement a dark/light theme with custom properties
- Build a dashboard layout with named grid areas
- Convert an existing project to logical properties

**Thank you!** — Built with Reveal.js
