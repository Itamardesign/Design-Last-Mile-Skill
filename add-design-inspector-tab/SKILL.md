---
name: add-design-inspector-tab
description: Add or upgrade a designer-only Design tab beside a read-only Inspect/handoff panel in React, Vite, Next.js, HTML/CSS/JS, or similar frontend apps. Use when the user wants a Figma-like in-app design mode with reliable numeric/text inputs, direct canvas resize and in-place text editing, Free vs Component editing, live assets and states, device previews and breakpoint sweeps, token binding and audits, accessibility checks, persistent reversible sessions, embedded walkthrough actions, and handoff output connected to a Design System.
---

# Add Design Inspector Tab

## Purpose

Build the designer side of a two-surface inspection system:

- `Inspect` is for developers and handoff. It must stay read-only.
- `Design` is for designers. It may edit the live preview without requiring source-code changes.
- `DesignSystem` is the canonical token/reference page that Inspect links to and Design borrows tokens from.

Prefer upgrading an existing inspector component over adding an unrelated overlay. Keep the implementation scoped and removable.

For the complete standalone clone pattern, read `references/design-inspect-system-pattern.md`.

## Workflow

1. Find the existing inspector, overlay, or handoff component.
   - Common names: `HandoffInspector`, `Inspector`, `DesignMode`, `DevTools`, `InspectOverlay`.
   - In React apps, mount the inspector once near the root layout.
2. Preserve `Inspect` as read-only.
   - Do not pass edit callbacks into Inspect details.
   - Keep CSS, DOM, text, states, assets, and component preview copyable/downloadable.
3. Add or upgrade `Design` as the only editing surface.
4. Reuse the same selected element, snapshot, overlay, and component-family data for Inspect and Design.
5. Add a persistent scope toggle:
   - `Free change`: edit only the selected part.
   - `Component change`: edit all matching variants on the current page.
6. Detect the component family before edits.
7. Classify the selected element before rendering controls.
8. Store original element state before the first edit to each target.
9. Add reset, persistent change log, CSS diff, implementation instructions, and non-CSS notes.
10. Add responsive, token-governance, component-state, and accessibility tools when they are in scope.
11. Verify with TypeScript/build and real browser interactions.

## Required Data Model

Use these concepts even if names differ:

- `ElementSnapshot`: read-only snapshot of DOM, styles, text, geometry, states, colors, assets, and distances.
- `uniquePath`: structural DOM locator that can resolve the equivalent element after reload or inside a same-origin device frame.
- `DesignOriginalState`: original live DOM state for reset. Store style attribute, text/value, innerHTML, all attributes needed for assets, computed values, parent, and next sibling.
- `ComponentFamilyInfo`: label, selector/reason, matching elements, and match count.
- `DesignChanges`: map of selector to property/value changes.
- `AssetReplacement`: `{ type: 'url' | 'svg', value: string, label: string }`.
- `StoredSession`: pathname-scoped replayable changes, generated CSS variables, and timestamp. Persist only changes that can be replayed safely.

## Mode Separation

- Inspect details may render component previews, asset cards, token matches, state rules, and copy buttons.
- Inspect details must not call style/text/asset/attribute mutation functions.
- Design mode receives callbacks such as `onStyleChange`, `onBulkStyleChange`, `onTextChange`, `onAttributeChange`, `onAssetChange`, `onResetElement`, `onResetAll`, and `onReorder`.
- If a previous Character/Voice/Popup tab exists, move that content into the design system or sandbox when the user asks for a cleaner split. Keep the primary inspector modes to `inspect` and `design`.

## Selection

- Use `document.elementsFromPoint(clientX, clientY)` and ignore `[data-inspector-ui]`.
- On hover, preview the target.
- On click, lock the selected element and prevent app navigation.
- Let users select nested children inside selected parents.
- The selected rectangle must be `pointer-events: none`.
- Only resize handles and explicit drag/reorder controls should use `pointer-events: auto`.
- Provide `Unlock` and `Escape` behavior.
- Refresh the snapshot after scroll, resize, and edits.

## Component Scope Toggle

The Design tab should always make scope visible:

- `Free change`: "Property edits apply only to the selected part."
- `Component change`: "Property edits apply to N matching [component] variants on this page."

