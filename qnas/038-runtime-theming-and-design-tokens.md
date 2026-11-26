# 038 - Runtime Theming and Design Tokens: Q&A

## Token foundations

### 1. What is a design token?

A design token is a named value for a reusable design decision, such as a text color, spacing step, radius, or type size. It lets multiple components share the same decision and lets themes change the value without rewriting every component rule.

### 2. What is the difference between a palette token and a semantic token?

A palette token describes a raw value, such as <code>--palette-blue-700</code>. A semantic token describes the job that value performs, such as <code>--color-link</code> or <code>--color-text-primary</code>. A theme can change the palette value assigned to a semantic role while keeping component rules stable.

### 3. Should components use palette tokens directly?

Usually, components should use semantic tokens. A link can use <code>--color-link</code> rather than knowing it currently uses a blue palette value. Direct palette usage is reasonable for design-system internals or a decorative detail that has no semantic role.

### 4. What is a component token?

A component token is a variable scoped to one component or component family, such as <code>--button-background</code>. It can be useful for variants or a documented customization API. It is unnecessary when it merely adds an alias without a clear use.

### 5. Must every token system have primitive, semantic, and component layers?

No. These are useful ways to organize values, not mandatory layers. A small project may need only semantic tokens. Add a layer when it clarifies ownership, theme mapping, or component customization.

### 6. Why prefer role-based names over appearance-based names?

Appearance-based names such as <code>--dark-gray-text</code> describe the current palette and can become misleading when themes change. Role-based names such as <code>--color-text-primary</code> preserve intended use while the actual value changes.

### 7. Should every repeated CSS value become a token?

No. A token is useful when the value represents a shared decision, needs a theme mapping, or should be centrally configurable. One-off values can remain local; needless tokens add names and maintenance work.

## Sass and generated tokens

### 8. What is Sass doing in a token system?

Sass can organize build-time source values in variables or maps, generate repeated declarations, and emit CSS custom properties. It can reduce repetitive source code, but the browser only receives the compiled CSS.

### 9. Can JavaScript read a Sass map at runtime?

No. Sass compiles before the page runs, so the map is not present in the browser. If runtime CSS or JavaScript needs a value, emit it as CSS or provide it through another runtime data source.

### 10. Does a Sass map become CSS automatically?

No. A map is a Sass value, not a CSS value. A loop or other Sass code must turn its entries into declarations.

### 11. Does the lesson's Sass @each need @use "sass:map"?

No. Sass's @each syntax can iterate over a map directly. Load the sass:map module when calling its functions, such as map.get() or map.merge().

### 12. Should all raw palette values be emitted as public CSS properties?

Not necessarily. Emit values that CSS, themes, components, or runtime code need. Keeping unused values in the output can make the token surface larger without helping consumers.

### 13. Can a Sass variable and a CSS custom property be used together?

Yes. Sass can generate a default custom-property declaration, and the browser can later cascade or inherit that custom property at runtime. This uses Sass at build time and CSS at runtime.

## Theme behavior

### 14. How does a CSS custom property help with theming?

A theme can set semantic custom properties at a root or container boundary. Descendant component rules can keep reading the same names while the cascade supplies different values.

### 15. Should theme values be repeated in every component selector?

Usually no. Set shared semantic tokens at a root or theme container and let them inherit. Repeat a component-specific override only when that component has a real variant or exception.

### 16. Can a theme apply only to part of a page?

Yes. Put the theme's custom-property declarations on a wrapper around the themed subtree. Descendants inherit those values, so the rest of the document can keep its existing theme.

### 17. Why might a nested palette override fail to update a semantic alias?

Custom-property values that contain var() references are resolved as part of computing the custom property on the element where it is declared. If a root semantic token was computed from a root palette value and then inherited, changing only the palette token on a descendant may not recalculate that inherited semantic value there. Set or reassign the semantic token at the theme boundary as well.

### 18. How do prefers-color-scheme and data-theme differ?

prefers-color-scheme is a media feature that reflects a user's preferred light or dark scheme. An attribute such as data-theme="dark" is application state that can represent an explicit user choice. A design can let explicit choices override the system preference.

### 19. What does a system theme mode mean?

It means the app has not forced light or dark and lets CSS follow the user's current system preference. Store the mode as system, rather than storing a resolved palette that will become stale if the preference changes.

### 20. How should system, light, and dark choices be represented?

A common approach is a validated value such as system, light, or dark, reflected in a root data-theme attribute. CSS owns the colors; JavaScript only changes the selected mode.

### 21. What does color-scheme do?

