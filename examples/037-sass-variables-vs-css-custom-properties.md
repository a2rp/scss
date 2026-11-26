# 037 — Sass Variables versus CSS Custom Properties: Examples

These examples show which work happens in Sass during compilation and which values remain for the browser to resolve.

## 1. A Sass variable compiles to a literal CSS value

### Sass source

~~~scss
$brand: #1769aa;

.button {
  color: $brand;
}
~~~

### Compiled CSS

~~~css
.button {
  color: #1769aa;
}
~~~

The Sass variable is gone from the generated stylesheet.

## 2. Reassign a Sass variable for later declarations

~~~scss
$surface: white;

.light-card {
  background: $surface;
}

$surface: #20242a;

.dark-card {
  background: $surface;
}
~~~

Compiled CSS:

~~~css
.light-card {
  background: white;
}

.dark-card {
  background: #20242a;
}
~~~

Sass evaluates each use as it compiles the file. Reassigning a variable later does not change CSS that Sass already emitted.

## 3. Declare a CSS custom property and read it with var()

~~~css
:root {
  --brand: #1769aa;
}

.button {
  color: var(--brand);
}
~~~

The generated CSS keeps both the custom property and its reference. The browser resolves `var(--brand)` when it applies the button's styles.

## 4. Let a theme value inherit through a subtree

~~~css
:root {
  --surface: white;
  --text: #263238;
}

[data-theme="dark"] {
  --surface: #20242a;
  --text: #f5f7fa;
}

.card {
  color: var(--text);
  background: var(--surface);
}
~~~

A card inside an element marked `data-theme="dark"` receives the dark values. The declarations on `.card` do not need separate light and dark copies.

## 5. Override a custom property on one component

~~~css
.control {
  --control-height: 2.75rem;

  min-height: var(--control-height);
}

.control--compact {
  --control-height: 2rem;
}
~~~

The modifier changes the token on the matching element. The control's `min-height` declaration continues to read the same custom property.

## 6. Insert a Sass value into a custom property

~~~scss
$brand: #1769aa;

:root {
  --brand: #{$brand};
}

.button {
  color: var(--brand);
}
~~~

Output:

~~~css
:root {
  --brand: #1769aa;
}

.button {
  color: var(--brand);
}
~~~

Sass interpolation inserts the Sass value. The browser receives a real custom property.

## 7. See what happens without interpolation

~~~scss
$brand: #1769aa;

:root {
  --brand: $brand;
}
~~~

Sass passes the custom property value through as CSS tokens, so the result includes the literal Sass-looking token:

~~~css
:root {
  --brand: $brand;
}
~~~

That value does not refer to the Sass variable. Use interpolation when the value should be computed by Sass.

## 8. Calculate with Sass before emitting a custom property

~~~scss
$space-unit: 0.25rem;
$space-factor: 6;

:root {
  --space-large: #{$space-unit * $space-factor};
}

.panel {
  padding: var(--space-large);
}
~~~

Compiled CSS:

~~~css
:root {
  --space-large: 1.5rem;
}

.panel {
  padding: var(--space-large);
}
~~~

Sass performs the multiplication at build time. The browser still resolves the custom property at runtime.

## 9. Preserve quotes in an interpolated string

~~~scss
@use "sass:meta";

$font-stack: ("Field Sans", sans-serif);

:root {
  --font-stack: #{meta.inspect($font-stack)};
}

.heading {
  font-family: var(--font-stack);
}
~~~

The custom property is emitted with the font name quoted:

~~~css
:root {
  --font-stack: "Field Sans", sans-serif;
}

.heading {
  font-family: var(--font-stack);
}
~~~

Use `meta.inspect()` when interpolation would otherwise remove quotes that are part of the CSS value.

## 10. Generate CSS tokens from a Sass map

~~~scss
$colors: (
  "brand": #1769aa,
  "text": #263238,
  "surface": white
);

:root {
  @each $name, $value in $colors {
    --color-#{$name}: #{$value};
  }
}

.button {
  color: var(--color-text);
  background: var(--color-brand);
}
~~~

Generated CSS:

~~~css
:root {
  --color-brand: #1769aa;
  --color-text: #263238;
  --color-surface: white;
}

.button {
  color: var(--color-text);
  background: var(--color-brand);
}
~~~

The Sass map and loop create CSS custom property declarations. The browser uses the generated tokens in ordinary CSS.

## 11. Use a var() fallback for a missing token

