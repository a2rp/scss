# 038 - Runtime Theming and Design Tokens: Examples

These examples use Sass for build-time token generation and CSS custom properties for values the browser can change at runtime. The sample colors illustrate the structure; check each actual foreground/background pair in your product before shipping.

## 1. Generate palette tokens from a Sass map

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

Dart Sass emits CSS similar to:

~~~css
:root {
  --palette-blue-700: #194b9b;
  --palette-blue-100: #dce8fb;
  --palette-gray-950: #171a20;
  --palette-gray-100: #f0f2f5;
  --palette-white: #ffffff;
}
~~~

The map and loop are Sass build-time tools. The emitted custom properties remain available to browser styles.

## 2. Map palette values to semantic roles

~~~css
:root {
  --palette-blue-700: #194b9b;
  --palette-gray-950: #171a20;
  --palette-gray-100: #f0f2f5;
  --palette-white: #ffffff;

  --color-page: var(--palette-white);
  --color-surface: var(--palette-gray-100);
  --color-text-primary: var(--palette-gray-950);
  --color-link: var(--palette-blue-700);
}

body {
  color: var(--color-text-primary);
  background: var(--color-page);
}

.card {
  color: var(--color-text-primary);
  background: var(--color-surface);
}

a {
  color: var(--color-link);
}
~~~

Components consume role names. A later theme can change the values assigned to the semantic names.

## 3. Override semantic roles for dark mode

