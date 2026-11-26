# 038 — Runtime Theming and Design Tokens

## Learning goals

After this lesson, you should be able to:

- Explain what a design token represents and why semantic names help.
- Separate palette values, semantic tokens, and component tokens.
- Use Sass maps to generate a maintainable set of CSS custom properties.
- Build light, dark, and system themes with CSS custom properties.
- Apply a theme to the whole page or to a nested preview region.
- Connect a theme control to CSS without duplicating theme values in JavaScript.
- Keep contrast, focus, forced-colors behavior, and no-JavaScript defaults in mind.
- Choose which values belong in Sass at build time and which belong in CSS at runtime.

## 1. What a design token is

A design token is a named value that represents a reusable design decision, such as a surface color, spacing step, border radius, or type size. A token gives a decision one name that can be reused across components and themes.

A useful token system has clear levels:

| Level | Example | Purpose |
| --- | --- | --- |
| Primitive or palette token | `--palette-blue-700` | Stores a raw value from a palette or scale |
| Semantic token | `--color-text-primary` | Describes the role the value plays in the interface |
| Component token | `--button-background` | Names a component-specific choice or override |

The names describe different things. `blue-700` describes a color value. `text-primary` describes how the interface uses a value. A dark theme can keep the semantic name while assigning it a different palette token.

Tokens can also represent non-color values:

~~~css
:root {
  --space-2: 0.5rem;
  --space-4: 1rem;
  --radius-control: 0.375rem;
  --font-size-body: 1rem;
}
~~~

A token should have a clear reason to exist. Avoid creating multiple aliases that all mean the same thing without helping the design system.

## 2. Separate raw values from semantic roles

A palette token records a raw value. A semantic token maps that value to an interface role.

~~~css
:root {
  --palette-blue-700: #194b9b;
  --palette-gray-950: #171a20;
  --palette-white: #ffffff;

  --color-link: var(--palette-blue-700);
  --color-page: var(--palette-white);
  --color-text-primary: var(--palette-gray-950);
}
~~~

Components use the semantic names:

~~~css
body {
  color: var(--color-text-primary);
  background: var(--color-page);
}

a {
  color: var(--color-link);
}
~~~

This indirection lets a theme change the palette assignment while components continue to express their intent. A component that needs a warning color should use a warning role, not guess which orange value is currently appropriate.

## 3. Use Sass maps to generate a palette

Sass can keep build-time source values in a map and emit CSS tokens from them.

~~~scss
$palette: (
  "blue-700": #194b9b,
  "blue-100": #dce8fb,
  "gray-950": #171a20,
  "gray-100": #f0f2f5,
  "white": #ffffff
);

:root {
  @each $name, $value in $palette {
    --palette-#{$name}: #{$value};
  }
}
~~~

The browser receives ordinary CSS custom properties. The Sass map is not available to JavaScript or CSS after compilation.

Generate only values that the system actually uses. A very large map can be as hard to understand as a long list of handwritten declarations. Keep token names stable and describe the intended role in documentation.

## 4. Define semantic tokens for the default theme

Start by assigning semantic roles to palette values.

~~~css
:root {
  --color-page: var(--palette-white);
  --color-surface: var(--palette-gray-100);
  --color-text-primary: var(--palette-gray-950);
  --color-link: var(--palette-blue-700);
  --color-border: #687386;
}
~~~

A component can now depend on the role rather than the specific palette value:

~~~css
.card {
  color: var(--color-text-primary);
  background: var(--color-surface);
  border: 1px solid var(--color-border);
}

.card a {
  color: var(--color-link);
}
~~~

This default can serve as the light theme. A reset or design system may choose a different default, but there should be one predictable fallback when no theme selector or media query applies.

## 5. Add a dark theme with the same semantic names

Override semantic tokens at a theme boundary:

~~~css
:root[data-theme="dark"] {
  color-scheme: dark;

  --color-page: #111318;
  --color-surface: #20242c;
  --color-text-primary: #f2f4f8;
  --color-link: #a9c7ff;
  --color-border: #798397;
}
~~~

The `.card` rules do not change. The browser cascades the new custom-property values on the root and descendants inherit them.

Keep semantic names stable between themes. `--color-text-primary` should remain the main readable text color in both themes; the actual color value changes.

## 6. Respect the operating-system preference

The `prefers-color-scheme` media feature lets CSS respond to a user's preferred light or dark scheme. The `color-scheme` property also tells the browser which schemes an element supports, affecting browser-provided UI such as form controls and scrollbars.

~~~css
:root {
  color-scheme: light;
  --color-page: #ffffff;
  --color-surface: #f0f2f5;
  --color-text-primary: #171a20;
  --color-link: #194b9b;
  --color-border: #687386;
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]):not([data-theme="dark"]) {
    color-scheme: dark;
    --color-page: #111318;
    --color-surface: #20242c;
    --color-text-primary: #f2f4f8;
    --color-link: #a9c7ff;
    --color-border: #798397;
  }
}
~~~

Here, an explicit `data-theme="light"` or `data-theme="dark"` choice can take priority over the media preference. With no explicit choice, the media query supplies a dark palette when the user prefers dark.