~~~css
.button {
  color: var(--button-text, #263238);
  background: var(--button-surface, white);
}
~~~

If the custom property is missing or has its guaranteed-invalid value, the fallback can supply a value for that reference. The fallback does not declare the custom property.

## 12. A fallback does not validate the substituted value

~~~css
:root {
  --button-text: 12px;
}

.button {
  color: var(--button-text, #263238);
}
~~~

The custom property exists, so `var()` substitutes `12px`. Since `12px` is not a valid `color` value, the browser does not retry `#263238` as the fallback. The consuming declaration becomes invalid at computed-value time.

Use an appropriately typed token and consider `@property` when the browser should validate values.

## 13. Use nested var() fallbacks

~~~css
.button {
  color: var(--local-text, var(--theme-text, #263238));
}
~~~

The browser checks the local token first, then the theme token, then the literal fallback. Each fallback is used if the referenced custom property is missing or guaranteed-invalid.

## 14. Use calc() with a CSS custom property

~~~css
:root {
  --space-unit: 0.25rem;
}

.card {
  padding: calc(var(--space-unit) * 4);
  gap: calc(var(--space-unit) * 2);
}
~~~

The browser substitutes the custom property and resolves the calculation. If a theme or component overrides `--space-unit`, both calculations use the updated token.

## 15. Create default tokens with Sass and allow CSS overrides

~~~scss
$default-brand: #1769aa;
$default-radius: 0.5rem;

:root {
  --brand: #{$default-brand};
  --radius: #{$default-radius};
}

.button {
  background: var(--brand);
  border-radius: var(--radius);
}

.button--rounded {
  --radius: 999px;
}
~~~

Sass supplies initial values in the generated CSS. A later theme rule or component modifier can override a custom property through the cascade.

## 16. Configure Sass output at build time

A Sass library can expose a build-time default:

~~~scss
// _design-system.scss
$brand: #1769aa !default;

:root {
  --brand: #{$brand};
}

.button {
  color: var(--brand);
}
~~~

An entrypoint can configure that module before its CSS is emitted:

~~~scss
// campaign.scss
@use "design-system" with (
  $brand: #6b4eff
);
~~~

The generated stylesheet contains the configured value. Changing `$brand` after compilation requires another build; it is not a browser runtime switch.

## 17. Register a typed custom property

~~~css
@property --progress {
  syntax: "<percentage>";
  inherits: false;
  initial-value: 0%;
}

.progress-bar {
  width: var(--progress);
}
~~~

The registration says that `--progress` accepts percentage values, does not inherit, and starts at zero percent. Because the syntax is not universal `*`, the initial value is required and must match the syntax.

## 18. Animate a registered custom property

~~~css
@property --turn {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}

.spinner {
  --turn: 0deg;
  transform: rotate(var(--turn));
  transition: --turn 500ms ease;
}

.spinner.is-active {
  --turn: 1turn;
}
~~~

The browser can interpolate the registered angle value during the transition. Without registration, a custom property generally animates as an untyped token sequence rather than as an angle.

## 19. Change a custom property from JavaScript

CSS:

~~~css
:root {
  --accent: #1769aa;
}

.link {
  color: var(--accent);
}
~~~

JavaScript:

~~~js
document.documentElement.style.setProperty("--accent", "#6b4eff");
~~~

The browser updates the root custom property, and declarations that reference it use the new value. No Sass recompilation is involved.

## 20. Use Sass for a build-time branch

~~~scss
$high-contrast-build: false;

@if $high-contrast-build {
  .notice {
    border: 2px solid currentColor;
  }
}
~~~

With the flag set to false, Sass emits no `.notice` rule from this branch. This is a build-time decision.

## 21. Use CSS for a browser-time theme choice

~~~css
:root {
  --notice-border: transparent;
}

@media (forced-colors: active) {
  :root {
    --notice-border: currentColor;
  }
}

.notice {
  border: 2px solid var(--notice-border);
}
~~~

The browser evaluates the media query at runtime and applies the custom property value from the matching rule.

## 22. Compare custom properties and Sass variables in one file

~~~scss
$gutter: 1.5rem;

:root {
  --gutter: #{$gutter};
}

.grid {
  gap: $gutter;
  padding-inline: var(--gutter);
}
~~~

The `gap` value is fixed during Sass compilation. The `padding-inline` value uses a CSS custom property that can be overridden by another CSS rule.

## 23. Practice tasks

1. Compile a Sass variable used in an ordinary declaration and inspect the emitted literal value.
2. Emit the same value as a custom property and read it with `var()`.
3. Remove interpolation from a custom property assignment and note what CSS Sass emits.
4. Create a theme wrapper that overrides inherited custom properties.
5. Generate custom properties from a Sass map using `sass:map` and a loop.
6. Preserve quotes around a font list with `sass:meta` and `meta.inspect()`.
7. Add a `var()` fallback and explain when the browser uses it.
8. Register a typed custom property with `@property` and use it for animation.
9. Change a root custom property from JavaScript and observe its consumers.
10. Decide which tokens in a small component should be compile-time Sass values and which should be runtime CSS values.

## Quick reference

~~~scss
@use "sass:meta";

$brand: #1769aa;
$font-stack: ("Field Sans", sans-serif);

:root {
  --brand: #{$brand};
  --font-stack: #{meta.inspect($font-stack)};
}

.button {
  color: var(--button-text, #263238);
  background: var(--brand);
}
~~~

- Use `$variable` for Sass build-time values.
- Use `--property` and `var(--property)` for browser-level CSS values.
- Interpolate Sass expressions inside custom property declarations.
- Custom properties cascade and inherit; Sass variables do not.
- Use `@property` to register a custom property's syntax and inheritance behavior.

## Related material

- [Lesson: Sass variables versus CSS custom properties](../lessons/037-sass-variables-vs-css-custom-properties.md)
- [Q&A: Sass variables versus CSS custom properties](../qnas/037-sass-variables-vs-css-custom-properties.md)
- [Sass: Variables](https://sass-lang.com/documentation/variables/)
- [Sass: Custom property declarations](https://sass-lang.com/documentation/style-rules/declarations/#custom-properties)
- [Sass: Interpolation](https://sass-lang.com/documentation/interpolation/)
- [CSS Custom Properties for Cascading Variables](https://www.w3.org/TR/css-variables/)
- [CSS Properties and Values API: @property](https://www.w3.org/TR/css-properties-values-api-1/)
