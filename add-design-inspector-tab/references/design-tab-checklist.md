# Design Tab Implementation Checklist

Use this checklist when implementing a designer-only tab beside an existing read-only Inspect panel.

## Contents

- [Component and snapshot structure](#component-shape)
- [Inspect and Design panels](#inspect-panel)
- [Reliable fields, devices, and canvas editing](#reliable-field-editing)
- [States, host integration, tokens, and accessibility](#live-state-overrides)
- [Controls, component families, reset, and assets](#designer-controls-by-type)
- [Common failure modes](#common-failure-modes)

## Component Shape

- Keep a single inspector root with shared selection state.
- Use `mode: 'inspect' | 'design'`.
- Keep selected element in a ref to avoid unnecessary rerenders.
- Keep selected snapshot in state for the panel and overlay.
- Keep design originals in a ref keyed by stable selector.
- Keep design changes in state as selector -> property/value map.
- Keep inspector UI marked with `[data-inspector-ui]`.
- Store a structural path for frame mirroring and session replay.
- Keep generated token variables and a pathname-scoped replayable session separate from element originals.

## Snapshot Data To Capture

- tag, selector, DOM path, parent selector, child count.
- clicked text/full text and whether the click hit a direct text node.
- bounding rect.
- display, position, flex direction, align, justify, gap, parent gap.
- font family, size, line height, weight, text align.
- margin, padding, border widths, border style/color, radius.
- background, text color, shadow, opacity, filter, transform, transition.
- distances to parent and nearest siblings, with measurement lines.
- readable hover/active/focus/focus-visible/focus-within rules.
- current state classes such as active, selected, disabled, open, expanded, loading, error, success.
- img/svg/background assets.
- copyable CSS snippet and JSON.

## Inspect Panel

- Keep read-only.
- Top section should be Component preview.
- Clone the selected element into a preview host.
- Remove inspector UI from the clone.
- Disable pointer events, autofocus, and input editing inside the clone.
- Constrain absolute/fixed/sticky clones to preview bounds.
- Show main component label, tag/type, match count, and component-family selector.
- Add Design System link to `#design`, `/design`, or the host route.
- Show design token matches by comparing snapshot colors to `brandColorTokens`.
- Assets are thumbnails/download links only.

## Design Panel

- Header shows the selected component label.
- Scope toggle is always visible:
  - Free change
  - Component change
- Reset element and Reset all are near the top.
- Render controls by profile, not by every possible CSS property.
- Designer changes section shows change log and CSS diff.
- CSS diff excludes text, values, attributes, layout notes, and asset notes.
- Handoff offers Copy everything, Copy CSS, and Copy instructions.

## Reliable Field Editing

- Numeric field owns a draft while focused.
- Snapshot refresh updates the field only while it is not being edited.
- Blur/Enter commits one normalized value; Escape restores the external value; arrow keys commit their step.
- Numeric commit clamps min/max and rounds once.
- Textarea owns raw text while focused so repeated/trailing whitespace survives snapshot refresh.
- Editing text on an element with child markup shows a destructive-content warning.
- Keyboard events from inspector inputs do not bubble to host `window` shortcuts.

## Device Preview And Breakpoint Sweep

- Uses the real page in a same-origin iframe when possible.
- Removes inspector/design/preview parameters from the iframe URL.
- Includes useful phone, tablet, and desktop models, rotation, zoom, reload, pick mode, RTL, and reveal freeze.
- Keeps the preview frame mounted across temporary close/open.
- Resolves the selected element with a structural path.
- Uses cross-realm-safe element checks and the frame's own `getComputedStyle`.
- Mirrors/replays edits after load and across breakpoint variants.
- Falls back to a page-owned `postMessage` measurement bridge when frame DOM access is isolated.
- Reports hidden/collapsed, overflow, edge, type, padding, and touch-target problems.
- Sweeps multiple widths, groups adjacent failing samples into ranges, and copies a useful report.

## Direct Canvas Editing

- Explicit toggle persists independently from panel open/closed state.
- Eight resize handles are keyboard-focusable and labeled; overlay remains pointer-transparent.
- Pointer delta is divided by preview zoom.
- Temporary preview styles are rolled back before normal style commits record the true before value.
- Shift preserves ratio; Alt snaps to a spacing-token step; Escape/pointercancel restores exact inline styles.
- Left/top drags compensate position and only touched axes are committed.
- Rendered rect, not requested dimensions, drives the live overlay.
- Inline elements become inline-block before width/height commit.
- Double-click text uses plain-text editing, stops host shortcuts, commits on Enter/blur, and cancels on Escape.
- Elements with child markup refuse direct text editing.
- Frame edits commit through the matching host element and normal scope/reset/handoff pipeline.

## Live State Overrides

- Every state-edited target gets a unique temporary data marker.
- A tool-owned stylesheet is rebuilt from state changes and injected into host and device documents.
- State marker and rules replay after device reload.
- Existing readable pseudo-state rules remain visible beside editable previews.
- Reset all removes markers/rules and clears state-editor drafts.

## Host Integration API

- Namespaced custom event can open the tool, select a selector, choose a device, and focus a panel section.
- Tool sections expose stable ids and open state.
- Passthrough attribute keeps documentation/navigation controls clickable and outside selection/audits.
- Frame-owned controls forward requests to the parent.
- Framed routes never mount another inspector.
- Disconnected selections stop drawing stale overlays.

## Token Governance

- Browser-normalizes modern colors such as `oklch()` without a fake fallback.
- Exact match covers colors, spacing, radii, and type sizes.
- Near-match suggestion appears before Create token.
- Create token persists the token, declares a CSS variable, binds the property to `var(...)`, and adds a source-change instruction.
- Local tokens can be exported as TypeScript arrays plus CSS custom properties.
- Page audit groups visible values into tokenized, near-token, and one-off buckets.
- Clicking an audit value selects a sample element.
- Token context exposes its provenance.
- Live/generated/fallback tokens may populate previews but do not unlock binding, page audit, or All variants.
- Provided/static tokens unlock canonical-only controls; disabled controls explain how to connect them.
- Detection preserves CSS-variable names, merges missing rendered colors, detects type signatures, and fills absent spacing/radius scales from an audit.

## Fonts, Browse Mode, And Comments

- Font picker lists project stacks first, then offline system fonts, then capped optional external previews.
- External fonts load only when the picker opens and are labeled as requiring source adoption.
- Browse mode removes selection/edit listeners and hides overlays without closing the panel or clearing work.
- Comments persist by pathname + structural path, show pins/counts, resync on scroll/resize, and tolerate absent elements.
- Comment mode focuses the selected element's composer; Enter posts and Shift+Enter creates a line.
- Handoff includes grouped comments plus a dedicated Copy comments action.

## States And Accessibility

- Side-by-side previews: Hover, Focus, Pressed, Disabled, Loading, Error.
- Preview clones are inert and do not modify source/live state.
- Handoff selectors use pseudo-classes and semantic ARIA attributes.
- Checks cover name, contrast, target size, focus, type size, and image alt.
- Thresholds come from the design system.

## Designer Controls By Type

Text:

- textarea for content.
- font family.
- font size dropdown.
- font weight dropdown.
- line height, letter spacing.
- text align segmented control.
- color picker plus brand swatches.
- typography presets if available.

Image:

- thumbnail preview.
- URL input.
- upload replacement.
- alt text.
- object-fit/object-position.
- size and radius.

Button/link:

- text/href.
- text color/background.
- padding, radius, border, shadow.
- state editor.

Layout:

- display mode.
- flex/grid controls.
- direction, align, justify, gap.
- padding/margin.
- background/border/radius.
- move before/after for flex/grid parents.

Input:

- value/text.
- placeholder.
- aria-label.
- font/surface/focus state.

Assets:

- list all detected assets with preview/type/name.
- paste URL and apply.
- upload image/SVG.
- choose from built-in SVG icon library.
- replace `img`, inline `svg`, or `background-image` based on asset id.

## Component Family Algorithm

- Prefer `data-component`, `data-design-component`, `data-ui`, `data-testid`, `data-test`.
- Then tag plus comparable classes.
- Then tag plus role.
- Then similar tag/class/child structure.
- Ignore inspector UI and invisible elements.
- Require same component kind.
- Limit broad fallback matches.
- Keep selected element first.

## Reset State

Store before the first edit:

- element reference.
- original style attribute.
- original textContent.
- original innerHTML.
- original form value.
- all original attributes and tracked null attributes.
- original computed values.
- original parent.
- original next sibling.

Reset must restore:

- style.
- text/value.
- innerHTML and attributes for asset changes.
- tracked attributes for attribute changes.
- original order for layout changes.

## Asset Replacement Algorithm

- Id assets as `img-N`, `svg-N`, and `bg-N`.
- `img`: set `src`, clear `srcset` and `sizes`.
- `svg`: parse SVG with `DOMParser`, remove scripts/foreignObject/event attributes, replace inner markup, preserve useful class/style/size/accessibility attributes.
- `background`: set `background-image` with escaped URL and `important`.
- SVG replacement can become a data URL for `img` and background targets.
- Keep change-log asset summaries short.

## Common Failure Modes

- Full selection overlay blocks nested child selection: set overlay `pointer-events: none`.
- Inspect edits accidentally: do not pass Design callbacks into Inspect.
- Component mode edits too much: tighten family matching.
- Reset does not restore assets: store `innerHTML` and all attributes.
- Asset changes pollute CSS diff: exclude `asset:*`.
- Drag feels broken in menus: reorder siblings, do not apply free `transform`.
- Color input fails for rgba/transparent: normalize to 6-digit hex or fallback.
- State cards feel like code: parse declarations into editable property/value rows.
- User cannot see new controls: confirm they are in the Design tab, not Inspect.
- Number becomes `4820` after typing `482`: stop committing on every keystroke and isolate a focused draft.
- Text loses spaces: keep raw text separate from normalized snapshot text.
- Space/arrow keys control the host page: stop tool-owned keyboard events before host window listeners.
- `oklch()` is reported as a brand token: reject unparseable colors instead of returning a fallback hex.
- Frame picking fails although the node exists: do not use parent-window `instanceof` for iframe nodes.
- Responsive preview recurses: strip inspector flags from the frame URL.
- Off-screen sweep hangs: use bounded timer settle points instead of relying on `requestAnimationFrame`.
- Saved session disappears on mount: do not clear storage from the initial empty effect or StrictMode replay.
- Resize jumps at 50% zoom: divide pointer movement by the preview scale.
- Resize log has the wrong before value: rollback preview styles before committing through the standard edit path.
- Left/top edge drifts: compensate the corresponding margin while resizing.
- Direct text edit destroys nested spans: refuse elements with child markup.
- State editor changes only the handoff code: inject marker-scoped rules into host and frame documents.
- Embedded Try button becomes selected instead of clicked: mark it as inspector passthrough.
- The device preview mounts another inspector: block tool mounting when `window.self !== window.top`.
- Overlay remains after route/slide replacement: require `snapshot.element.isConnected` before drawing.
- Detected palette unlocks token binding: distinguish heuristic sources from provided/static sources.
- Font picker offers project-specific names from another app: detect the consuming page's actual stacks.
- Opening the inspector blocks site navigation: add a Browse mode that unbinds global selection behavior.
- Comment pins drift after scrolling: derive positions from stored structural paths on scroll/resize.