A simple version can set `color-scheme: light dark` when the page genuinely supports both schemes and wants browser controls to follow the user's preference. When the page has an explicit theme control, set the used scheme for each selected theme so native controls match the page.

`color-scheme` does not create a complete theme for author-styled content. Define your own page, text, surface, border, and interactive tokens.

## 7. Model system, light, and dark as explicit modes

An app can represent the user's choice as one of three modes:

- `system`: let the browser preference choose the palette.
- `light`: force the light palette.
- `dark`: force the dark palette.

A document can start in system mode:

~~~html
<html lang="en" data-theme="system">
~~~

The CSS can then define the default light values and the system dark override:

~~~css
:root {
  color-scheme: light;
  --color-page: #ffffff;
  --color-surface: #f0f2f5;
  --color-text-primary: #171a20;
  --color-link: #194b9b;
  --color-border: #687386;
}

:root[data-theme="light"] {
  color-scheme: light;
  --color-page: #ffffff;
  --color-surface: #f0f2f5;
  --color-text-primary: #171a20;
  --color-link: #194b9b;
  --color-border: #687386;
}

:root[data-theme="dark"] {
  color-scheme: dark;
  --color-page: #111318;
  --color-surface: #20242c;
  --color-text-primary: #f2f4f8;
  --color-link: #a9c7ff;
  --color-border: #798397;
}

@media (prefers-color-scheme: dark) {
  :root[data-theme="system"] {
    color-scheme: dark;
    --color-page: #111318;
    --color-surface: #20242c;
    --color-text-primary: #f2f4f8;
    --color-link: #a9c7ff;
    --color-border: #798397;
  }
}
~~~

The repeated declarations make the modes easy to see, but they can become tedious for a large theme. Keep the final CSS behavior understandable when debugging in browser developer tools.

## 8. Apply a theme to one subtree

Because custom properties inherit, a theme can be scoped to a region rather than the whole page.

~~~css
.theme-preview {
  --color-page: #ffffff;
  --color-surface: #f0f2f5;
  --color-text-primary: #171a20;
}

.theme-preview[data-theme="dark"] {
  --color-page: #111318;
  --color-surface: #20242c;
  --color-text-primary: #f2f4f8;
}

.theme-preview__panel {
  color: var(--color-text-primary);
  background: var(--color-surface);
}
~~~

This is useful for component previews, embedded widgets, or side-by-side theme demonstrations. Put the theme values on the element that contains the themed content so every descendant reads the same token set.

Avoid adding a dark selector to every component. That couples theme choice to component styling and makes new components easy to miss.

## 9. Add component tokens only when they help

A component can alias semantic tokens to give itself a documented extension point:

~~~css
.button {
  --button-background: var(--color-action);
  --button-foreground: var(--color-on-action);

  color: var(--button-foreground);
  background: var(--button-background);
  border-radius: var(--radius-control);
}
~~~

A variant can override the component tokens:

~~~css
.button--quiet {
  --button-background: transparent;
  --button-foreground: var(--color-text-primary);
}
~~~

Use component tokens when a component needs a local variant or public customization point. If every component simply repeats a semantic token once, the extra alias may add indirection without adding value.

Do not make component tokens the only way to define core semantic roles. Keep the high-level theme understandable independently of specific component selectors.

## 10. Keep JavaScript focused on the user's choice

JavaScript should choose a mode, not contain a second copy of every theme color.

~~~html
<label for="theme-choice">Color theme</label>
<select id="theme-choice" name="theme">
  <option value="system">Use device setting</option>
  <option value="light">Light</option>
  <option value="dark">Dark</option>
</select>
~~~

~~~js
const root = document.documentElement;
const themeChoice = document.querySelector("#theme-choice");
const allowedThemes = new Set(["system", "light", "dark"]);

themeChoice.addEventListener("change", () => {
  const selectedTheme = themeChoice.value;

  if (!allowedThemes.has(selectedTheme)) {
    return;
  }

  root.dataset.theme = selectedTheme;
});
~~~

CSS owns the theme values; JavaScript changes only the `data-theme` state. Keep the control's selected value in sync with the current mode when restoring a saved preference.

A real app may persist the mode, but it should validate stored values and handle storage being unavailable. Store the mode (`system`, `light`, or `dark`), not a snapshot of the resolved color palette. A system mode should continue to follow later operating-system preference changes.

## 11. Avoid a flash when restoring a saved theme

If a site restores a saved explicit theme only after the page has painted, users may briefly see the default palette. For a small page, that may be acceptable. For a polished app, make the initial mode available before the main stylesheet is applied, or render it from a server-managed preference.

Keep any early initialization short and defensive:

~~~html
<script>
  try {
    const savedTheme = localStorage.getItem("theme");
    const allowedThemes = ["system", "light", "dark"];

    if (allowedThemes.includes(savedTheme)) {
      document.documentElement.dataset.theme = savedTheme;
    }
  } catch {
    // CSS still provides the system-preference default.
  }
</script>
<link rel="stylesheet" href="/styles.css">
~~~

