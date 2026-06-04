---
name: design-to-render-page
description: Use when the user provides a mobile UI design image and a sliced-asset directory, then asks Codex to implement or visually refine a render-side web page in an existing project. Covers project discovery, design asset mapping, Vue/Vant route integration, static-vs-dynamic text handling, JSON mock API data, build verification, and screenshot-level visual QA for localhost pages.
---

# Design To Render Page

Use this skill when converting a provided design image plus icons/sliced assets into an existing render-side page, especially in mobile Vue projects.

The goal is not only to make the page look close to the design, but to fit the existing project: routing, global navbar, build scripts, style conversion rules, API/mock patterns, and local visual verification.

## Required Inputs

Extract these from the user request or discover them from the repo before editing:

- Design image path.
- Icon/sliced-asset directory.
- Target route, e.g. `/agent`.
- Local preview URL, e.g. `http://127.0.0.1:8081/render/agent`.
- Text the user explicitly wants to keep static.
- Text or data the user wants driven by backend/mock data.

If a design path is wrong, search nearby directories first. Ask the user only after reasonable local discovery fails.

## Mandatory Project Discovery

Before implementation, inspect the project. Do not assume the tech stack or page structure.

1. Read `package.json`.
   - Identify the framework, UI library, style language, build scripts, and dev scripts.
   - Prefer existing dependencies over adding new ones.

2. Read the style conversion config.
   - Check `postcss.config.js`, Vite config, webpack config, or similar files.
   - If `postcss-px-to-viewport` uses `viewportWidth: 375`, write CSS against a 375 px mobile viewport.
   - If the design is 750 px wide and the viewport is 375 px, divide design pixels by 2.
   - Treat the target design canvas as `375px × 811px` unless the user explicitly provides a different viewport size.

3. Find route and layout behavior.
   - Inspect the router config for route path, component, `meta.title`, and `meta.pageName`.
   - Inspect app/layout files for a global navbar.
   - Prefer the project's unified navbar over a custom navbar inside the page component.
   - If animations or interactions are active, interactive UI components must be extracted as separate `"use client"` components.
   - Never use emoji in code, markup, text content, or alt text.
   - Normalize breakpoints. Contain the page layout with `max-w-[1400px] mx-auto` or `max-w-7xl`. Never use `h-screen` for full-height hero sections; always use `min-h-[100dvh]`.
   - Prefer CSS Grid over Flexbox math.

4. Find page and data patterns.
   - Inspect nearby page components for Vue style, naming conventions, API usage, image handling, and CSS conventions.
   - Check `src/apis`, `src/mock`, stores, and shared utilities before adding new patterns.

## Asset Handling

Use the user-provided image assets whenever possible.

- Identify semantic roles: page background, top card background, feature icons, section title art, selected/checked icon, button art.
- Copy or move needed assets into the target page asset directory according to project conventions.
- Rename copied assets semantically when useful, e.g. `hero-bg.png`, `package-panel.png`, `voice-icon.png`, `flow-icon.png`, `rule-title.png`, `check-icon.png`.
- Preserve original user assets unless the user asks to clean them up.
- Use actual image dimensions to drive CSS variables and layout values.

Dimension conversion examples when using a 750 px design in a 375 px viewport project:

- Design background `750 × 1452` → CSS `375px × 726px`.
- Design card `696 × 278` → CSS `348px × 139px`.
- Design icon `30 × 30` → CSS `15px × 15px`.

## Measurement Discipline

Use this section when converting precise mobile designs into CSS, especially when a module uses sliced assets with overlaid text/icons.

- Prefer actual sliced-asset dimensions over visible bounds measured from the design screenshot. If a card background asset is `204 × 256` in a 750 px design workflow, implement it as `102px × 128px` in a 375 px viewport project, even if the visible border in the design appears slightly smaller due to transparent pixels, anti-aliasing, or shadow edges.
- Measure spacing by box-model edges, not by visible glyph pixels. For example, a benefit row's left offset should be measured from the card background box's left edge to the benefit icon box's left edge, not to the first colored visible pixel or text glyph.
- For icon-and-text rows, explicitly separate icon position from text position: `icon left`, `icon width/height`, `icon-to-text gap`, `text font-size`, `line-height`, and each row's `top` should be defined from measured values.
- For dense repeated cards, do not rely on normal document flow to infer vertical spacing. Use fixed coordinates for title, divider, benefit rows, price pill, and selected-state corner badge when the design is fixed-format.
- When the design and asset dimensions disagree, state the discrepancy before coding. Example: the sliced asset is `204 × 256` but the visible border measured from the screenshot is approximately `196 × 252`; use the sliced asset size for `background-size` in CSS unless the user explicitly asks to crop to visible bounds.
- Before implementing complex fixed-format modules, produce a short measurement audit table when precision matters or after any visual mismatch. Include original design pixels and converted CSS pixels, covering card size, icon size, left offsets, row gaps, title top gap, price top/bottom gaps, and selected-corner badge size.
- If the user provides exact dimensions, treat them as authoritative and update the CSS directly to those values. Do not keep previously visually adjusted values that conflict with explicit dimensions.

