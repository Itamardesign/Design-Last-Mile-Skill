# Design System Page Spec

Use this spec when creating a portable design-system source and reference page.

## Contents

- [Canonical source and page](#source-file)
- [Tabs and multiple collections](#tabs)
- [Routing and copy behavior](#routing)
- [Local token promotion, detection, and inspector integration](#local-token-promotion)
- [Governance, visual style, and verification](#project-governance)

## Source File

Create a source file such as `src/designSystem.ts`.

Recommended sections:

- types for state/theme objects.
- `brandTokens`: named constants.
- `brandColorTokens`: array for UI swatches and inspector token matching.
- neutral scale tokens.
- product states: `callStates`, `stateLabels`, `previewVolumeByState`, or domain-specific equivalents.
- state UI metadata: background, shell class, marker class, glow class, panel class, active class.
- variant themes: character/product/component themes per state.
- reusable pattern classes.
- alignment checklist.
- inspector/tool visual tokens.
- spacing and radius scales.
- responsive breakpoint and named-device presets.
- accessibility thresholds.
- custom-token type plus a namespaced storage/event contract.
- collection metadata when multiple visual systems coexist.

The source file should not render UI. It should be safe to import from app components, design-system page, inspector, sandbox, and tests.

## Reference Page

Create a component such as `DesignSystem.tsx`.

Suggested props:

```ts
type DesignSystemProps = {
  onClose: () => void;
  characterType?: CharacterType;
  onChangeCharacterType?: (value: CharacterType) => void;
};
```

Adjust names for the target product. The important part is that shared visual choices can be previewed and copied.

## Tabs

States/Character:

- show active variant selector.
- render every state for every variant.
- show live visual preview.
- copy component snippet.
- show/copy state token colors.

Text:

- show typography samples.
- display class string or CSS recipe in a code row.

Color:

- render brand tokens and neutral tokens.
- each token card shows swatch, label, and value.
- click copies hex/value.

Patterns:

- render reusable components as they appear in product UI.
- list copyable class strings or component snippets.
- include common patterns: bubbles, inputs, buttons, icon buttons, cards, navigation, action clusters.

Overview:

- explain which source file is canonical.
- summarize token/style/state/pattern counts.
- show the alignment/governance checklist.
- list locally promoted tokens with copyable values.

## Multiple Collections

When the product contains a distinct sub-product or guide language:

- model each library as a `DesignSystemCollection` with colors, neutrals, typography, states, patterns, checklist, and accent;
- select the collection via a compact library switch;
- support a deep link such as `?system=guide`;
- keep shared inspector/device/accessibility foundations outside individual collections.

## Routing

Provide one reliable way for Inspect to open the page:

- `#design`
- `/design`
- `?design=true`

If using hash/query in a prototype:

- listen to `hashchange` and `popstate`.
- close by cleaning the hash/query/path.
- keep page as a full-screen overlay so it can be copied into another project without router setup.

## Copy Behavior

- Use `navigator.clipboard.writeText`.
- Show copied state for around 1200ms.
- Keep code snippets LTR.
- Token cards should copy the raw value, not the label.

## Local Token Promotion

- Persist provisional tokens under one exported, namespaced storage key.
- Include name, category, property, value, id, and creation time.
- Refresh the page from a same-tab custom event and the cross-tab `storage` event.
- Show promoted tokens in the Overview and make values copyable.
- Treat them as provisional until moved into the canonical source file.
- Provide a paste-ready export containing typed token arrays and CSS custom-property declarations.

## Detection And Provenance

Use a generic `DesignTokens` contract plus a `DesignTokensSource` union such as:

- `provided`
- `detected-static`
- `detected-live-css`
- `generated-audit`
- `fallback-default`

Resolve per category instead of letting one weak source replace everything:

1. Prefer provider/static values and their names.
2. Read same-origin `:root` custom properties and bucket color/spacing/radius values.
3. Merge rendered role-aware colors not already represented.
4. Always derive typography from full computed signatures: family, size, weight, line-height, letter-spacing, and transform.
5. Generate clustered spacing/radius drafts only for missing categories.
6. Fall back to generic tokens only when the page yields nothing useful.

Run live detection after mount so SSR and first paint remain safe. Limit sample counts and ignore inspector-owned nodes. Generated audit output is a human-review draft, never an unattended source write.

Keep Node-only static loaders outside the browser entry. Support JSON token files and importable Tailwind JS/CJS/MJS; require a host-side loader/JSON bridge for TypeScript configs.

## Relationship To Inspector

Export tokens used by inspector:

- `brandColorTokens` for token-match chips in Inspect and swatches in Design.
- pattern classes for copyable recipes.
- state labels/themes for state preview cards.

The inspector should not own design-system data. If a token changes, update `designSystem.ts`, then let the app, page, and inspector reflect it.

Tooling should also import shared breakpoint/device presets, spacing/radius scales, visual tokens, and accessibility thresholds from the same source.

If direct canvas editing is supported, derive its snap step from the exported spacing scale. Do not create a separate inspector-only grid constant.

## Project Governance

Add a repository-level instruction for future UI work:

- use existing tokens/components first;
- register justified reusable additions in both canonical source and visible catalog;
- consume the shared definition in product code;
- include interaction, responsive, RTL, and accessibility states;
- explain intentionally local one-off values;
- verify system reuse before declaring UI work complete.

## Visual Style

- Tool-like, not a marketing landing page.
- Dense and scannable.
- Cards radius around 8px unless the app system differs.
- Avoid nested decorative cards.
- Full-width tool canvas or constrained content area.
- Use actual product states/components as visuals.

## Verification

- Direct route opens.
- Close returns to app.
- Every tab renders from imported tokens.
- Copy works.
- State/variant previews match app components.
- Inspector link opens this page.
- Changing a token updates the page and inspector swatches.
- Collection deep links and switching work.
- Promoted tokens appear without reload and across tabs.
- Tooling imports spacing, radius, breakpoint/device, and accessibility definitions.
- Direct-manipulation snapping uses the shared spacing scale.
- Token provenance is visible and heuristic sources cannot unlock canonical binding/audit/component-scope actions.
- New reusable patterns appear in both source and visible previews.