If inline scripts are restricted by a Content Security Policy, use the site's approved nonce or an external script loaded early. Do not weaken the policy just to remove a small visual flash.

## 12. Test contrast and interaction states in every theme

A theme is a complete set of visual relationships, not merely a dark background. Check at least:

- Main text against the page background.
- Secondary text against its actual surface.
- Links and visited links.
- Buttons in default, hover, active, disabled, and focus states.
- Borders, separators, icons, and form controls.
- Validation messages, selected states, and status colors.
- Focus indicators against every surface they may appear on.

For WCAG 2.2 Success Criterion 1.4.3, normal text generally needs a contrast ratio of at least 4.5:1; large text has a 3:1 threshold. Evaluate the actual foreground and background pair used in each theme. A token name like `--color-text-muted` does not guarantee sufficient contrast.

Do not communicate meaning only through color. Pair status colors with text, icons, shape, or another visible cue.

## 13. Preserve user agent adaptations

Forced colors mode lets users apply a high-contrast palette. Avoid disabling it across the page. Keep visible focus indicators and prefer system colors when custom drawing is necessary.

~~~css
@media (forced-colors: active) {
  .status-dot {
    border: 1px solid CanvasText;
    background: Canvas;
  }

  :focus-visible {
    outline: 2px solid Highlight;
    outline-offset: 3px;
  }
}
~~~

Most ordinary text and controls should be allowed to use the browser's forced palette. Use `forced-color-adjust: none` only for a specific element when you are deliberately supplying an accessible replacement and have verified it in that mode.

## 14. Keep the no-JavaScript behavior useful

The page should still have a readable initial theme without JavaScript. CSS can follow the operating-system preference directly, even if the interactive mode switch is unavailable.

If an interface requires JavaScript for changing the theme, the selected mode should still be visible and understandable. Avoid rendering an inactive control that appears to work but has no behavior.

## 15. Common design-token mistakes

### Naming by current appearance

Names such as `--dark-gray-text` or `--blue-button` make theme changes harder. Prefer role names such as `--color-text-primary` and `--color-action`.

### Putting all values in one global bucket

A raw palette, semantic roles, and component decisions have different jobs. Separate them so it is clear which layer a theme is expected to override.

### Duplicating values in Sass, CSS, and JavaScript

Choose one source of truth for each decision. Sass can generate base declarations, CSS can assign runtime themes, and JavaScript can select a mode. Do not copy the same palette into three languages.

### Overriding every component

Theme tokens should usually be set at a root or container boundary and inherited. Per-component overrides are for genuine exceptions or variants.

### Treating dark mode as an inverted light mode

Dark surfaces, borders, shadows, images, and emphasis often need deliberate choices. Verify each semantic role rather than applying a blanket filter or mechanically swapping black and white.

### Omitting browser-provided controls

Use `color-scheme` to tell the browser which schemes the page supports. Check form controls and scrollbars as well as authored elements.

## 16. A practical workflow

1. List the interface roles the product actually needs.
2. Create a small palette or Sass map for raw values.
3. Map palette values to semantic tokens for the default theme.
4. Override the same semantic names in other themes.
5. Add component tokens only for clear variants or customization needs.
6. Set the theme at the root or on a scoped container.
7. Let CSS respond to system preference when the mode is `system`.
8. Let JavaScript update the mode only when the user chooses.
9. Review contrast, focus, controls, and forced colors in every theme.
10. Remove tokens that have no meaningful role or use.

## Quick reference

~~~css
:root {
  color-scheme: light;
  --color-page: #ffffff;
  --color-surface: #f0f2f5;
  --color-text-primary: #171a20;
  --color-link: #194b9b;
}

:root[data-theme="dark"] {
  color-scheme: dark;
  --color-page: #111318;
  --color-surface: #20242c;
  --color-text-primary: #f2f4f8;
  --color-link: #a9c7ff;
}

.card {
  color: var(--color-text-primary);
  background: var(--color-surface);
}
~~~

- Sass maps organize values at build time.
- Semantic CSS custom properties express roles at runtime.
- `data-theme` selects explicit application modes.
- `prefers-color-scheme` represents a user preference.
- `color-scheme` affects browser-provided UI; it does not author the page's full palette.
- Test every important color pair and interaction state in each theme.

## Related material

- [Previous lesson: Sass variables versus CSS custom properties](037-sass-variables-vs-css-custom-properties.md)
- [Q&A: Runtime theming and design tokens](../qnas/038-runtime-theming-and-design-tokens.md)
- [Examples: Runtime theming and design tokens](../examples/038-runtime-theming-and-design-tokens.md)
- [Next lesson: Reusable responsive mixins](039-reusable-responsive-mixins.md)
- [Sass: @use modules](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: Maps](https://sass-lang.com/documentation/values/maps/)
- [CSS Custom Properties for Cascading Variables](https://www.w3.org/TR/css-variables/)
- [CSS Color Adjustment Module](https://www.w3.org/TR/css-color-adjust-1/)
- [WCAG 2.2: Contrast Minimum](https://www.w3.org/TR/WCAG22/#contrast-minimum)
