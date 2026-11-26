# 037 — Sass Variables versus CSS Custom Properties

## Learning goals

After this lesson, you should be able to:

- Explain how Sass variables differ from CSS custom properties.
- Choose whether a value should be resolved by Sass at build time or by the browser at runtime.
- Use Sass values to generate custom property declarations safely.
- Reference custom properties with the CSS `var()` function.
- Use custom properties for inherited themes and per-component overrides.
- Understand Sass's special parsing rules for custom property values.
- Preserve quotes when interpolating Sass strings into custom properties.
- Register a custom property with `@property` when type checking, inheritance control, or an initial value is useful.
- Combine Sass design tokens and browser-level runtime tokens without duplicating the source of truth.

## 1. Two different kinds of variables

Sass variables and CSS custom properties both store values, but they operate at different stages.

| Sass variable | CSS custom property |
| --- | --- |
| Starts with `$` in Sass source | Starts with `--` as a CSS property |
| Evaluated by Sass during compilation | Emitted to CSS and evaluated by the browser |
| Usually does not appear in compiled CSS | Appears in compiled CSS |
| Has Sass scope and Sass value types | Participates in CSS cascade and inheritance |
| Useful for build-time choices and calculations | Useful for runtime theming and context-specific values |
| Referenced as `$brand` | Declared as `--brand` and used as `var(--brand)` |

They can work together, but one is not a renamed version of the other.

## 2. Sass variables are resolved during compilation

Sass evaluates Sass variables while compiling:

~~~scss
$brand: #1769aa;

.button {
  color: $brand;
}
~~~

Compiled CSS:

~~~css
.button {
  color: #1769aa;
}
~~~

The Sass variable name disappears. The browser only receives the color value.

Sass variables are imperative: when a variable is reassigned, earlier declarations keep the value they already used.

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

The later assignment affects later Sass evaluation, not earlier emitted CSS.

## 3. CSS custom properties remain in the output

A custom property is declared using a dashed name and read with `var()`:

~~~css
:root {
  --brand: #1769aa;
}

.button {
  color: var(--brand);
}
~~~

The browser receives both the token and the reference. It resolves the custom property through the CSS cascade for the element being styled.

Custom properties can be reassigned by CSS rules, inherited by descendants, and read or changed through browser APIs such as the CSS Object Model.

## 4. CSS custom properties participate in the cascade and inheritance

A custom property can have different values in different parts of the document:

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

A card inside an element with `data-theme="dark"` receives the dark values through inheritance. A card outside that subtree uses the root values.

This makes custom properties useful for themes and local overrides that the browser can resolve without recompiling Sass.

## 5. Use Sass variables for compile-time values

Use Sass variables when a value is needed while generating CSS:

~~~scss
$space-unit: 0.25rem;
$card-padding: $space-unit * 6;
$wide-layout: 68rem;

.card {
  padding: $card-padding;
}

@media (min-width: $wide-layout) {
  .page {
    max-width: 72rem;
  }
}
~~~

Sass performs the multiplication and writes concrete values into declarations and media queries.

Sass variables also work well for maps, loops, mixins, functions, module configuration, and compile-time conditional generation.

## 6. Use custom properties for browser-time values

Use CSS custom properties when the value should respond to the document, the cascade, or a runtime change:

~~~scss
:root {
  --control-height: 2.75rem;
  --control-radius: 0.5rem;
}

.button {
  min-height: var(--control-height);
  border-radius: var(--control-radius);
}

.button--compact {
  --control-height: 2rem;
}
~~~

The compact modifier changes the custom property on that element. The browser resolves the button's declarations using the values available in its CSS context.

## 7. Interpolate Sass values into custom property declarations

Sass treats custom property values specially. Most text in a custom property is passed through as CSS rather than evaluated as SassScript. Use interpolation when the value should come from a Sass variable:

~~~scss
$brand: #1769aa;

:root {
  --brand: #{$brand};
}
~~~

Compiled CSS:

~~~css
:root {
  --brand: #1769aa;
}
~~~

Without interpolation, Sass passes a token such as `$brand` through as text:

~~~scss
$brand: #1769aa;

:root {
  --brand: $brand;
}
~~~

The emitted custom property value is literally `$brand`, not the color. This is valid to Sass because custom property values may contain arbitrary token streams, but it is probably not what the author intended.

## 8. Interpolate calculated Sass values

A Sass expression can be evaluated before its result is placed in a custom property:

~~~scss
$unit: 0.25rem;
$spacing-scale: 5;

:root {
  --space-large: #{$unit * $spacing-scale};
}
~~~

Compiled CSS:

~~~css
:root {
  --space-large: 1.25rem;
}
~~~

The calculation happens in Sass. The resulting CSS custom property remains available to the browser and can be overridden at runtime.

## 9. Preserve quotes when interpolating Sass strings

Interpolation removes quotes from Sass strings. When the custom property needs a quoted string value, use `sass:meta` and `meta.inspect()`:

~~~scss
@use "sass:meta";

$font-stack: ("Field Sans", sans-serif);

:root {
  --font-stack: #{meta.inspect($font-stack)};
}
~~~

The output preserves the quotes around the font name:

~~~css
:root {
  --font-stack: "Field Sans", sans-serif;
}
~~~

For values that do not need quotes, ordinary interpolation is enough. Do not use `meta.inspect()` indiscriminately; it serializes Sass values in a readable form and is especially useful for quoted content passed into custom property values.

## 10. Generate custom properties from a Sass token map

A Sass map can be the build-time source of values used to generate browser-level tokens:

~~~scss
@use "sass:map";

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

The map and loop are Sass features. The generated custom properties and `var()` references are CSS features. If both Sass code and browser themes need the same values, document which representation is the source of truth.

