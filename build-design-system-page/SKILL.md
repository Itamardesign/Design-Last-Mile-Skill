---
name: build-design-system-page
description: Build or upgrade a standalone design-system source and reference page for React, Vite, Next.js, HTML/CSS/JS, or dashboard apps. Use when the user wants canonical portable tokens, multiple product collections, colors, spacing, radii, typography, interaction states, accessibility and responsive thresholds, reusable patterns, local token promotion/export, automatic/static token detection with provenance, an optional React token provider, governance rules, and integration with Inspect and Design tools.
---

# Build Design System Page

## Purpose

Create the canonical design-system source that the app, Inspect panel, and Design panel can all use.

The design system has two parts:

- token/source file: framework-light data such as colors, state names, typography recipes, reusable pattern classes, and alignment checklist
- reference page: visible UI that lets designers/developers inspect and copy tokens, states, patterns, snippets, and component recipes

For the complete standalone clone pattern, read `references/design-inspect-system-pattern.md`.

## Workflow

1. Read the target app and identify recurring colors, text styles, states, components, and UI patterns.
2. Create or update a central token file such as `src/designSystem.ts`.
3. Create or update a page component such as `src/components/DesignSystem.tsx`.
4. Mount the page through the app router or a simple route/hash such as `#design`.
5. Connect the handoff inspector to the design system:
   - Inspect component preview links to the page.
   - Inspect token chips compare snapshot colors against exported tokens.
   - Design tab color swatches use exported color tokens.
6. Verify the app, design system page, and inspector still agree after visual edits.
7. Add or update a project-level design-system rule so future UI work reuses the system and registers justified reusable additions.

## Required Source Structure

The token file should be importable from both product components and tooling panels.

Recommended exports:

- `brandTokens`: named color constants.
- `brandColorTokens`: array of `{ label, value, className }` for swatches and token matching.
- neutral/stone/gray scale tokens.
- state list such as `callStates` or product-specific states.
- `stateLabels`.
- state metadata such as descriptions, backgrounds, active classes, marker/glow/panel styles.
- component/character/theme variants if the product has visual stateful subjects.
- reusable pattern class strings such as chat bubbles, input bars, buttons, cards, headers.
- `alignmentChecklist`: short rules that keep future implementation aligned.
- tool visual tokens such as panel canvas, surfaces, borders, text, muted text, selection, focus, and success colors.
- spacing and radius scales used by product and token auditing.
- responsive breakpoint and named device presets.
- accessibility thresholds for contrast, large type, readable type, and touch targets.
- a versioned/namespaced custom-token contract and storage key when live token promotion is supported.
- `designSystemCollections` when one product contains multiple visual languages, such as a core brand and a guide-specific system.

Use TypeScript literal arrays/objects where possible so other components can infer valid state names.

Keep the source rendering-free. Product UI, catalog UI, inspectors, audits, previews, and tests should all import from it.

## Portable Token Contract

For a reusable inspector package, define generic host-owned shapes for color, spacing, radius, typography, collections, and token-source provenance. Keep inspector chrome/device/accessibility configuration in a separate module; those values belong to the tool, not the consuming brand.

Provide an optional `DesignTokensProvider` and `useDesignTokens` contract:

- explicit provider tokens are authoritative;
- statically detected config is authoritative enough for binding/audit;
- named live CSS variables preserve useful source names but remain heuristic;
- rendered-page detection fills colors and typography gaps;
- a page audit proposes spacing/radius/type/color drafts for human review;
- generic fallback keeps basic editing usable but never pretends to be the host's system.

Expose the active source in UI. Lock token binding, page audit, and All variants until the source is `provided` or `detected-static`; explain the unlock path on disabled controls.

For static detection, use a Node-only subpath that can read `design-tokens.json`, `tokens.json`, and importable Tailwind JS/CJS/MJS configuration. Do not pull `node:fs` into the browser entry. Treat Tailwind TypeScript config as requiring the consuming project's loader or a generated JSON bridge.

## Required Page Shape

The reference page should be a real product tool, not a marketing page.

Use compact tabs or side navigation. Recommended tabs:

- Character/States: every visual state and variant, with live preview, copyable snippets, and token swatches.
- Text: typography scale with sample text and class/CSS recipe.
- Color: brand tokens and neutral scale, click-copyable.
- Patterns: reusable component recipes with visual examples and copyable class strings.
- Overview: source-of-truth explanation, counts, governance checklist, and locally promoted tokens.

