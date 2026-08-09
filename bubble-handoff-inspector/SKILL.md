---
name: bubble-handoff-inspector
description: Build or upgrade a frontend Bubble handoff inspector for HTML/CSS/JS, React, Vite, Next.js, or dashboard apps. Use when a user wants a URL-gated, read-only Inspect mode with a dockable Figma-like panel, component preview, design-system link, relevance-first selected-element specs, click-to-select, visual box model, responsive evidence, accessible handoff data, CSS snippets, and downloadable assets that work with a separate Design tab and Design System.
---

# Bubble Handoff Inspector

## Purpose

Add an in-app developer handoff mode that behaves like a lightweight Figma inspector for Bubble rebuilds. It should be safe to ship in a prototype, easy to toggle, visual first, and able to inspect dynamically rendered UI.

This skill owns the read-only `Inspect` side. If the user also wants live editing, pair it with the `add-design-inspector-tab` skill. If the user wants a canonical token/page source, pair it with the `build-design-system-page` skill.

For the complete standalone clone pattern, read `references/design-inspect-system-pattern.md`.

## Workflow

1. Read the app structure and identify the frontend entry points.
   - Plain apps: usually `index.html`, `style.css`, and `app.js`.
   - React/Next apps: add a component mounted near the root layout.
2. Implement the inspector as a self-contained overlay and mark every tool-owned node with `[data-inspector-ui]`.
3. Gate the entire tool behind an explicit URL parameter such as `?inspect=true` or `?inspect=1` and lazy-load it. Do not render the launcher when the gate is absent.
4. Add a visible toggle labeled `Inspect` only inside the enabled tool.
5. On hover, preview the element under the pointer.
6. On click, lock the selected element so the user can move the cursor and read/copy details.
7. Make the side panel relevance-first.
8. Put exhaustive technical details after the useful handoff sections.
9. Add component preview and design-system link when a design-system page exists.
10. Verify the disabled URL, enabled URL, both dock sides, and the inspected product in a browser.

## Required Features

- URL gate and lazy import so production visitors do not receive a hidden launcher or unnecessary inspector code.
- Side launcher with on/off state. Let the product decide whether it starts left or right.
- Explicit left/right dock controls with clear pressed state. Persist the preference with a namespaced storage key.
- Panel that does not hide inspected content; use a drawer, docked panel, or full-width mobile shell when space is limited.
- Hover preview plus click-to-select lock state.
- `Unlock` and `Escape`.
- Prevent page clicks from triggering app navigation while inspector mode is active.
- Ignore `[data-inspector-ui]` while selecting.
- Support an explicit passthrough marker for host navigation, tutorial actions, or documentation controls that must remain clickable while inspection is active. Exclude passthrough nodes from hover, selection, editing, and page audits.
- Visual overlay for selected bounds, margin, padding, layout children, flex/grid gaps, and sibling distances.
- Relevance-first panel:
  - Component preview at top.
  - Bubble/component hint.
  - Size, selector, important layout/type/color/border facts.
  - Copy buttons.
  - Full technical details below the relevant sections.
- Component preview:
  - clone selected element.
  - remove inspector UI.
  - disable pointer events/autofocus/editing inside clone.
  - constrain the clone inside a preview host.
  - show main component label and match count if component-family info is available.
- Design System link:
  - link to `#design`, `/design`, or the app route.
  - show token matches when `brandColorTokens` exists.
- Text content:
  - show readable text when present.
  - show clicked direct text node and full selected text when they differ.
  - normalize whitespace.
- Element data:
  - selector, DOM path, parent, child count.
  - Bubble-style hint: Button, Text, Input, Dropdown, Image/icon, Group/flex, Group/grid, Form.
  - size, position, display, flex/grid settings, gap, parent gap.
  - nearest sibling distances and parent distances.
  - typography, spacing, border/radius, colors, shadow, opacity, filter, transform, transition.
  - hover, active, focus, focus-visible, and focus-within rules when readable.
  - useful attributes: id, class, data attributes, ARIA/role, href, src.
- Visual color swatches with click-to-copy HEX values.
- CSS/code snippet card with copy button.
- Assets section with thumbnails and download links for selected/descendant images, inline SVGs, and CSS background images.
- Limitations section explaining what cannot be known automatically.
- Modern computed-color support. Normalize any browser-supported color syntax, including `oklch()`, without inventing a fallback token match.
- If inspection is supported inside a same-origin device frame, use structural paths and cross-realm-safe element checks instead of parent-window `instanceof` checks.
- Never mount another inspector inside the device iframe; guard with `window.self === window.top` as well as removing recursive URL flags.

## Relationship To Design Tab

Inspect and Design may live in the same `HandoffInspector` component, but Inspect must remain read-only.

- Inspect may read `ComponentFamilyInfo` to show "N matching variants".
- Inspect may explain that the Design tab uses the same component family for Free change/Component change.
- Inspect may link to Design System.
- Inspect may not mutate styles, text, attributes, assets, layout order, or state declarations.
- Inspect asset cards are download/read-only. Design asset cards can edit.

## Information Hierarchy

The first screen of the panel should answer:

- what is this component?
- what text/content does it contain?
- what are the visible dimensions?
- what are the most important styles a Bubble developer needs next?
- where is this represented in the design system?

Prioritize by selected type:

- Text/Button/Input/Dropdown: text value, typography, colors, border/radius, spacing, states.
- Image/icon: asset preview/download, dimensions, radius/object-fit, spacing.
- Group/flex or Group/grid: layout, gap, padding, child count, distances, background, border/radius.
- Form: fields/buttons, layout, spacing, states, labels, ARIA attributes.

