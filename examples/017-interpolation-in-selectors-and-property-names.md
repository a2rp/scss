# 017 — Interpolation in Selectors and Property Names: Examples

These examples demonstrate when interpolation constructs CSS syntax and when a Sass value should be used directly.

## 1. Construct a selector from a variable

Source SCSS:

~~~scss
$name: "alert";

.#{$name} {
  color: #b42318;
}
~~~

Generated CSS:

~~~css
.alert {
  color: #b42318;
}
~~~

The variable is part of the selector, so interpolation inserts its text before Sass parses the selector.

## 2. Build a component variant class

Source SCSS:

~~~scss
$component: "button";
$variant: "primary";

.#{$component}--#{$variant} {
  color: white;
  background-color: #175cd3;
}
~~~

Generated CSS:

~~~css
.button--primary {
  color: white;
  background-color: #175cd3;
}
~~~

Interpolation creates the class name. It does not create a descendant relationship.

## 3. Reuse selector construction in a mixin

Source SCSS:

~~~scss
@mixin button-variant($name, $background) {
  .button--#{$name} {
    background-color: $background;
  }
}

@include button-variant("primary", #175cd3);
@include button-variant("danger", #b42318);
~~~

Generated CSS:

~~~css
.button--primary {
  background-color: #175cd3;
}
.button--danger {
  background-color: #b42318;
}
~~~

The selector name is dynamic; the background value is passed as a normal Sass value.

## 4. Generate part of a property name

Source SCSS:

~~~scss
$side: "inline-start";

.panel {
  margin-#{$side}: 1rem;
}
~~~

Generated CSS:

~~~css
.panel {
  margin-inline-start: 1rem;
}
~~~

Only the property name needs interpolation. The 1rem value is ordinary SassScript.

## 5. Interpolate a complete property name

Source SCSS:

~~~scss
@mixin set-spacing($property, $amount) {
  #{$property}: $amount;
}

.card {
  @include set-spacing(padding-block, 1rem);
  @include set-spacing(padding-inline, 1.5rem);
}
~~~

Generated CSS:

~~~css
.card {
  padding-block: 1rem;
  padding-inline: 1.5rem;
}
~~~

This pattern is useful when a reusable helper intentionally accepts different property names. Keep the supplied names controlled and valid.

## 6. Do not interpolate ordinary values

Unnecessary interpolation:

~~~scss
$accent: #175cd3;

.link {
  color: #{$accent};
}
~~~

Prefer:

~~~scss
$accent: #175cd3;

.link {
  color: $accent;
}
~~~

Both look similar as output, but the second keeps the Sass value in its normal typed form while Sass evaluates the declaration.

Generated CSS:

~~~css
.link {
  color: #175cd3;
}
~~~

## 7. Keep units numeric

Avoid building a length by concatenating text:

~~~scss
$gap: 12;

.grid {
  gap: #{$gap}px;
}
~~~

Prefer a value that already has its unit:

~~~scss
$gap: 12px;

.grid {
  gap: $gap;
}
~~~

Generated CSS:

~~~css
.grid {
  gap: 12px;
}
~~~

If a value is intentionally unitless, unit-aware Sass arithmetic is another option:

~~~scss
$gap: 12;

.grid {
  gap: $gap * 1px;
}
~~~

The arithmetic produces a Sass number with a length unit rather than unquoted text.

## 8. Insert a Sass value into a custom property

Source SCSS:

~~~scss
$accent: #175cd3;

:root {
  --accent: #{$accent};
}
~~~

Generated CSS:

~~~css
:root {
  --accent: #175cd3;
}
~~~

Custom property values are passed through as CSS text. Interpolation is how this example inserts the Sass variable value.

## 9. Preserve quotes for a custom property value

Source SCSS:

~~~scss
@use "sass:meta";

$font-stack: "Arial, sans-serif";

:root {
  --font-stack: #{meta.inspect($font-stack)};
}
~~~

Generated CSS:

~~~css
:root {
  --font-stack: "Arial, sans-serif";
}
~~~

Interpolating the quoted value directly removes its quotation marks. meta.inspect() serializes the value with quotes for this custom-property use case.

## 10. Build an icon selector and URL from one name

Source SCSS:

~~~scss
$icon: "download";

.icon-#{$icon} {
  display: inline-block;
  inline-size: 1.25rem;
  block-size: 1.25rem;
  background-image: url("/icons/#{$icon}.svg");
  background-repeat: no-repeat;
  background-size: contain;
}
~~~

Generated CSS:

~~~css
.icon-download {
  display: inline-block;
  inline-size: 1.25rem;
  block-size: 1.25rem;
  background-image: url("/icons/download.svg");
  background-repeat: no-repeat;
  background-size: contain;
}
~~~

The name is interpolated into the class and URL. The dimension remains a normal Sass value with units.

## 11. Use a map instead of a dynamic variable name

Do not try to construct a Sass variable name such as $color-#{$kind}. Store choices in a map instead.

Source SCSS:

~~~scss
@use "sass:map";

$colors: (
  "success": #067647,
  "warning": #b54708
);

$kind: "warning";

.message {
  color: map.get($colors, $kind);
}
~~~

Generated CSS:

~~~css
.message {
  color: #b54708;
}
~~~

The map keeps the available names and values explicit. Interpolation is not needed to look up the value.

## 12. A generated selector with a property helper

Source SCSS:

~~~scss
@mixin utility($name, $property, $value) {
  .u-#{$name} {
    #{$property}: $value;
  }
}

@include utility("text-center", text-align, center);
@include utility("hidden", display, none);
~~~

Generated CSS:

~~~css
.u-text-center {
  text-align: center;
}
.u-hidden {
  display: none;
}
~~~

This illustrates how selector and property-name interpolation can work together. A real utility generator should accept only known names and values so it cannot produce accidental or invalid CSS.

## 13. Compile and inspect

Save the examples in src/interpolation.scss and compile them with Dart Sass:

~~~sh
sass src/interpolation.scss dist/interpolation.css
~~~

Inspect the output and check:

- The selector has the intended class or state name.
- The generated property name is a valid CSS property.
- Normal values retain their numeric units.
- Interpolation did not remove quotes that CSS needs.
- CSS custom properties contain the evaluated Sass value where intended.

## 14. Practice tasks

1. Create a .badge--success selector from a variable.
2. Use a mixin parameter to generate .button--primary and .button--secondary.
3. Build a logical margin property using a variable for its suffix.
4. Rewrite a color declaration to remove unnecessary interpolation.
5. Replace a number-plus-unit interpolation with a unit-bearing Sass value.
6. Insert an accent variable into a CSS custom property.
7. Preserve quotes for a custom font-stack property with meta.inspect().
8. Create an icon selector and URL from the same name.
9. Replace a proposed dynamic Sass variable name with a map lookup.
10. Compile each result and read the generated CSS as ordinary browser CSS.

## Key takeaways

- Interpolate values into selector or property-name syntax when that syntax must be generated.
- Pass Sass values directly in ordinary property values.
- Keep numbers numeric and unit-aware.
- Remember that interpolated quoted strings lose their quotes.
- Interpolate Sass values inside CSS custom properties; use meta.inspect() when quotes matter.
- Prefer maps over attempts to construct variable names dynamically.
- Inspect generated selectors and property names for correctness and clarity.