It tells the browser which color schemes an element supports. This can affect browser-provided UI such as form controls and scrollbars. It does not generate the page's own semantic palette.

### 22. Is color-scheme: light dark enough to create a dark theme?

No. It informs the browser about supported schemes, but author-styled content still needs suitable foreground, background, border, link, and state colors. Define those with theme tokens.

### 23. Why should native controls be checked in each theme?

Controls can use browser-provided colors and may otherwise look inconsistent with authored surfaces. Set an appropriate color-scheme for the selected theme, then inspect controls in target browsers.

### 24. Should JavaScript store every theme color?

Usually no. Keep color values in CSS and let JavaScript choose a theme mode. Duplicating palettes in both CSS and JavaScript creates two sources of truth that can drift.

### 25. What should a theme preference store?

Store the user's mode, such as light, dark, or system. Validate stored data before applying it. Do not assume storage is always available; access can fail in some environments.

### 26. What causes a flash of the wrong theme on page load?

The page may first render its CSS default and only later apply a saved explicit mode from JavaScript. To avoid that, make the initial mode available before the main stylesheet is applied or render it from a server-managed preference. Keep any early script compatible with the site's Content Security Policy.

### 27. Should the page still have a useful appearance without JavaScript?

Yes. CSS can provide readable default tokens and respond to prefers-color-scheme without JavaScript. A theme selector can enhance the page, but the basic page should remain readable if its script does not run.

## Accessibility and verification

### 28. What contrast should normal text meet under WCAG 2.2 SC 1.4.3?

The general minimum is 4.5:1 for normal text. Large text has a 3:1 minimum. Check the actual foreground and background pair in each theme; a token name does not establish compliance.

### 29. Does a successful light theme guarantee that the dark theme is accessible?

No. Each theme creates different color pairs. Check body text, secondary text, links, controls, boundaries, status colors, and focus indicators on the surfaces where they actually appear.

### 30. Is color alone enough to communicate status?

No. Pair color with text, an icon, shape, or another visible signal so the information does not depend on color perception alone.

### 31. What is forced colors mode?

It is a user-agent mode that applies a user-selected color palette to improve readability. Most page content should let the browser make these adjustments. Check the page in this mode instead of assuming authored colors will remain unchanged.

### 32. Should a design system use forced-color-adjust: none globally?

No. That prevents user-agent color adjustments for the affected content. Use it only on a specific element when you provide and verify an accessible alternative. Prefer allowing the browser's forced palette to work.

### 33. Why use system colors such as CanvasText or Highlight?

They refer to colors from the active user-agent palette. They can help custom borders or focus indicators remain visible when users apply a forced color scheme.

### 34. What should happen to keyboard focus in every theme?

Keep the focus indicator clearly visible against each relevant surface. Do not remove the outline without providing a visible replacement, and verify ordinary themes and forced colors mode.

### 35. Which states should be checked when reviewing a token palette?

Check default, hover, active, focus, disabled, selected, and error states where a component supports them. Also inspect adjacent surfaces, since a state color may be used in more than one context.

## Maintenance

### 36. Should design-token names be treated as an API?

If other stylesheets, components, or teams rely on them, yes. Keep names stable, document intended meaning, and treat renames or semantic changes as migrations rather than casual cleanup.

### 37. How can duplicate tokens be reduced?

Search for tokens with the same meaning and value, decide which name best expresses the role, and migrate consumers before removing the duplicate. Keep intentionally different roles separate even if they happen to share a value today.

### 38. How can themes avoid becoming hard to maintain?

Keep a consistent set of semantic token names across themes, set them at clear boundaries, and test the same component states in each theme. Avoid theme-specific selectors scattered throughout component styles.

### 39. Should dark mode be made by mechanically inverting light colors?

No. Dark surfaces and readable foregrounds often need deliberate values. Review each semantic role, border, shadow, image, and interaction state in context.

### 40. How can a team review a token change safely?

Identify its consumers, inspect every theme and state that uses it, check color contrast and focus, and verify component customization points. Change one semantic decision at a time when practical so visual regressions are easier to trace.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="037-sass-variables-vs-css-custom-properties.md">037 - Sass Variables versus CSS Custom Properties: Q&amp;A</a></td><td><a href="../lessons/038-runtime-theming-and-design-tokens.md">Lesson</a></td><td><a href="../examples/038-runtime-theming-and-design-tokens.md">Example</a></td><td><a href="039-reusable-responsive-mixins.md">039 - Reusable Responsive Mixins: Q&amp;A</a></td></tr>
  </tbody>
</table>
