# Design + Inspect + Design System Pattern

Use this reference when cloning the complete panel system into another frontend project. The pattern has three cooperating surfaces:

## Contents

- [Architecture and contracts](#file-map)
- [Inspect and Design responsibilities](#inspect-responsibilities)
- [Direct editing and assets](#direct-canvas-manipulation)
- [Design System and tool shell](#design-system-responsibilities)
- [Responsive, tokens, states, and accessibility](#responsive-device-lab)
- [Sessions, walkthrough integration, and governance](#persistent-handoff-sessions)
- [Portable package delivery and token resolution](#portable-react-package)
- [Clone order and verification](#clone-order)

- `DesignSystem`: the canonical source and reference page for tokens, state visuals, text styles, colors, and reusable UI patterns.
- `Inspect`: a read-only developer/Bubble handoff mode that explains the selected live component, shows a preview, links back to the design system, and exposes copyable specs.
- `Design`: a designer editing mode that uses the same selection and snapshot data as Inspect, but applies reversible live-preview edits to either the selected part or the full component family.

The three surfaces should feel like one product. Inspect answers "what is this and how do I rebuild it?", Design answers "can I try a change now?", and DesignSystem answers "what should this become permanently?".

## File Map

For a React/Tailwind/Vite-style project, clone or recreate these files:

- `src/designSystem.ts`
  - tokens, color arrays, typography/pattern classes, state labels, state metadata, character/visual themes
  - this file has no React UI and should be safe to import from components, inspectors, sandboxes, and docs pages
- `src/components/DesignSystem.tsx`
  - full-screen reference page with tabs such as Character, Text, Color, Patterns
  - imports from `designSystem.ts` and exposes copyable values/snippets
- `src/components/HandoffInspector.tsx`
  - one overlay component with shared selection state and two main modes: `inspect` and `design`
  - contains snapshot capture, overlay measurements, Inspect details, Design controls, component-family matching, reset, and handoff output
- app entry or layout
  - mounts `<HandoffInspector />` once, outside the mobile/app frame when possible
  - exposes the design system page through `#design`, `/design`, `?design=true`, or the host router

When cloning into a non-React app, keep the same responsibilities but change the rendering layer. The important contract is the data flow, not the exact component names.

## Data Flow

```text
designSystem.ts
  -> app components use tokens/pattern classes
  -> DesignSystem page renders the token/state/pattern reference
  -> HandoffInspector imports tokens for swatches and token matches

User selects live DOM element
  -> createElementSnapshot(element)
  -> InspectDetails reads snapshot only
  -> DesignMode reads snapshot + selected live element

Inspect
  -> component preview clone
  -> token matches
  -> read-only assets/specs/states
  -> Design System link

Design
  -> Free change edits selected element only
  -> Component change edits all matching component variants on page
  -> stores original state before first edit
  -> reset element/reset all restores DOM
  -> change log + CSS/content/asset notes become handoff output
```

## Core Contracts

`ElementSnapshot` should include:

- identity: tag, selector, DOM path, parent selector, child count, Bubble/component hint
- text: clicked text, full text, whether a direct text node was selected
- geometry: rect, display, position, flex direction, alignment, justify, gap, parent gap
- typography: font family, size, line height, weight, text align
- box/style: margin, padding, border width/style/color, radius, background, text color, shadow, opacity, filter, transform, transition
- distance data: parent distances, nearest sibling distances, measurement lines
- handoff data: attributes, colors converted to copyable hex, CSS snippet, current state classes, readable pseudo-state CSS rules
- assets: selected/descendant `img`, inline SVG, and CSS background URLs
- a structural `uniquePath` that can resolve the equivalent node after reload and inside a same-origin preview frame

`DesignOriginalState` should store enough to reset every live edit:

- original `style` attribute
- original `textContent`, `innerHTML`, and form `value`
- all original element attributes plus tracked attributes such as `src`, `srcset`, `sizes`, `alt`, `href`, `placeholder`, `aria-label`, `title`
- original computed values for the change log
- original parent and next sibling for reorder reset

`ComponentFamilyInfo` should include:

- user-facing component label
- matching selector or reason
- ordered list of matching live elements, with the selected element first
- match count

## Component Family Matching

Use a conservative matching ladder:

1. Prefer explicit markers: `data-component`, `data-design-component`, `data-ui`, `data-testid`, `data-test`.
2. Use tag plus comparable classes when there are enough stable classes.
3. Use tag plus `role`.
4. Fall back to similar tag/class/child-structure matching.

Filter all matches through:

- not inside `[data-inspector-ui]`
- visible rect width/height
- same component kind (`text`, `button`, `link`, `input`, `image`, `layout`, `form`, `generic`)

This makes Component change useful without unexpectedly editing unrelated elements.

## Inspect Responsibilities

Inspect is read-only. Never pass edit callbacks into Inspect details.

The top of Inspect should show:

- Component preview: clone the selected element, remove inspector UI, disable pointer events/autofocus, make inputs read-only, constrain absolute/fixed clones into the preview box
- Main component label: from `ComponentFamilyInfo`, designer profile, or Bubble hint
- Match count: number of page variants the Design tab can edit in Component change mode
- Design System link: open `#design`, `/design`, or the app's design route
- Token matches: compare snapshot colors with `brandColorTokens` and show matching token labels

Then show relevance-first handoff sections:

- layer properties and box model
- typography when relevant
- colors
- readable text content
- assets as thumbnails/download links only
- states as copyable CSS/readable declarations
- selection details and distances
- full JSON/code only behind lower-priority sections

## Design Responsibilities

Design edits the live preview only; it should not write source files by itself.

Always show the scope toggle:

- `Free change`: property edits apply only to the selected element/part.
- `Component change`: property edits apply to every matching component variant on this page.

Render controls by component profile:

- Content: text/value, placeholder, ARIA label, link href, image URL/alt/upload.
- Layout: width, height, display, flex/grid controls, direction, align, justify, gap, margin, padding, reorder.
- Appearance: opacity, radius.
- Fill: background color plus design-token swatches.
- Stroke: border color, width, style.
- Text: font family, weight dropdown, size dropdown, line height, letter spacing, align segmented control, color.
- Effects: shadow presets and custom value.
- Assets: replace any discovered image/SVG/background asset by upload, URL, uploaded SVG, or an internal icon library.
- States: parse hover/focus/active/focus-within declarations into editable property/value rows and quick-add chips.

Normalize numeric values before applying CSS. For length properties such as `font-size`, `width`, `height`, `gap`, `padding`, `margin`, and `border-radius`, a bare number should become `px`.

Keep numeric fields stable while live measurements refresh. The field owns a local draft while focused, commits once on blur/Enter, cancels on Escape, and commits arrow-key steps. Keep content editing on raw text with its own draft so repeated and trailing spaces are not normalized away.

## Direct Canvas Manipulation

Add this only when the user wants Figma-style editing on the canvas itself.

- Gate it behind an explicit persisted toggle; panel inputs remain the fallback.
- Render eight labeled resize handles inside the selection box while leaving the overlay pointer-transparent.
- Start a resize session with the rendered box, margins, aspect ratio, preview scale, and exact prior inline values.
- Preview touched properties in host and device twin nodes. Divide pointer movement by preview zoom.
- Shift preserves the starting ratio; Alt snaps to a step imported from the spacing scale; Escape/pointercancel restores the exact inline values.
- Compensate `margin-left`/`margin-top` for west/north handles and commit only touched axes.
- Read back `getBoundingClientRect()` during drag because layout may settle differently from requested dimensions.
- Roll back preview styles before committing through the normal edit function so handoff `before` values stay truthful.
- Convert inline elements to inline-block before committing dimensions.
- Double-click simple text/fields for in-place plain-text editing. Commit on Enter/blur, allow Shift+Enter, cancel on Escape, and block host shortcuts.
- Refuse elements with child markup. Commit device-frame text through the matching host node so component scope, reset, persistence, and handoff stay centralized.

## Asset Replacement Flow

Inspect assets are downloadable/read-only. Design assets are editable.

Detect assets with stable ids:

- `img-0`, `img-1`, ...
- `svg-0`, `svg-1`, ...
- `bg-0`, `bg-1`, ...

For replacements:

- Link: paste a URL and apply it.
- Image upload: use `URL.createObjectURL(file)` for raster images.
- SVG upload: read `file.text()`, sanitize scripts/event attributes, then inline it into the target SVG or convert it to a data URL for image/background targets.
- Icon library: provide a small built-in SVG set such as Check, Spark, Heart, Book, Mic, Arrow. It is fine to use `lucide-react` elsewhere, but the asset picker should store plain SVG markup so it can replace inline SVGs.

Apply by target type:

- `img`: set `src`, clear `srcset` and `sizes`.
- inline `svg`: replace attributes/innerHTML from parsed SVG while preserving useful class/style/size/accessibility attributes.
- background: set `background-image: url("...") !important`.

Change logs should summarize asset edits without dumping giant SVG strings. Example: `svg-0 asset: Spark icon (inline SVG)`.

## Design System Responsibilities

The design system page is the source-of-truth surface for the same values Inspect and Design reference.

Recommended page tabs:

- Character or Brand/States: every visual state and variant, with copyable component snippets and token colors.
- Text: typography scale with class strings or CSS recipes.
- Color: `brandColorTokens` plus neutral scale, each click-copyable.
- Patterns: reusable component recipes such as chat bubbles, inputs, buttons, voice/action buttons, cards, and navigation pieces.

Expose a route the inspector can open:

- simple apps: `#design`
- router apps: `/design`
- query-driven prototypes: `?design=true`

When permanent source changes are requested, update `designSystem.ts` first, then the app components, then confirm Inspect still reads the updated computed styles.

## URL Gate And Tool Shell

- Lazy-load the inspector only for an explicit URL gate such as `?inspect=true` or `?inspect=1`.
- Keep the launcher inside the gate so normal visitors receive no visible tool UI or selection listeners.
- Add explicit left/right dock buttons and persist the side with a namespaced storage key.
- Strip inspector, design-system, and preview-only parameters from nested device-preview URLs.
- Also block mounting when `window.self !== window.top`; route-driven auto-enable rules can recurse without a query parameter.
- Keep all tool nodes marked with `[data-inspector-ui]`.
- Define a host passthrough marker for navigation and tutorial controls that must remain clickable; exclude it from hover, selection, editing, matching, and audits.
- Stop non-Escape keyboard events from tool-owned controls before they reach host-page window shortcuts.
- Use tool visual tokens from the canonical source, larger readable labels/inputs, focus feedback, restrained entrance motion, and `prefers-reduced-motion`.

## Responsive Device Lab

Use a full-screen real-page preview when responsive review is requested.

- Define device presets in the design-system source: useful phones, tablets, and desktops with width, height, shell type, and radius.
- Support device family/model, portrait/landscape, fit/50/75/100% zoom, reload, pick mode, and LTR/RTL.
- Keep the preview iframe mounted while hidden after first open to avoid rebooting the app and replaying entrance motion.
- Resolve the selection through `uniquePath`; mirror edits into the frame and replay them after load or breakpoint changes.
- Use `ownerDocument.defaultView.getComputedStyle` and node/tag checks. Parent-window `instanceof` fails for iframe nodes.
- If same-origin DOM access is isolated by the browser shell, implement a page-owned `postMessage` bridge that resolves the path and returns real metrics.
- Freeze reveal animations only when needed for review, with a narrowly scoped injected rule that restores opacity/transform for mid-reveal inline states.
- Detect hidden/collapsed output, horizontal overflow, edge crowding, undersized type, tight control padding, and small touch targets.
- Sweep representative widths including gaps between named breakpoints, replay edits at every width, group adjacent failures into ranges, and make the report copyable.
- Use bounded timers for off-screen iframe settle points; `requestAnimationFrame` may not fire when the frame is not compositing.

## Token Binding And Audit

Token binding must change the preview and produce a durable source instruction.

- Normalize colors through browser parsing/canvas so `oklch()` and other modern computed values can become real sRGB values.
- Never substitute a default color when parsing fails; that creates false token matches and corrupts contrast math.
- Audit fill, text, stroke, type size, gap, each padding side, and radius.
- Match canonical and local tokens exactly first; then suggest a conservative nearest color/spacing/radius token before offering a new token.
- Creating a token persists a typed provisional token, declares a CSS custom property, binds the selected property to `var(--token)`, mirrors it to the device preview, and records the exact source change required.
- A page-wide audit scans visible elements and groups distinct values as tokenized, near-token, or one-off; selecting a row should jump to a sample element.
- Export provisional tokens as paste-ready TypeScript token arrays and CSS custom-property declarations.
- The Design System listens to a same-tab custom event and cross-tab `storage` updates and shows provisional tokens until they graduate into source.

## Component States And Accessibility

- Render inert cloned previews for Hover, Focus, Pressed, Disabled, Loading, and Error.
- Let state controls edit fill, text, stroke, opacity, and scale without forcing the live source element into the state.
- Generate handoff CSS with `:hover`, `:focus-visible`, `:active`, `[aria-disabled="true"]`, `[aria-busy="true"]`, and `[aria-invalid="true"]`.
- Apply state edits live with stable temporary data markers and one tool-owned stylesheet. Mirror the marker and rebuilt rules into the device frame, replay after reload, and remove them on reset.
- Show readable existing pseudo-state CSS near the state editor so designers understand the current source behavior.
- Reset selection/all must also reset state-editor drafts.
- Audit accessible name, actual foreground/background contrast, touch-target size, focus visibility, readable type, and image alt text.
- Import all thresholds from `designSystem.ts`.

## Persistent Handoff Sessions

- Store replayable CSS, content, attribute, and token edits by pathname. Keep asset/layout/state entries out unless they can be serialized and replayed safely.
- Store generated CSS variables alongside the change list.
- On reload, offer Restore or Discard and report paths that no longer resolve.
- Do not clear saved state from the initial empty effect or React StrictMode replay; clear it only after this mounted session previously saved changes and then becomes empty.
- Offer Copy everything, Copy CSS, and Copy instructions. Token instructions should explicitly say whether to add a new token or bind the nearest existing one.

## Embedded Walkthrough API

- Export a namespaced custom event with optional `select`, `device`, and `section` fields.
- Give tool sections stable slug ids and open-state attributes.
- Mount/open first, then resolve selection and section focus after the shell exists.
- Forward requests from same-origin preview content to the parent window.
- Briefly spotlight the requested section and honor reduced motion.
- Check `element.isConnected` before drawing a selection after route, slide, or conditional-render changes.
- Keep the API optional. Do not couple ordinary inspection to a product-specific blog or tutorial.

## Browse Mode, Fonts, And Comments

- Add a Design/Browse switch when users need to navigate the real site while the panel remains open. Browse mode unbinds selection behavior and hides overlays/pins without discarding state.
- Detect project font stacks from visible text and show them first. Follow with offline system fonts and a small optional external-font list; lazy-load external previews only when opened.
- Persist comments by pathname plus structural element path. Show on-canvas counts, resync pins on scroll/resize, tolerate absent responsive targets, navigate to other commented elements, and include grouped notes in handoff.

## Portable React Package

When extracting the workflow as a reusable package:

- Export one `HandoffInspector`, optional `DesignTokensProvider`, `useDesignTokens`, detection helpers, token types, and the URL parameter constant.
- Keep React/ReactDOM as peer dependencies and require a client component boundary in server-component frameworks.
- Default to an exact opt-in URL such as `?designmode=true`; let an explicit `enabled` prop override it.
- Read URL state after mount and listen to `popstate` to avoid SSR hydration mismatch. Guard browser globals and exception-prone storage access.
- Inject a generated CSS-string module once after enablement using a stable style id. This avoids a mandatory consumer CSS-loader configuration.
- Put Node-only config detection behind a separate package subpath so filesystem imports never enter the browser bundle.
- If Git installs do not include `dist`, use `prepare` to build from source. Skip the build when a registry package contains prebuilt output but no source.

## Token Resolution And Trust

Use generic host-facing token types and keep tool chrome/device/accessibility constants separate.

Resolve token data by category:

1. Explicit provider tokens.
2. Static JSON or importable Tailwind configuration.
3. Named same-origin `:root` CSS custom properties.
4. Rendered role-aware colors and full typography signatures.
5. Clustered page-audit suggestions for missing spacing/radius scales.
6. Generic fallback.

Expose provenance (`provided`, `detected-static`, `detected-live-css`, `generated-audit`, `fallback-default`). Detection makes zero-config editing useful, but only provided/static sources unlock token binding, page audit, and component-wide editing. Disabled controls explain how to connect the provider.

## Design-System Governance

Add a repository-level rule for future UI work:

1. Reuse semantic tokens, typography recipes, states, patterns, and shared components first.
2. Permit a new decision when the current system does not fit a real product need.
3. If the decision is reusable, add it to the canonical source, preview/document it in the visible page, and consume that shared definition in product code.
4. Cover interaction, responsive, RTL, and accessibility states.
5. Keep intentional one-offs local and document why they should not become tokens.
6. Verify reuse and justified additions before calling the UI task complete.

## Clone Order

1. Read the target app structure, styling system, and router.
2. Add or port `designSystem.ts` with tokens, state arrays, pattern classes, and alignment checklist.
3. Add or port `DesignSystem.tsx`, adapting tabs to the target product.
4. Mount the design-system route/page.
5. Add or port `HandoffInspector.tsx`.
6. Ensure the inspector root and all panel/overlay controls carry `data-inspector-ui`.
7. Connect `brandColorTokens` to token swatches and token-match chips.
8. Connect the Inspect component preview link to the design-system route.
9. Verify Inspect has no edit callbacks.
10. Verify Design free/component modes, resets, asset replacement, state previews, and change-log output.
11. Verify numeric/text draft behavior and host-keyboard isolation.
12. Verify device preview, cross-frame selection, mirrored edits, RTL, reveal freeze, and breakpoint sweep.
13. Verify exact/near token matches, actual `var(...)` binding, page audit, export, and catalog refresh.
14. Verify accessibility findings and reload session restore/discard.
15. If direct canvas editing exists, verify resize modifiers, zoom scaling, cancel/rollback, in-place text safety, and normal change logging.
16. If host integration exists, verify passthrough controls, event-driven selection/device/section focus, frame forwarding, and stale-selection cleanup.
17. If packaged, verify disabled/SSR imports, URL and `enabled` gating, style injection, client navigation, provider priority, detection source labels, and locked canonical-only features.
18. Verify Browse mode, project-font ordering/lazy external previews, and pathname-scoped comment handoff.

## Verification

Run static checks and a production build:

```bash
npx tsc --noEmit
npm run build
```

Then browser-check:

- Design system route opens and closes.
- Inspector toggle opens.
- Inspect selects nested elements without selecting inspector UI.
- Inspect component preview renders a clone and shows the Design System link.
- Inspect asset cards are read-only/downloadable.
- Design tab shows Free change and Component change.
- Text size dropdown applies visible `font-size`.
- Free change edits only the selected element.
- Component change edits all matching variants and tells the user match count.
- Reset element and Reset all restore style, text, attributes, assets, and order.
- Asset panel can replace inline SVG with library icon, uploaded SVG, raster upload, and URL.
- CSS diff excludes `asset:*`, `textContent`, `value`, attributes, and layout notes.
- Numeric fields do not mutate measured values mid-entry, and raw content retains intended whitespace.
- Modern CSS colors do not fall back to a fake brand match.
- Device frames do not recurse into another inspector, cross-realm picking works, and breakpoint ranges are evidence-based.
- Token creation declares and applies a CSS variable and produces a source instruction.
- Saved replayable edits survive reload for the current pathname.
- Direct manipulation commits through the same scope/reset/handoff pipeline and leaves nested markup safe.
- State overrides render live in host and device documents and disappear on reset.
- Embedded walkthrough controls remain clickable and framed routes do not recursively mount the tool.
- The package works without setup, but never presents heuristic design values as authoritative tokens.
- On mobile or non-desktop breakpoints, either hide the full inspector intentionally or make the drawer usable.