## Distance Measurement Rules

Measure spacing relative to layout, not only absolute X/Y.

- Find selected element parent.
- Compare selected rect to visible sibling rects in that parent.
- For each side, find the nearest sibling that overlaps on the opposite axis.
- Also show distance from selected element to parent edges.
- Draw measurement lines on the page with `px` labels.
- Make every distance value copyable.

## State Inspection Rules

Computed inactive pseudo-states are not honestly available. Inspect readable CSS rules instead.

- Search `document.styleSheets` for selectors containing `:hover`, `:active`, `:focus`, `:focus-visible`, or `:focus-within`.
- Remove the pseudo-class and test whether the base selector matches the selected element.
- Include ancestor matches when useful.
- Show state label, selector, and declarations.
- Catch and ignore cross-origin stylesheet read errors.
- Show current state classes such as `active`, `selected`, `disabled`, `open`, `expanded`, `loading`, `error`, `success`.

## Color Rules

Convert computed `rgb(...)` and `rgba(...)` values into copy-ready HEX.

- `rgb(14, 165, 233)` -> `#0ea5e9`
- `rgba(15, 23, 42, 0.16)` -> `#0f172a29`
- Preserve `transparent`, `none`, and non-rgb values when conversion is not possible.

## Asset Detection

Detect assets in the selected node and descendants:

- `<img src>` and `currentSrc`
- inline `<svg>`
- CSS background URLs from computed `background-image`

For inline SVG downloads:

```js
const clone = svg.cloneNode(true);
clone.setAttribute('xmlns', 'http://www.w3.org/2000/svg');
const svgText = new XMLSerializer().serializeToString(clone);
const href = `data:image/svg+xml;charset=utf-8,${encodeURIComponent(svgText)}`;
```

## Implementation Notes

- Use `document.elementsFromPoint(clientX, clientY)`.
- Use `window.getComputedStyle(element)` and `element.getBoundingClientRect()`.
- Use `requestAnimationFrame` to throttle overlay updates.
- Use `navigator.clipboard.writeText(...)` with visible `Copied` state.
- Keep z-index very high and overlay guides `pointer-events: none`.
- Keep implementation scoped and removable.
- Keep the panel UI isolated from host shortcuts. Let React/input handlers run, then stop non-Escape keyboard events before they reach host `window` listeners.
- Use local draft state for fields that can be refreshed from live measurements. Do not rewrite an actively edited value from a new snapshot.
- Honor `prefers-reduced-motion` for panel, section, hover, and status animations.
- Before drawing an overlay, verify the selected element is still connected. Routes, slides, and conditional UI can replace it.

## Programmatic Integration

When a product guide or documentation page needs “Try it” actions:

- Export a namespaced custom event with optional selector, device, and panel-section fields.
- Give panel sections stable slug ids and observable open state.
- Open the tool first, then resolve selection and section focus after the shell mounts.
- From a same-origin frame, dispatch the request to the parent window.
- Temporarily spotlight the requested section and honor reduced motion.
- Keep the integration optional; normal hover/click inspection must work without it.

## Browse And Review Notes

- Add Browse mode when the inspector must remain open while the user navigates the real product. It should remove selection listeners and overlays without clearing selection or handoff work.
- When comments are requested, persist them by pathname and structural element path, display synchronized pins/counts, and include grouped notes in handoff output.
- Treat missing elements as expected after responsive or conditional rendering; keep the note but omit its pin until the element resolves again.

## Portable React Delivery

For a reusable React package, keep the host integration zero-config and SSR-safe:

- Expose a single `HandoffInspector` component and an optional `enabled` override. Default to an exact opt-in URL such as `?designmode=true`.
- Read the URL after mount to avoid server/client hydration disagreement. Subscribe to `popstate` for client navigation.
- Guard every pre-mount `window`, `document`, and storage read. Storage failures in private/sandboxed contexts must degrade safely.
- Inject the tool stylesheet once, only after enablement, using a stable style id. A generated TypeScript CSS-string module avoids requiring the consumer's bundler to understand CSS imports.
- Keep React/ReactDOM as peer dependencies and mark Next.js/App Router usage as client-only.
- Keep Node-only static token loaders in an explicit subpath so `node:fs` never enters the browser bundle.

## Verification

- Toggle opens the panel.
- Without the URL gate, neither the launcher nor the panel is rendered.
- With the URL gate, the inspector is lazy-loaded and usable.
- Left/right docking works and survives reload.
- Hover preview updates.
- Click locks selection.
- Moving cursor after lock does not change panel data.
- Component preview renders and does not receive pointer events.
- Design System link opens the reference page.
- Token match chips appear for known design-system colors.
- Text appears and copies for text, buttons, headings, labels, inputs, and cards.
- Distance rows and measurement lines render.
- Border color/radius rows render and copy.
- State rules render when matching CSS exists.
- Swatches render with HEX values.
- Assets render with thumbnails and download links.
- Inspect remains read-only.
- Inputs in the tool do not trigger host-page keyboard shortcuts.
- `oklch()`, rgb/rgba, short/long hex, alpha, and transparent colors do not produce false token matches.
- Mobile panel remains usable or is intentionally disabled with a clear breakpoint.
- Passthrough controls remain interactive and cannot be selected.
- Framed pages do not recursively mount the inspector.
- A disconnected selected node leaves no stale overlay.
- Programmatic open/select/device/section requests work when the host exposes them.
- Browse mode restores real page interaction without losing review state.
- Packaged delivery renders nothing and injects no CSS/listeners while disabled, does not crash SSR, and responds to client URL changes.
