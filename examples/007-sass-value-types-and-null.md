# 007 — Sass Value Types and Null: Examples

These examples use SCSS syntax. Save each snippet in a .scss file and compile it with Dart Sass to compare the result.

## 1. Store and calculate values

~~~scss
$base: 1rem;
$double: $base * 2;

.card {
  padding: $base;
  margin-bottom: $double;
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1rem;
  margin-bottom: 2rem;
}
~~~

The variables hold number values. Sass evaluates the multiplication during compilation.

## 2. Quoted and unquoted strings

~~~scss
$heading-font: "Inter";
$fallback-font: sans-serif;

body {
  font-family: $heading-font, "Segoe UI", $fallback-font;
}
~~~

Generated CSS:

~~~css
body {
  font-family: "Inter", "Segoe UI", sans-serif;
}
~~~

The quoted strings keep their quotes. The unquoted CSS identifier remains unquoted.

## 3. Color values

~~~scss
$brand: #2457a7;
$surface: rgb(245 247 250);

.button {
  color: white;
  background-color: $brand;
  border-color: $surface;
}
~~~

Generated CSS:

~~~css
.button {
  color: white;
  background-color: #2457a7;
  border-color: rgb(245 247 250);
}
~~~

Colors can be written as keywords, hex values, or CSS color functions.

## 4. Space-separated and comma-separated lists

~~~scss
@use "sass:list";

$space-scale: 0.25rem 0.5rem 1rem 1.5rem;
$font-stack: "Inter", "Segoe UI", sans-serif;

.card {
  padding: list.nth($space-scale, 3);
  font-family: $font-stack;
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1rem;
  font-family: "Inter", "Segoe UI", sans-serif;
}
~~~

This demonstrates a space-separated list and a comma-separated list. List separators and list functions are covered in the list chapter.

## 5. Look up values from a map

~~~scss
@use "sass:map";

$colors: (
  "brand": #2457a7,
  "text": #20242b,
  "surface": #f5f7fa
);

.page {
  color: map.get($colors, "text");
  background-color: map.get($colors, "surface");
}

.link {
  color: map.get($colors, "brand");
}
~~~

Generated CSS:

~~~css
.page {
  color: #20242b;
  background-color: #f5f7fa;
}

.link {
  color: #2457a7;
}
~~~

The map itself does not become CSS. Sass reads values from it and uses those results in declarations.

## 6. A boolean controls an optional style

~~~scss
$use-rounded-corners: true;

.button {
  padding: 0.75rem 1rem;

  @if $use-rounded-corners {
    border-radius: 0.5rem;
  }
}
~~~

Generated CSS:

~~~css
.button {
  padding: 0.75rem 1rem;
  border-radius: 0.5rem;
}
~~~

If the variable is changed to false, Sass skips the conditional block:

~~~scss
$use-rounded-corners: false;

.button {
  padding: 0.75rem 1rem;

  @if $use-rounded-corners {
    border-radius: 0.5rem;
  }
}
~~~

Generated CSS:

~~~css
.button {
  padding: 0.75rem 1rem;
}
~~~

A boolean is a clear choice for an on/off configuration.

## 7. Sass truthiness differs from JavaScript

~~~scss
@if 0 {
  .zero-condition {
    display: block;
  }
}

@if "" {
  .empty-string-condition {
    display: block;
  }
}

$empty-list: ();

@if $empty-list {
  .empty-list-condition {
    display: block;
  }
}
~~~

All three conditions are truthy in Sass. The generated CSS includes all three rules:

~~~css
.zero-condition {
  display: block;
}

.empty-string-condition {
  display: block;
}

.empty-list-condition {
  display: block;
}
~~~

Only false and null are falsey in Sass. Use explicit booleans when you need an on/off switch.

## 8. A null property value disappears

~~~scss
$optional-shadow: null;

.card {
  color: #20242b;
  box-shadow: $optional-shadow;
}
~~~

Generated CSS:

~~~css
.card {
  color: #20242b;
}
~~~

Sass removes the whole box-shadow declaration. It does not emit box-shadow: null.

## 9. A null list item disappears

~~~scss
$font-stack: "Inter", null, sans-serif;

body {
  font-family: $font-stack;
}
~~~

Generated CSS:

~~~css
body {
  font-family: "Inter", sans-serif;
}
~~~

The null item is removed while the remaining comma-separated values form the font list.

## 10. Use null as an optional mixin argument

~~~scss
@mixin card-surface($shadow: null) {
  border: 1px solid #d8dde5;
  box-shadow: $shadow;
}

.card {
  @include card-surface;
}