When Component change is active:

- Collect targets through `getCurrentDesignTargets`.
- Use `ComponentFamilyInfo.elements`.
- Keep selected element first.
- Store originals for every target before mutation.
- Commit the same property/value change to every matching selector.

## Component Matching

Use a conservative matching ladder:

1. explicit markers: `data-component`, `data-design-component`, `data-ui`, `data-testid`, `data-test`
2. tag + stable comparable classes
3. tag + role
4. similar tag, class similarity, and child-structure signature

Always filter out inspector UI, invisible elements, and different component kinds.

## Adaptive Designer Controls

Classify elements into practical product types:

- Text: content, font family, size dropdown, weight dropdown, line height, letter spacing, align, color, typography presets.
- Button/link: text/href, colors, border, radius, padding, shadow, hover/focus/active states.
- Image: preview, src URL, file replacement, alt text, object-fit, object-position, size, radius.
- Input/textarea: value, placeholder, ARIA label, typography, surface, focus/error states.
- Layout/group: display, flex/grid settings, gap, padding, background, border/radius, child reorder.
- Generic: size, spacing, surface, opacity, transform, assets if present.

Use familiar controls: swatches, color inputs, selects, segmented controls, numeric drag labels, icon buttons, and compact panels.

## Text And Numeric Values

- Make font size a dropdown of known values such as `12px`, `13px`, `14px`, `15px`, `16px`, `18px`, `20px`, `24px`, `28px`, `32px`, `40px`, `48px`, `56px`, `64px`.
- Normalize bare numbers to `px` for length properties such as `font-size`, `width`, `height`, `padding`, `margin`, `gap`, `border-width`, `border-radius`, and `letter-spacing`.
- Preserve unitless values where appropriate, such as normal line-height below common pixel thresholds.
- Never commit width, height, spacing, opacity, or scale on every keystroke while the live snapshot is refreshing.
- Give numeric inputs a local draft. Sync from the measured value only while unfocused; commit on blur/Enter; cancel on Escape; commit arrow-key steps; clamp and round once.
- Give textareas a local raw-text draft so refreshes do not eat repeated or trailing spaces. Warn before replacing nested markup with plain text.
- Stop tool-owned keyboard events before they reach host-page shortcuts, while preserving Escape behavior.

## Direct Canvas Editing

When the user wants Figma-like direct manipulation, add an explicit, persisted canvas-edit toggle. Keep panel controls as the reliable fallback.

- Draw eight accessible resize handles inside the existing selection box. Keep handles interactive while the selection overlay itself remains pointer-transparent.
- Preview the drag live, but rollback the temporary inline styles before committing through the normal style/change-log pipeline.
- Scale pointer deltas by device-preview zoom. Mirror the temporary resize into the equivalent frame node.
- `Shift` on a corner preserves the starting aspect ratio. `Alt` snaps to a base step imported from the spacing-token scale. `Escape` and `pointercancel` restore the exact pre-drag inline values.
- For left/top handles, adjust the relevant margin so the opposite edge stays anchored. Commit only the axes/properties that the dragged handle touched.
- Read the rendered rectangle back during the drag; flex/grid/wrapping may not honor the requested size exactly.
- If width/height is committed to an inline element, explicitly change it to `inline-block` and log that change.
- Use pointer capture plus window-level move/up/cancel listeners so the drag survives leaving a small handle.
- On double-click, edit simple text or form values in place. Use `contenteditable="plaintext-only"` when supported, paste plain text, commit on Enter/blur, allow Shift+Enter, and cancel on Escape.
- Refuse in-place editing for elements with child markup; direct the user to the Content panel so nested spans and line breaks are not flattened.
- Commit frame text edits against the matching host element so scope, reset, persistence, and handoff remain centralized.
- Use a shared busy ref so Escape cancels the active drag/text edit before it closes the preview or tool.

## Responsive Device Lab

When the user asks for responsive preview, render the real page instead of a static miniature.