## 11. Add a fallback to var()

The second argument to `var()` is used when the referenced custom property is missing or has its guaranteed-invalid value:

~~~css
.button {
  color: var(--button-text, #263238);
  background: var(--button-surface, white);
}
~~~

The fallback is CSS text. It can be a fixed value or another valid CSS expression. A fallback does not create or assign the custom property; it only supplies a value at the use site when the custom property is missing or guaranteed-invalid. It is not a type-check fallback: if a defined value substitutes successfully but makes the consuming property invalid, the browser does not retry the fallback.

~~~css
:root {
  --brand: #1769aa;
}

.button {
  color: var(--brand, navy);
}
~~~

The button uses `--brand` while it has a usable value. `navy` is the fallback if the custom property cannot be used there.

## 12. Use custom properties for theme switching

A theme can be changed by applying a class or attribute without rebuilding the stylesheet:

~~~scss
:root {
  color-scheme: light;
  --page-surface: #ffffff;
  --page-text: #263238;
  --page-accent: #1769aa;
}

[data-theme="dark"] {
  color-scheme: dark;
  --page-surface: #20242a;
  --page-text: #f5f7fa;
  --page-accent: #8ab4f8;
}

.page {
  color: var(--page-text);
  background: var(--page-surface);
}

.link {
  color: var(--page-accent);
}
~~~

The theme selector changes the custom property values. The component rules remain the same and inherit the values from their context.

## 13. Provide a component-level override

A component can define a default custom property and allow callers to override it:

~~~scss
.callout {
  --callout-accent: #1769aa;

  border-inline-start: 0.25rem solid var(--callout-accent);
  padding: 1rem;
}

.callout--warning {
  --callout-accent: #9a5b00;
}
~~~

The warning variant overrides the token on the element. The callout declaration continues to refer to the same custom property.

This pattern is useful when a component's styles should stay stable while a small number of values are customizable.

## 14. Use Sass variables and CSS custom properties together

Sass can define build-time defaults and emit them as custom properties:

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
~~~

Here Sass supplies the default token values during compilation. The resulting CSS custom properties remain overridable by later CSS rules, theme selectors, or inline styles that participate in the cascade.

Do not assume that changing the Sass variable in a running browser is possible. Sass has already finished compiling.

## 15. Register a custom property with @property

The `@property` at-rule can add a syntax, inheritance behavior, and initial value to a custom property:

~~~scss
@property --progress {
  syntax: "<percentage>";
  inherits: false;
  initial-value: 0%;
}

.progress-bar {
  width: var(--progress);
}
~~~

A registered property can be validated according to its syntax and may support typed animation behavior. Its registration is CSS output; the browser implements the behavior.

The `inherits` descriptor controls inheritance for this registered custom property. This differs from an ordinary unregistered custom property, which inherits by default.

## 16. Choose deliberately whether a token is static or runtime

A useful decision guide:

| Need | Good starting point |
| --- | --- |
| Calculate a value while generating CSS | Sass variable |
| Iterate over values to generate selectors or declarations | Sass map and loop |
| Change a theme by applying a class or attribute | CSS custom property |
| Override a value for a component subtree | CSS custom property |
| Configure a Sass library at build time | Sass module configuration |
| Let JavaScript update a style token | CSS custom property |
| Use one value in Sass arithmetic and expose it to the browser | Sass variable that emits a custom property |

Some systems need both. For example, Sass can create the initial CSS tokens from a map, while the browser uses those tokens for runtime theme changes.

## 17. Common mistakes

### Expecting a Sass variable to remain in CSS

Sass variables are compiled away. If the browser must see or override a token, emit a CSS custom property.

### Expecting a CSS custom property to be calculated by Sass

A custom property is CSS. Sass does not evaluate browser-time values or replace arbitrary token streams inside it. Use interpolation for a Sass value that should be inserted.

### Forgetting interpolation in a custom property value

`--gap: $gap;` passes the Sass-looking text through. Use `--gap: #{$gap};` when the value should be evaluated by Sass.

### Assuming custom properties are global constants

They participate in the cascade and inheritance. A later or more specific declaration can change which value applies to an element.

### Trying to read CSS custom properties as Sass variables

Sass cannot use a browser's runtime computed custom property value in a Sass map lookup or Sass calculation. Those values are resolved after the CSS reaches the browser.

### Interpolating a quoted string and losing its quotes

Interpolation removes quotes from Sass strings. Use `meta.inspect()` when a custom property needs the quoted representation.

### Using custom properties where Sass must decide the output

A Sass variable is needed for Sass functions, loops, selectors, and compile-time conditions. A CSS custom property cannot control Sass compilation.

### Using a Sass variable for a theme that must switch without rebuilding

If the browser must switch a theme at runtime, emit custom properties and change their CSS context.

## 18. Summary

Sass variables are build-time values that Sass evaluates and compiles away. CSS custom properties are emitted CSS values that participate in the browser's cascade and inheritance and can change at runtime. Use Sass for calculations, maps, mixins, and build-time configuration. Use custom properties for themes, local overrides, and values that CSS or JavaScript should update. Interpolate Sass values into custom property declarations, and use `meta.inspect()` when quotes must be preserved.

## Further reading

- [Sass: Variables](https://sass-lang.com/documentation/variables/)
- [Sass: Custom property declarations](https://sass-lang.com/documentation/style-rules/declarations/#custom-properties)
- [Sass: Interpolation](https://sass-lang.com/documentation/interpolation/)
- [CSS Custom Properties for Cascading Variables](https://www.w3.org/TR/css-variables/)
- [Chapter 036: Sass with CSS at-rules](036-sass-and-css-at-rules.md)
