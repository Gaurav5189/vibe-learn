---
name: cross-document-transitions
description: Use this skill when a user wants animated page transitions between separate HTML files — slides, fades, or shared element morphs. Triggers: "page transitions," "app-like navigation," "smooth page change," "card-to-detail animation." Do NOT use for SPAs (React Router, Vue Router) or same-page animations.
---

# CSS Cross-Document View Transitions — Design Guide

## Overview

The CSS View Transitions API enables smooth, app-like animations when a user navigates between separate HTML pages on the same domain — with zero JavaScript. Before this API existed, achieving cross-page animations required either a single-page application framework or complex JavaScript hacks (preloading the next page in a hidden iframe, then swapping DOM). The View Transitions API solves this natively at the browser level.

---

## Architecture Constraints

| Constraint | Detail |
|---|---|
| **Domain** | Origin and destination must share the exact same domain |
| **Stylesheet** | Both pages must import the identical shared CSS file |
| **Browser Support** | Chrome, Edge, Safari — Firefox degrades to instant page load (no polyfill needed) |

---

## Base Implementation

Wrap all view transitions in a `prefers-reduced-motion` media query — mandatory for accessibility.

```css
@media (prefers-reduced-motion: no-preference) {
  @view-transition {
    navigation: auto;
  }
}
```

This enables the default crossfade. No JavaScript required.

---

## Custom Page Animations

Define `@keyframes` and map them to transition pseudo-elements:

| Pseudo-element | Targets | Use for |
|---|---|---|
| `::view-transition-group(root)` | Shared properties | `animation-duration`, `animation-timing-function` |
| `::view-transition-old(root)` | Leaving page snapshot | Outgoing `animation-name` |
| `::view-transition-new(root)` | Incoming page | Incoming `animation-name` |

**Example — Slide transition:**

```css
@media (prefers-reduced-motion: no-preference) {
  @view-transition {
    navigation: auto;
  }

  @keyframes slide-out-left {
    to { transform: translateX(-100%); }
  }

  @keyframes slide-in-right {
    from { transform: translateX(100%); }
  }

  ::view-transition-group(root) {
    animation-duration: 300ms;
    animation-timing-function: ease-in-out;
  }

  ::view-transition-old(root) {
    animation-name: slide-out-left;
  }

  ::view-transition-new(root) {
    animation-name: slide-in-right;
  }
}
```

---

## Isolating Elements (e.g., Fixed Navigation)

To keep elements like a nav bar static while content animates:

**Step 1** — Name the moving container:

```css
main {
  view-transition-name: page-content;
}
```

**Step 2** — Target that name instead of `root`:

```css
::view-transition-group(page-content) {
  animation-duration: 300ms;
  animation-timing-function: ease-in-out;
}

::view-transition-old(page-content) {
  animation-name: slide-out-left;
}

::view-transition-new(page-content) {
  animation-name: slide-in-right;
}
```

The nav bar receives no `view-transition-name` and remains fixed throughout.

---

## Shared Element Transitions (Morphing)

Creates seamless morph effects where an element on page A flows into its counterpart on page B.

**Rules:**
1. Target the element on **both** pages in the shared CSS file
2. Assign **identical** `view-transition-name` values to both elements
3. **Uniqueness is strictly enforced** — every `view-transition-name` must be unique within the current page DOM

```css
/* Both pages include this shared CSS */
.card-image,
.hero-image {
  view-transition-name: article-image-unique-id;
}
```

### Generating Unique Names for Dynamic Lists

For lists or card grids, names must be dynamically scoped — never reused:

```css
/* ❌ Wrong — collides when multiple cards exist */
.card-image {
  view-transition-name: card-image;
}

/* ✅ Correct — unique per item */
.card[data-id="1"] .card-image { view-transition-name: card-image-1; }
.card[data-id="2"] .card-image { view-transition-name: card-image-2; }
.card[data-id="3"] .card-image { view-transition-name: card-image-3; }
```

Or inline via templating:

```html
<div class="card" style="--img-vt-name: card-image-{{ item.id }}">
  <img style="view-transition-name: var(--img-vt-name)" />
</div>
```

---

## Quick Reference

```
Shared CSS file
├── @view-transition { navigation: auto; }          ← enables cross-doc transitions
├── @keyframes (enter/exit animations)
├── ::view-transition-group(name)                   ← timing + easing
├── ::view-transition-old(name)                     ← outgoing page
├── ::view-transition-new(name)                     ← incoming page
└── element { view-transition-name: unique-id; }   ← shared element morph
```

**Invariants:**
- All rules inside `@media (prefers-reduced-motion: no-preference)`
- One `view-transition-name` per unique DOM node
- Same CSS file imported on every participating page
- Same domain for all participating pages