- Provide phone, tablet, and desktop families plus useful named device presets.
- Support rotate, fit/50/75/100% zoom, reload, pick-inside-preview, LTR/RTL, and optional reveal-animation freeze.
- Keep the main preview iframe mounted while temporarily hidden so reopening does not reboot the app or replay every entrance animation.
- Strip inspector/design/preview flags from the iframe URL to avoid recursion.
- Resolve the selected node through `uniquePath`, mirror edits into the frame, and replay edits after load/device change.
- Use cross-realm-safe checks (`nodeType`, tag name, `ownerDocument.defaultView`) for iframe nodes.
- If direct frame DOM access is unavailable, use a page-owned `postMessage` measurement bridge.
- Detect hidden/collapsed elements, page overflow, edge crowding, undersized type, tight control padding, and touch targets below the design-system threshold.
- Add a sampled breakpoint sweep across common and awkward widths, group consecutive failures into readable ranges, and make the report copyable.
- Use timers for settle points in non-compositing/off-screen frames; animation-frame callbacks may never fire there.

## Token Governance

- Normalize colors using browser parsing so `oklch()` and other modern values can be matched and audited. Never fall back to a default hex for an unparseable color.
- Audit fill, text, stroke, type size, gap, padding sides, and radius against shared and local tokens.
- Before creating a token, suggest a close existing token within a conservative color/spacing/radius tolerance.
- Creating or binding a token must do real work: declare a CSS custom property, apply `var(--token)` to the selected scope, persist the local token, and record an implementation instruction.
- Show a page-wide audit of visible values, grouped as tokenized, near-token, and one-off. Let the user jump to a sample element.
- Export local promoted tokens as paste-ready source for the canonical design-system file and CSS custom properties.
- Removing a local token must update the shared local store and all listening design-system surfaces.
- Track token provenance such as `provided`, `detected-static`, `detected-live-css`, `generated-audit`, and `fallback-default`. Surface it in the UI.
- Treat live detection as useful context, not canonical intent. Keep token binding, page token audit, and All variants locked until the host provides authoritative or statically detected tokens; show the reason on the disabled control.
- Resolve automatic tokens category by category: preserve named `:root` custom properties, merge missing rendered colors, always detect typography from computed styles, and audit spacing/radii only when declared scales are absent.

## Fonts

- Detect the full font stacks actually used by visible page text and rank them by usage.
- Group the picker as Project fonts, local/system fonts, and optional external fonts. Preview each choice in its own face and preserve authored fallback stacks.
- Lazy-load external font previews only when the picker opens; disclose that network request and do not imply the host project has adopted the font.

## Component States And Accessibility

- Preview Hover, Focus, Pressed, Disabled, Loading, and Error side by side using cloned content; do not mutate the live element merely to preview a state.
- Let designers adjust state fill, text, stroke, opacity, and scale, then emit semantic handoff selectors such as `:hover`, `:focus-visible`, `:active`, `[aria-disabled="true"]`, `[aria-busy="true"]`, and `[aria-invalid="true"]`.
- Make state edits visible in the live page: assign a stable temporary data marker to each edited element, rebuild a tool-owned stylesheet from the change log, and mirror the marker/rules into the device frame. Remove the stylesheet on Reset all.
- Replay state markers and generated rules after a device reload. Also show readable existing pseudo-state rules so the designer does not override a declaration that already agrees.
- Reset selection/all must clear state-editor drafts as well as live DOM edits.
- Check accessible name, actual contrast, minimum touch target, visible focus behavior, readable type, and image alt text using exported design-system thresholds.

## Asset Editing

Design mode should let the user replace any detected asset:

- selected or descendant `<img>`
- selected or descendant inline `<svg>`
- CSS `background-image: url(...)`

The Assets panel should support:

- paste image/SVG link and apply
- upload raster image
- upload SVG text and inline it when possible
- choose an icon from a small built-in SVG library

Inspect asset cards remain read-only/downloadable. Design asset cards are editable.

For asset reset, original state must include `innerHTML`, all existing attributes, and tracked image attributes like `src`, `srcset`, and `sizes`.

## Auto-Layout Reorder

Do not implement free dragging for auto-layout items.

- For flex/grid parents, reorder among siblings and let layout reflow.
- For non-layout parents, disable the affordance.
- Store original parent and next sibling for reset.

## State Editing

For hover/focus/active/focus-within rules:

