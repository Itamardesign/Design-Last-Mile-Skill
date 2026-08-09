# Visual Inspector Spec

Use this spec when implementing a Bubble handoff inspector in a new project.

## Contents

- [Delivery and UI](#delivery-gate)
- [Preview and information hierarchy](#component-preview)
- [Text, box model, distances, and states](#text-inspection)
- [Copy and color normalization](#copyable-output)
- [Frames and guided actions](#same-origin-frame-support)
- [Assets, Design System, limitations, and verification](#asset-detection)

## Delivery Gate

- Mount and lazy-load the inspector only when an explicit URL flag is present, for example `?inspect=true` or `?inspect=1`.
- Keep the launcher inside the gated bundle. A hidden inspector must not leave a launcher or selection listeners behind.
- Remove inspector-only query parameters from any nested preview URL to prevent recursive inspectors.
- Also refuse to mount when `window.self !== window.top`; route-based auto-enable rules can otherwise recurse even after query cleanup.

## UI Composition

- Toggle: fixed, vertical, on the currently selected dock side, above app content.
- Panel: docked, draggable, or fixed sidebar around 340-420px wide on desktop; drawer on smaller screens if supported.
- Dock controls: separate left/right icon buttons, visible pressed state, and a namespaced persisted preference.
- Header: product label, Inspect/Design mode tabs when Design exists, Unlock, close/toggle controls.
- State bar: `Preview mode` until clicked, `Selected element` after click.
- Sections:
  - Component preview
  - Layer properties
  - Typography
  - Colors
  - Text content
  - Assets
  - States
  - Selection details
  - Distances
  - Code/CSS
  - Full technical details
  - Not shown

## Component Preview

The component preview is the bridge between Inspect, Design, and Design System.

- Clone the selected element into a preview host.
- Remove anything matching `[data-inspector-ui]`.
- Set `aria-hidden`.
- Disable pointer events.
- Remove autofocus.
- Make form controls read-only.
- Constrain absolute/fixed/sticky elements by resetting positioning inside the preview.
- Show:
  - main component label
  - selected tag/type
  - match count
  - component-family selector/reason when available
  - Design System link
  - token match chips

## Information Hierarchy

Use a relevance-first panel to prevent handoff overload.

- The top section should answer what the selected thing is and where it belongs in the system.
- Show readable text prominently whenever text exists.
- Choose first visible sections by element type:
  - Text/Button/Input/Dropdown: text, typography, colors, border/radius, spacing, states.
  - Image/icon: asset preview/download, dimensions, radius/object-fit, spacing.
  - Group/flex or Group/grid: layout, gap, padding, child count, distances, background, border/radius.
  - Form: labels, fields, buttons, layout, spacing, ARIA, states.
- Put exhaustive selector/style/attribute JSON and low-priority rows later.

## Text Inspection

- Show normalized text near the top.
- For inputs/textareas, use current `value`.
- For regular elements, use normalized `textContent`.
- If caret APIs reveal the direct text node under the click, show it separately.
- Keep typography, layout, spacing, and colors visible too.

## Visual Layer Properties

The box model diagram should show:

- outer margin top/right/bottom/left
- border width
- padding top/right/bottom/left
- content size
- full rendered size

Use different tints for margin, padding, and content.

## Distance Measurements

Show two distance groups:

- nearby siblings: nearest top/right/bottom/left sibling in the same parent
- parent container: distance from selected element to parent top/right/bottom/left edges

Measurement algorithm:

```js
function overlaps(a, b, axis) {
  if (axis === 'x') return Math.max(a.left, b.left) < Math.min(a.right, b.right);
  return Math.max(a.top, b.top) < Math.min(a.bottom, b.bottom);
}
```

For top/bottom, require x-overlap. For left/right, require y-overlap. Choose the smallest positive distance per side.

## Border And Radius

Show border separately from generic style rows:

- Border color with swatch and copyable HEX.
- Radius with copyable CSS value.
- Border width and style.
- Include `border` and `border-radius` in CSS snippets.

## State Inspection

- Search readable stylesheets for `:hover`, `:active`, `:focus`, `:focus-visible`, and `:focus-within`.
- Process normal `CSSStyleRule` declarations before recursing into nested rules.
- Remove pseudo-class and test base selector matches selected element or relevant ancestor.
- Show state label, original selector, and declarations.
- Convert rgb/rgba colors to HEX.
- Ignore cross-origin stylesheet read failures.
- Show current state classes.

## Copyable Output

Make these click-copyable:

- each HEX color swatch
- border color/radius/style
- each distance value
- CSS code block
- state CSS blocks
- visible text content
- selected element selector/DOM path
- full selected element JSON

## Color Normalization

Computed styles may return `oklch()`, `color()`, or other modern syntax instead of rgb. Resolve colors through a 1px canvas when the browser accepts the value, then read the rendered sRGB bytes.

- Seed `fillStyle` twice before accepting a parse so an invalid assignment cannot reuse the previous color.
- Preserve alpha.
- Cache resolved values.
- Return `null` for unparseable input; never substitute a brand color, because that creates false token matches and invalid contrast results.

## Same-Origin Frame Support

When a responsive preview renders the same app in an iframe:

- store a structural `uniquePath` in the snapshot and resolve it in the frame document;
- use `ownerDocument.defaultView.getComputedStyle`;
- detect elements by `nodeType`/tag name, not parent-window `instanceof`, because iframe nodes belong to another realm;
- prevent selection/navigation inside the frame while pick mode is active;
- remove `inspect`, `design`, and preview-only parameters from the frame URL.

If the browser isolates frame DOM access even for the preview, add a page-owned `postMessage` bridge that resolves the selector and returns measurements. Treat this as a fallback architecture, not a reason to fake measurements.

## Passthrough And Guided Actions

- Define one ignored selector that includes both inspector-owned UI and host-owned passthrough controls.
- Use the passthrough marker for navigation, tutorial “Try it” buttons, or controls that must still activate while inspection is open.
- Exclude ignored nodes from selection, component matching, and page-wide audits.
- For embedded walkthroughs, accept a namespaced custom event containing an optional selector, device family, and section id.
- Give sections stable slug ids, expand them on request, scroll them into view, and briefly spotlight them.
- Forward the event from same-origin frame content to the parent window.
- Do not draw overlays for `!selectedElement.isConnected`.

## Browse Mode And Comments

- Browse mode leaves the panel mounted but unbinds selection listeners and hides overlays/comment pins.
- Persist review comments under a pathname-scoped key with structural path plus selector fallback.
- Re-resolve comment targets and marker geometry after scroll/resize; tolerate targets absent in the current render.
- Show selected and elsewhere comments, navigate back to their elements, and include grouped notes in handoff output.

## Package Safety

- Default reusable packages to disabled unless an exact URL flag is present; an explicit `enabled` prop wins.
- Decide URL enablement after mount and listen for `popstate` to remain SSR/hydration safe.
- Make all storage reads exception-safe and browser-guarded.
- Inject CSS once after enablement with a stable id; keep browser CSS and Node-only configuration loaders in separate entry points.
- For Git installs without committed build output, run a `prepare` build; for registry packages with prebuilt output, skip rebuilding when source is absent.

Recommended CSS snippet fields:

```css
width: 320px;
height: 72px;
display: flex;
flex-direction: row;
gap: 12px 12px;
margin: 0px 0px 0px 0px;
padding: 16px 20px 16px 20px;
border: 1px solid #dfe3e8;
border-radius: 12px;
background: #ffffff;
color: #111827;
box-shadow: #0f172a14 0px 8px 24px 0px;
```

## Asset Detection

Detect assets in selected node and descendants:

- `<img>` currentSrc/src
- inline SVG serialized as downloadable data URL
- CSS `background-image: url(...)`

Inspect shows download links. Design mode may add replacement controls, but that belongs to the design-tab workflow.

## Relationship To Design System

When a design-system source exists:

- Import token arrays into the inspector.
- Show token-match chips beside snapshot colors.
- Link the selected component preview to the design-system route.
- Keep design-system content out of Inspect tabs unless it is direct context for the selected element.

## Limitations Section

Include these caveats:

- CSS variable names may be lost after computed style resolution.
- JavaScript-only states are visible only if reflected in DOM/classes/attributes.
- Cross-origin CSS rules may be inaccessible.
- Remote font file sources may not be discoverable.
- Complex animations are summarized, not fully simulated.

## Verification

- Toggle visible and opens panel.
- Hover preview updates.
- Click locks selection.
- Nested children can be selected.
- Component preview renders.
- Design System link works when available.
- Full technical details are available but not the first thing the user sees.
- Distance rows and measurement lines appear.
- Border/radius rows render and copy.
- State rules render when matching CSS exists.
- Assets render with thumbnails/download links.
- Inspect has no mutation controls.
- Disabled URL has no launcher, global selection listeners, or inspector bundle work.
- Dock side persists.
- Host keyboard shortcuts do not fire while typing in inspector-owned controls.
- Modern CSS colors do not fall back to a fake match.
- Passthrough controls remain usable, frame recursion is blocked, and disconnected selections clear visually.
- Browse mode and pathname-scoped comments behave across navigation and layout changes.
- Disabled/SSR package imports do not create UI, listeners, styles, or browser-global errors.