.card--raised {
  @include card-surface(0 0.5rem 1.25rem rgb(20 30 50 / 12%));
}
~~~

Generated CSS:

~~~css
.card {
  border: 1px solid #d8dde5;
}

.card--raised {
  border: 1px solid #d8dde5;
  box-shadow: 0 0.5rem 1.25rem rgb(20 30 50 / 12%);
}
~~~

The default null makes the shadow optional. The first include emits no shadow declaration; the second passes a real value.

## 11. Use null to skip or enable a conditional style

~~~scss
$focus-color: null;

.field {
  border: 1px solid #87909e;

  @if $focus-color {
    &:focus {
      outline: 2px solid $focus-color;
    }
  }
}
~~~

With null, the generated CSS is:

~~~css
.field {
  border: 1px solid #87909e;
}
~~~

Now set the variable to a color:

~~~scss
$focus-color: #2457a7;

.field {
  border: 1px solid #87909e;

  @if $focus-color {
    &:focus {
      outline: 2px solid $focus-color;
    }
  }
}
~~~

Generated CSS:

~~~css
.field {
  border: 1px solid #87909e;
}

.field:focus {
  outline: 2px solid #2457a7;
}
~~~

Null is falsey, so it skips the conditional. A color is truthy, so Sass includes the rule.

## 12. A missing map key can return null

~~~scss
@use "sass:map";

$font-sizes: (
  "small": 0.875rem,
  "large": 1.5rem
);

@debug map.get($font-sizes, "small");  // 0.875rem
@debug map.get($font-sizes, "medium"); // null
~~~

Representative compiler diagnostics:

~~~text
example.scss:10 DEBUG: 0.875rem
example.scss:11 DEBUG: null
~~~

A missing lookup result can be null. If assigned to a CSS property, that declaration is omitted.

~~~scss
$medium-size: map.get($font-sizes, "medium");

.caption {
  font-size: $medium-size;
}
~~~

No .caption rule is emitted because its only declaration has a null value.

## 13. Compare false with null

~~~scss
$feature-enabled: false;
$optional-color: null;

@if $feature-enabled {
  .feature {
    display: block;
  }
}

@if $optional-color {
  .optional {
    color: $optional-color;
  }
}
~~~

Neither rule is emitted. Both values are falsey, but they communicate different things:

- false means the feature is explicitly switched off.
- null means no color value is present.

## 14. Inspect values with @debug

~~~scss
$gap: 1rem 1.5rem;
$rounded: true;
$shadow: null;

@debug $gap;
@debug $rounded;
@debug $shadow;

.panel {
  padding: $gap;
  box-shadow: $shadow;
}
~~~

Representative compiler diagnostics:

~~~text
example.scss:5 DEBUG: 1rem 1.5rem
example.scss:6 DEBUG: true
example.scss:7 DEBUG: null
~~~

Generated CSS:

~~~css
.panel {
  padding: 1rem 1.5rem;
}
~~~

Diagnostics help inspect Sass values; they do not become CSS. The null shadow is omitted.

## 15. Full example: configurable alert

~~~scss
$alert-background: #f5f7fa;
$alert-text: #20242b;
$alert-border: null;
$alert-icon-enabled: true;
$alert-fonts: "Inter", null, sans-serif;

.alert {
  color: $alert-text;
  background-color: $alert-background;
  border: $alert-border;
  font-family: $alert-fonts;

  @if $alert-icon-enabled {
    &::before {
      content: "!";
      margin-right: 0.5rem;
    }
  }
}
~~~

Generated CSS:

~~~css
.alert {
  color: #20242b;
  background-color: #f5f7fa;
  font-family: "Inter", sans-serif;
}

.alert::before {
  content: "!";
  margin-right: 0.5rem;
}
~~~

The regular values are emitted, the null border is omitted, the null font-list item disappears, and the true boolean includes the pseudo-element.

## Try it yourself

1. Change $use-rounded-corners to false. Which declaration disappears?
2. Set $optional-shadow to 0 2px 8px rgb(0 0 0 / 15%). What changes in the CSS?
3. Replace null in the font stack with "Arial". What does Sass emit?
4. Change $focus-color from null to teal. Which selector is added?
5. Set $alert-icon-enabled to false. What happens to the pseudo-element?
6. Use @debug on a missing map lookup. What value does Sass report?

## References

- [Sass values](https://sass-lang.com/documentation/values/)
- [Sass null](https://sass-lang.com/documentation/values/null/)
- [Sass booleans](https://sass-lang.com/documentation/values/booleans/)
- [Sass lists](https://sass-lang.com/documentation/values/lists/)
- [Sass maps](https://sass-lang.com/documentation/values/maps/)