- Read matching CSS rules when stylesheets are accessible.
- Translate declarations into editable rows instead of raw CSS blocks.
- Add quick overrides for border color, background, text color, opacity, scale, and shadow.
- `Preview state` applies edited declarations to the selected live element.
- Keep raw state CSS in Inspect or advanced handoff, not as the main Design control.

## Reset And Handoff

Before changing an element, store:

- original `style` attribute
- text/value and `innerHTML`
- original attributes
- original parent and next sibling
- original computed values for change-log before/after text

Generate:

- human-readable change log
- CSS diff for actual style changes
- content/attribute/asset/layout notes for non-CSS edits
- implementation instructions for token decisions
- separate Copy everything, Copy CSS, and Copy instructions actions

Persist replayable live edits by pathname and offer Restore/Discard on reload. Avoid clearing a saved session during the initial empty/StrictMode effect. Report edits that can no longer be resolved because markup changed.

## Embedded Host Integration

When a guide, documentation page, or product tour needs to drive the tool:

- Export a namespaced custom-event contract with optional `select`, `device`, and `section` fields.
- Give collapsible sections stable slug ids and support expand, scroll, and temporary spotlight behavior.
- Mark host controls that must remain clickable with a passthrough attribute and exclude them from hover, selection, editing, and audits.
- From a same-origin preview frame, dispatch requests to the parent window because the framed app must not mount another inspector.
- Guard inspector mounting with `window.self === window.top`, in addition to stripping recursive query parameters.
- Clear or hide overlays when the selected element disconnects after a route, slide, or conditional-render change.
- Treat a unified single editing surface as an optional product decision. If Inspect and Design tabs remain, preserve Inspect as read-only.

## Browse Mode And Comments

- Provide a clear Design/Browse switch. Browse mode unbinds selection listeners and hides overlays/pins so links, buttons, and navigation behave normally without losing the panel, selection, edits, or comments.
- Store comments by pathname and structural element path. Include component label, selector fallback, text, id, and timestamp.
- Show comment count on the selected element and in the panel, keep marker positions synchronized on scroll/resize, and skip elements absent in the current render.
- Add a comment mode that opens/focuses the composer after selection. Enter posts and Shift+Enter adds a line.
- Let users navigate to comments on other elements and delete comments.
- Include grouped comments in Copy everything and expose Copy comments separately.

Exclude `textContent`, `value`, `@attribute`, `layout:*`, and `asset:*` entries from CSS diff blocks.

## Verification

Run:

```bash
npx tsc --noEmit
npm run build
```

Browser-check:

- Inspect cannot edit.
- Component preview appears in Inspect and links to the design system.
- Nested children can be selected.
- Design mode shows Free change and Component change.
- Free change edits one part.
- Component change edits matching variants only.
- Font size dropdown applies visible changes.
- Asset replacement works for image, inline SVG, background, upload, link, and icon library.
- Reset restores style, text, attributes, assets, and layout order.
- Change log is readable and CSS diff is valid.
- Numeric edits do not append digits, jump, or reformat while typing; Escape cancels cleanly.
- Repeated/trailing spaces survive content editing.
- Host shortcuts do not fire from panel controls.
- Device edits mirror live, frame picking works, rotate/zoom/RTL/reload work, and breakpoint failures are reported as ranges.
- Token creation applies `var(...)`, nearest-token suggestions work, local tokens export, and the page audit can select an example.
- Six state previews emit valid selectors and reset correctly.
- Accessibility findings use canonical thresholds and correct color parsing.
- A reload offers to restore replayable edits for the current pathname.
- Direct canvas resize previews, cancels, and commits through the same reset/handoff pipeline at page and device-preview zoom levels.
- In-place text editing preserves simple text, rejects nested markup, and does not trigger host shortcuts.
- Edited component states render live in both host and device documents and disappear on reset.
- Programmatic open/select/device/section requests work, passthrough controls remain interactive, and framed routes do not recursively mount the tool.
- Browse mode restores normal site interaction without discarding work.
- Comments persist per pathname, follow live geometry, navigate to their elements, and appear in handoff output.
- Project font choices reflect the host page; external previews load only on demand.
- Heuristic token sources remain visibly non-authoritative and cannot unlock canonical-only actions.