~~~css
:root {
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

.card {
  color: var(--color-text-primary);
  background: var(--color-surface);
  border: 1px solid var(--color-border);
}

.card a {
  color: var(--color-link);
}
~~~

The component rules stay the same. The root theme changes the semantic token values inherited by the card.

## 4. Follow the system preference by default

~~~html
<html lang="en" data-theme="system">
~~~

~~~css
:root {
  color-scheme: light;
  --color-page: #ffffff;
  --color-surface: #f0f2f5;
  --color-text-primary: #171a20;
  --color-link: #194b9b;
}

@media (prefers-color-scheme: dark) {
  :root[data-theme="system"] {
    color-scheme: dark;
    --color-page: #111318;
    --color-surface: #20242c;
    --color-text-primary: #f2f4f8;
    --color-link: #a9c7ff;
  }
}
~~~

With the system mode selected, changing the operating-system preference lets the browser use the matching token set. The page has a light default if the media query does not match.

## 5. Allow an explicit light or dark choice to override system mode

~~~css
:root[data-theme="light"] {
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
~~~

The system preference rule should target only the system mode. Explicit theme selectors should set their own values and scheme. This gives the user's explicit selection a clear CSS representation.

## 6. Theme a nested preview without changing the page

~~~html
<section class="preview" data-theme="dark">
  <article class="preview__card">
    <h2>Dark preview</h2>
    <p>The surrounding page can keep its own theme.</p>
  </article>
</section>
~~~

~~~css
.preview {
  --color-page: #ffffff;
  --color-surface: #f0f2f5;
  --color-text-primary: #171a20;
}

.preview[data-theme="dark"] {
  --color-page: #111318;
  --color-surface: #20242c;
  --color-text-primary: #f2f4f8;
}

.preview__card {
  color: var(--color-text-primary);
  background: var(--color-surface);
}
~~~

The values are scoped to the preview wrapper. Its descendants inherit them; the rest of the document does not.

## 7. Add component tokens for a real variant

~~~css
.button {
  --button-background: #194b9b;
  --button-foreground: #ffffff;

  color: var(--button-foreground);
  background: var(--button-background);
  border: 1px solid var(--button-background);
}

.button--quiet {
  --button-background: transparent;
  --button-foreground: var(--color-text-primary);
  border-color: var(--color-border);
}
~~~

A component token is useful here because the quiet variant has its own documented component-level choice. Check the contrast and hover, active, disabled, and focus states for both variants.

## 8. Build a theme selector that updates CSS state

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

JavaScript sets one mode. CSS keeps the palette definitions, so theme colors do not have to be duplicated in the script.

## 9. Persist a validated mode and handle storage errors

This script runs after the selector exists:

~~~js
const root = document.documentElement;
const themeChoice = document.querySelector("#theme-choice");
const allowedThemes = new Set(["system", "light", "dark"]);

let savedTheme = "system";

try {
  const storedTheme = localStorage.getItem("theme");

  if (allowedThemes.has(storedTheme)) {
    savedTheme = storedTheme;
  }
} catch {
  // Storage may be unavailable. Keep the system default.
}

root.dataset.theme = savedTheme;
themeChoice.value = savedTheme;

themeChoice.addEventListener("change", () => {
  const selectedTheme = themeChoice.value;

  if (!allowedThemes.has(selectedTheme)) {
    return;
  }

  root.dataset.theme = selectedTheme;

  try {
    localStorage.setItem("theme", selectedTheme);
  } catch {
    // Theme still changes for this page view.
  }
});
~~~

This version is easy to follow, but applying the saved theme after the page has painted can cause a brief flash. For first-paint consistency, read and validate the saved mode before the main stylesheet is loaded, or render the preference from the server.

## 10. Apply a saved mode before the main stylesheet

Place a small initialization script in the document head before the stylesheet:

~~~html
<script>
  try {
    const savedTheme = localStorage.getItem("theme");
    const allowedThemes = ["system", "light", "dark"];

    if (allowedThemes.includes(savedTheme)) {
      document.documentElement.dataset.theme = savedTheme;
    }
  } catch {
    // The stylesheet still follows the system preference.
  }
</script>
<link rel="stylesheet" href="/styles.css">
~~~

Keep the value allowlist small. If the site's Content Security Policy disallows inline scripts, use an approved nonce or an early external script instead of weakening the policy.

## 11. Keep the theme selector usable without JavaScript

The root attribute in the HTML supplies a safe default:

~~~html
<html lang="en" data-theme="system">
~~~

The CSS media query can still follow the user's preference if the theme script fails. The selector itself will not change themes without JavaScript, so do not present it as interactive unless its behavior is available.

## 12. Match native controls to the selected theme

~~~css
:root[data-theme="light"] {
  color-scheme: light;
  --color-page: #ffffff;
  --color-text-primary: #171a20;
}

:root[data-theme="dark"] {
  color-scheme: dark;
  --color-page: #111318;
  --color-text-primary: #f2f4f8;
}

body {
  color: var(--color-text-primary);
  background: var(--color-page);
}

input,
select,
textarea,
button {
  font: inherit;
}
~~~

The color-scheme declaration lets the browser choose matching colors for browser-provided controls. It does not set the colors of the authored body and components; those still need their own tokens.

## 13. Keep a visible focus indicator across themes

~~~css
:focus-visible {
  outline: 3px solid #194b9b;
  outline-offset: 3px;
}

:root[data-theme="dark"] :focus-visible {
  outline-color: #a9c7ff;
}
~~~

Check each focus color against the surrounding surface in its theme. Do not remove keyboard focus styling without providing a clear replacement.

## 14. Let forced colors users keep their palette

~~~css
@media (forced-colors: active) {
  .status-indicator {
    border: 1px solid CanvasText;
    background: Canvas;
  }

  :focus-visible {
    outline: 2px solid Highlight;
    outline-offset: 3px;
  }
}
~~~

Ordinary text and controls should usually be left to the browser's forced color adjustments. System colors can be useful when a custom border or focus indicator needs to remain visible.

## 15. Pair status colors with another signal

~~~html
<p class="form-message form-message--error">
  <span aria-hidden="true">!</span>
  <strong>Error:</strong> Enter a valid email address.
</p>
~~~

~~~css
:root {
  --color-error: #b3261e;
}

.form-message--error {
  color: var(--color-error);
}

.form-message--error > span {
  display: inline-grid;
  place-items: center;
  width: 1.25em;
  height: 1.25em;
  margin-inline-end: 0.25em;
  border: 1px solid currentColor;
  border-radius: 50%;
}
~~~

The text and icon shape reinforce the error meaning. Color supports the message but is not its only signal.

## 16. Keep a semantic alias and its palette source at the same theme boundary

A nested theme should set the semantic token it consumes:

~~~css
:root {
  --palette-blue-700: #194b9b;
  --color-link: var(--palette-blue-700);
}

.preview {
  --palette-blue-700: #a9c7ff;
  --color-link: var(--palette-blue-700);
}

.preview a {
  color: var(--color-link);
}
~~~

If only the nested palette token were changed, an inherited semantic custom property might already contain the value resolved on an ancestor. Reassigning the semantic token in the nested scope makes the relationship explicit.

## 17. Use Sass for generated structure and CSS for runtime selection

~~~scss
$spacing: (
  "small": 0.5rem,
  "medium": 1rem,
  "large": 1.5rem
);

:root {
  @each $name, $value in $spacing {
    --space-#{$name}: #{$value};
  }
}

.card {
  padding: var(--space-medium);
}
~~~

Sass generates the repeated spacing tokens. The browser can still override a CSS custom property later, for example on a theme or component container.

## 18. Keep state tokens separate from theme mode

~~~css
:root {
  --color-success: #176b45;
  --color-warning: #8a5200;
  --color-error: #b3261e;
}

:root[data-theme="dark"] {
  --color-success: #73d5a3;
  --color-warning: #f3c66d;
  --color-error: #ff9b91;
}
~~~

A status role keeps the same meaning in every theme. These sample values are illustrative; choose and test values for the real backgrounds and text that use them.

## 19. Review a token matrix before release

Use a simple matrix when reviewing a theme change:

| Role or state | Light | Dark | System mode | Forced colors |
| --- | --- | --- | --- | --- |
| Main text on page | Check pair | Check pair | Follows preference | Browser palette |
| Link on surface | Check pair | Check pair | Follows preference | Visible link cue |
| Focus indicator | Check every surface | Check every surface | Follows preference | System highlight |
| Error message | Text and icon | Text and icon | Follows preference | Text remains clear |
| Native form control | Light scheme | Dark scheme | Browser preference | User palette |

A check in one column does not prove the other combinations work. Inspect the actual combinations and interactive states used by the interface.

## 20. Practice tasks

1. Add a Sass map for a small palette and generate CSS custom properties with @each.
2. Create semantic roles for page, surface, text, link, border, and action.
3. Define light and dark values for every semantic role.
4. Add a system mode using prefers-color-scheme.
5. Add an explicit mode selector with system, light, and dark choices.
6. Apply a theme to a nested preview without changing the document root.
7. Add one component token for a real variant and explain why the alias helps.
8. Test normal, hover, active, disabled, and keyboard focus states in each theme.
9. Check text contrast against each background where it appears.
10. Inspect the page in forced colors mode and confirm the browser palette is not disabled.

## Quick reference

- Sass maps and loops generate repeated CSS at build time.
- Semantic custom properties express interface roles.
- Set theme tokens at the root or a wrapper and let descendants inherit them.
- Keep JavaScript responsible for choosing a mode, not storing a second color palette.
- Use color-scheme for browser-provided UI and theme-aware CSS for authored content.
- Validate text contrast, focus visibility, status cues, and forced-colors behavior in every theme.

## Related material

- [Lesson: Runtime theming and design tokens](../lessons/038-runtime-theming-and-design-tokens.md)
- [Q&A: Runtime theming and design tokens](../qnas/038-runtime-theming-and-design-tokens.md)
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="037-sass-variables-vs-css-custom-properties.md">037 - Sass Variables versus CSS Custom Properties: Examples</a></td><td><a href="../lessons/038-runtime-theming-and-design-tokens.md">Lesson</a></td><td><a href="../qnas/038-runtime-theming-and-design-tokens.md">Q&amp;A</a></td><td><a href="039-reusable-responsive-mixins.md">039 - Reusable Responsive Mixins: Examples</a></td></tr>
  </tbody>
</table>
