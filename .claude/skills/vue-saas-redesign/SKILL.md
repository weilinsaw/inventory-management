---
name: vue-saas-redesign
description: Guidelines for redesigning this app's UI from a top-nav layout into a modern SaaS-style interface with a left sidebar, a consistent spacing scale, and a polished look. Use this skill when asked to redesign, restyle, or modernize the app's layout/navigation.
---

# SaaS-Style Sidebar Redesign

Guidelines for converting this app's current top-nav layout into a left-sidebar SaaS layout. This is a **guidelines skill**: it tells you what to build and how it fits the existing design system — you (or the **vue-expert** subagent, per this repo's `CLAUDE.md` mandatory rule for any `.vue` creation/edit) do the actual implementation.

## Current State (before)

Everything lives in `client/src/App.vue`:
- `<header class="top-nav">` (template lines ~3-38): logo + horizontal `<nav class="nav-tabs">` with one `<router-link>` per view, plus `<LanguageSwitcher />` and `<ProfileMenu />` on the far right.
- `.top-nav`/`.nav-container`/`.nav-tabs` styles (style block lines ~188-270): sticky top bar, `.nav-tabs a.active` uses a bottom border accent (`::after`).
- `<FilterBar />` renders directly under the header.
- `<main class="main-content">` wraps `<router-view />`, capped at `max-width: 1600px`.
- Global design tokens are hardcoded hex values scattered through the style block (`#0f172a` dark text, `#64748b` gray, `#e2e8f0` borders, `#2563eb`/`#3b82f6` blue accent, `#f8fafc` page background) — **do not change these colors**, they're the existing design system (see root `CLAUDE.md`'s Design System section). This redesign is a layout/spacing change, not a re-theme.
- Current views (registered in `client/src/main.js`): Dashboard (`/`), Inventory, Orders, Demand, Restocking, Spending (labeled "Finance"), Reports.

## Target State (after)

Replace the horizontal top bar with a fixed-width vertical sidebar on the left; content (filters + router-view) fills the remaining width.

```
┌──────────┬─────────────────────────────────────┐
│          │  FilterBar                           │
│ Sidebar  ├─────────────────────────────────────┤
│ (fixed   │                                       │
│  width)  │  main-content (router-view)           │
│          │                                       │
└──────────┴─────────────────────────────────────┘
```

### 1. Extract `client/src/components/Sidebar.vue`

Move the logo, nav links, `LanguageSwitcher`, and `ProfileMenu` out of `App.vue`'s `<header>` into a new component:
- Logo/brand block at the top (reuse `t('nav.companyName')`/`t('nav.subtitle')`).
- `<nav>` with one `<router-link>` per route, stacked vertically. Keep `:class="{ active: $route.path === '/x' }"` — same logic, just reflowed.
- `LanguageSwitcher` and `ProfileMenu` pinned to the bottom of the sidebar (flex column with `margin-top: auto` on a wrapper around them), not floated right like today.
- Emit the same events `ProfileMenu` already emits today (`show-profile-details`, `show-tasks`) — `App.vue` still owns that modal state, just forward the events up through `Sidebar.vue`.

Active-state indicator: switch from the current bottom-border `::after` trick to a **left-border accent** (more idiomatic for vertical nav):
```css
.nav-link.active {
  color: #2563eb;
  background: #eff6ff;
}
.nav-link.active::before {
  content: '';
  position: absolute;
  left: 0; top: 0; bottom: 0;
  width: 3px;
  background: #2563eb;
}
```

### 2. Restructure `App.vue`'s layout

```vue
<div class="app-shell">
  <Sidebar @show-profile-details="showProfileDetails = true" @show-tasks="showTasks = true" />
  <div class="app-content">
    <FilterBar />
    <main class="main-content">
      <router-view />
    </main>
  </div>
</div>
```
`.app-shell` becomes `display: flex` (row) instead of the current `.app { flex-direction: column }`. `.app-content` is `flex: 1; min-width: 0; display: flex; flex-direction: column;` so `FilterBar` + `main-content` stack as they do today, just inside the right-hand column. Keep `main-content`'s existing `max-width`/padding — just drop the now-redundant top-nav-specific rules (`.top-nav`, `.nav-container`, `.logo`, `.nav-tabs`, `.subtitle`) out of `App.vue` and into `Sidebar.vue`'s scoped styles.

Sidebar width: pick one value and use it consistently — **260px** fits this app's nav-label lengths (e.g. "Demand Forecast") without wrapping. Make it sticky/full-height: `position: sticky; top: 0; height: 100vh;` (or `position: fixed` + matching `margin-left` on `.app-content` if sticky causes scroll issues with the modals' `Teleport`).

### 3. Introduce a spacing scale

Add CSS custom properties to `App.vue`'s global `:root` (or the existing `*`/`body` rule) so spacing stops being ad-hoc `rem` values sprinkled per-rule:
```css
:root {
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
}
```
Apply this scale going forward for anything you touch during the redesign (sidebar padding/gaps, `.card`/`.stat-card` padding, `.page-header` margins). **Don't do a mechanical find-replace across every existing `rem` value in the file** — that's a large, unrelated diff. Apply the scale where the redesign already requires touching a rule (new sidebar, adjusted `.main-content`/`.app-content`), and leave untouched views' existing spacing as-is unless the user asks for a full sweep.

### 4. Verify every view still renders correctly in the new shell

Each view under `client/src/views/*.vue` (`Dashboard`, `Inventory`, `Orders`, `Demand`, `Restocking`, `Spending`, `Reports`) relies on `.page-header`, `.card`, `.stats-grid`, and table styles being globally available and on `.main-content` providing the width/padding context — none of them should need changes themselves. After the layout change, spot-check each route for:
- Content not clipped or overlapping the sidebar.
- `.stats-grid`'s `repeat(auto-fit, minmax(280px, 1fr))` still produces a sensible column count at the new, narrower content width.
- Modals (`Teleport to="body"`) still render above the sidebar (they already teleport to `<body>`, so z-index should be unaffected — just confirm visually).

## Execution checklist

1. Delegate to **vue-expert** for all `.vue` work (mandatory per root `CLAUDE.md`): create `Sidebar.vue`, edit `App.vue`.
2. Create `Sidebar.vue` first, then wire it into `App.vue`, then remove the now-dead top-nav CSS from `App.vue`.
3. Use Playwright MCP against `http://localhost:3000` to visually check every route in the checklist above, plus resizing the window to confirm the sidebar doesn't break at common widths (1280px, 1440px, 1920px — this is a desktop-oriented demo app, not a target for mobile responsiveness unless asked).
4. Check `mcp__playwright__browser_console_messages` for new warnings/errors after the change.

## Pitfalls

- Don't change the color palette — this is a layout redesign, not a re-theme.
- Don't touch `server/` or any API contract — this is frontend-only.
- Don't add emojis or icon libraries not already in the project — plain text nav labels (matching the current app) or simple inline SVGs only, consistent with the "no emojis in UI" rule in root `CLAUDE.md`.
- Don't forget `ProfileMenu`'s existing emits (`show-profile-details`, `show-tasks`) need to keep flowing up to `App.vue` through `Sidebar.vue` — the modals are still mounted in `App.vue`, not moved into the sidebar.