For another domain, rename tabs to fit the product but keep the roles:

- States/Variants
- Typography
- Color
- Patterns/Components

## UI Guidance

- Use a full-screen or full-panel tool layout.
- Keep side navigation dense and clear.
- Cards are for repeated state/pattern items.
- Show the actual component/state, not only a text description.
- Every token or recipe should be copyable.
- Use real state names from the source file.
- Avoid duplicating token values inside the page component; import them.
- Render real previews from the collection data instead of screenshots or decorative placeholders.
- Keep tool typography comfortably readable and controls at practical target sizes; dense does not mean tiny.
- Add subtle entrance/focus/hover feedback and honor `prefers-reduced-motion`.
- If the app is RTL, set `dir` deliberately and keep code snippets `dir="ltr"`.

## App Integration

Mount the page so the inspector can open it:

- simple React: maintain `isDesignSystemActive` and open from `#design`, `/design`, or `?design=true`
- router app: add `/design` route
- modal/shell app: render full-screen overlay and close back to app

The app should keep shared visual choices in state when needed. For example, if a character variant is chosen in the design system, pass that same variant into the app and inspector previews.

For query-driven prototypes, use `?system=<collection>` to deep-link to a collection and update the query with `history.replaceState` when switching libraries.

## Inspector Integration

The design system and inspector are related but separate:

- Design System defines the canonical tokens and patterns.
- Inspect reads the live DOM and shows which tokens match the selected element.
- Design edits the live DOM for exploration, using the same token arrays for swatches.
- Direct canvas editing derives snap increments from the shared spacing scale instead of hard-coded pixels.
- Responsive previews import named devices and accessibility checks import shared thresholds.
- Permanent changes should be made back in source components and/or `designSystem.ts`.
- Locally promoted tokens are provisional. Show them in the catalog, listen for same-tab custom events and cross-tab `storage` events, and provide paste-ready source export so they can graduate into the canonical file.
- Auto-detected/generated tokens are also provisional. Preserve named declarations, merge missing rendered values by category, and keep source provenance visible.

Inspect component preview should include a "Design system" link. Use a helper that opens the design route and prevents normal anchor navigation when needed.

## Clone Recipe

When asked to clone this system into another project:

1. Copy or recreate `designSystem.ts`.
2. Copy or recreate the design-system page UI.
3. Wire route/hash/query open and close behavior.
4. Import token arrays into the inspector.
5. Add token-match logic in Inspect.
6. Add token swatches in Design.
7. Verify that changing a token in `designSystem.ts` updates the design page and inspector controls.
8. Verify a promoted local token appears without a reload and that its export is valid source.

## Governance Rule

Add a short repository instruction such as `AGENTS.md` that applies to UI, layout, styling, and component tasks:

- reuse semantic tokens, typography recipes, states, patterns, and shared components first;
- allow new design when the existing system does not cover a real product need;
- when a decision is reusable, add it to the canonical source, document/preview it in the visible catalog, and consume the shared definition in product code;
- cover interaction, responsive, RTL, and accessibility states;
- avoid duplicated magic values; leave intentional one-offs local and explain why;
- treat design-system reuse/update verification as part of completion.

It is acceptable to copy the UI from an existing implementation when the target stack is compatible. Adapt imports, types, router, direction, and styling conventions.

## Verification

Run:

```bash
npx tsc --noEmit
npm run build
```

Browser-check:

- design-system route opens from direct URL/hash/query.
- page can close back to the app.
- all tabs render.
- token copy buttons work.
- state previews render every state/variant.
- pattern examples render using exported classes.
- inspector link opens the page.
- inspector token chips match known colors.
- Design tab swatches use the same token array.
- collection deep links open the expected library.
- local token events refresh the catalog without reload and cross-tab storage changes also refresh it.
- breakpoint, device, spacing, radius, and accessibility values are imported by tooling rather than duplicated.
- canvas resize snapping visibly follows the exported spacing scale.
- all new reusable values are in the canonical source and visible catalog; intentional one-offs have a rationale.
- provided/static/live/generated/fallback sources resolve in the documented priority and canonical-only controls remain locked for heuristic sources.