## Implementation Rules

Implement in the project's existing style.

- Use the existing framework and component conventions.
- In Vue 3 pages, prefer the local pattern: `<script setup>`, `ref`, `computed`, and `onMounted` are appropriate when already used.
- Keep edits scoped to the target page, its route metadata, required assets, and mock/API files.
- Use provided sliced images for complex visuals instead of recreating detailed backgrounds with CSS.
- Keep fixed-format UI elements dimensionally stable: cards, icons, buttons, background panels, and horizontal lists should not resize because data changes.
- Do not add horizontal or vertical scrolling to a module unless the design clearly shows overflow/scroll behavior or the user explicitly requests it.
- If the design does not show overflow beyond the screen, fit the module's visible elements inside the `375px × 811px` canvas rather than letting content run off-screen.
- If using a global navbar, remove custom page-level navbars and adjust top padding so content does not hide beneath the global bar.
- Avoid unrelated refactors, dependency additions, or cleanup.
- The complete interaction cycle must be implemented: loading state (skeleton screen), empty state, error state, and haptic feedback.

## Data And Text Rules

Default rule: business text and variable product data should come from API/mock data unless the user explicitly says to keep it static.

Static text may include:

- User-specified labels, e.g. "Domestic Plan" and "Domestic Data".
- Units, e.g. "min" and "GB".
- Fixed interaction text, e.g. "Effective Next Month" and "Confirm".

Dynamic data should include:

- Product/package names and short names.
- Prices, price labels, badges, package tiers, and selected package data.
- Voice minutes and data allowance values.
- Rule descriptions, product introductions, fee details, toast text, and dialog copy.
- Configurable image URLs when the backend may provide them.

For mock data, prefer `src/mock/*.json` plus a thin wrapper under `src/apis/*.js`:

```js
import data from '@/mock/exampleOffer.json'

const exampleOfferApi = {
  getOfferDetail: () =>
    Promise.resolve({
      code: 200,
      data
    })
}

export default exampleOfferApi
```

Design mock JSON so the real backend can replace the wrapper without changing the page component.

## Visual Fidelity Rules

Style from the design dimensions, then refine by screenshot.

- Match the design's visible layout to a `375px × 811px` viewport by default.
- Base widths, heights, icon sizes, margins, and padding on measured image dimensions.
- Use CSS custom properties (variables) for important design dimensions.
- Use `background: url(...) top center / width height no-repeat` for fixed mobile background art.
- Use fixed icon sizes and avoid flex/image stretching.
- Keep text inside buttons/cards from overflowing or causing layout shift.
- Non-scrolling elements should be fully visible within the intended viewport; only use clipped or off-screen placement when it is visible in the design or explicitly requested.
- Add bottom padding to page content to account for fixed bottom action buttons.
- After user visual feedback, change the specific measured value rather than broadly restyling the page.

## Verification

Run verification after implementation.

1. Build verification:
   - Run the project build script, usually `npm run build`.
   - Report success or failure.
   - Distinguish pre-existing warnings from warnings introduced by this change.

2. Text verification:
   - Search the target component for hardcoded Chinese/business text.
   - Confirm only user-approved static text remains.
   - Confirm package names, prices, rules, and toast/dialog text come from JSON/API data.

3. Visual verification:
   - Use the user-specified localhost/127.0.0.1 URL when provided.
   - Check the page against the design image with screenshots at `375px × 811px` unless another viewport was specified.
   - Verify background, card size, icons, spacing, unified navbar, bottom button, and text overflow.
   - Verify that modules without explicit scroll behavior do not scroll, and that non-overflow design elements fit within one screen.

If localhost access is blocked by a browser extension, enterprise policy, or network policy, state the exact blocker and stop attempting to bypass it.

## Failure Handling

- If a design image cannot be read, search likely nearby paths and ask only if not found.
- If terminal output shows garbled characters (mojibake), do not assume file corruption; verify through build output, editor-readable files, or browser rendering.
- If several route/page candidates exist, inspect them and choose the one matching the user's target URL.
- If the existing generated page is too brittle, rewrite the target page component while preserving the route, global layout, and project conventions.
- If the user asks for a style tweak, apply the smallest targeted change and recheck visually when possible.

## Final Response

Keep the final response concise and include:

- Which page or skill-relevant files were changed.
- Whether mock/API dynamic data was added.
- Whether `npm run build` passed.
- Whether screenshot-level visual confirmation was completed or why it was blocked.
- Any important remaining caveats.
