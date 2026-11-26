# 017 — Interpolation in Selectors and Property Names

## Learning goals

After this lesson, you should be able to:

- Explain Sass interpolation with the syntax #{}.
- Insert Sass values into selectors and property names.
- Build selector names from a component or state value.
- Generate a property name from a Sass variable when a reusable pattern needs it.
- Recognize that interpolation produces unquoted text and may remove string quotes.
- Avoid interpolation where Sass can already evaluate an expression.
- Keep dynamically generated selectors and property names predictable.

## 1. What interpolation does

Interpolation inserts the result of a SassScript expression into a larger piece of CSS syntax. Write the expression inside #{ and }.

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

The value of $name becomes part of the selector before Sass emits CSS.

Interpolation is useful when the value is being used to construct a selector, property name, or another CSS token. It is not required every time a Sass variable is used.

## 2. Interpolate a variable into a selector

A selector can be built from a dynamic class suffix or state name.

~~~scss
$component: "button";
$state: "loading";

.#{$component}--#{$state} {
  cursor: progress;
  opacity: 0.7;
}
~~~

Generated CSS:

~~~css
.button--loading {
  cursor: progress;
  opacity: 0.7;
}
~~~

Interpolation places the expression's result directly in the selector. It does not add a descendant relationship by itself.

A more reusable pattern can be written as a mixin:

~~~scss
@mixin state-class($component, $state) {
  .#{$component}--#{$state} {
    @content;
  }
}

@include state-class("button", "loading") {
  cursor: progress;
  opacity: 0.7;
}
~~~

Generated CSS:

~~~css
.button--loading {
  cursor: progress;
  opacity: 0.7;
}
~~~

This is most useful when the same naming pattern is repeated. For a single static selector, writing the selector directly is usually clearer.

## 3. Generate several related selectors

Interpolation can build each selector from a value. In this example, the selectors are written out to make the result explicit:

~~~scss
$size: "compact";

.card-#{$size} {
  padding: 0.75rem;
}

.card-regular {
  padding: 1rem;
}
~~~

Generated CSS:

~~~css
.card-compact {
  padding: 0.75rem;
}
.card-regular {
  padding: 1rem;
}
~~~

When selectors are generated from a list or map, Sass control-flow rules can repeat the same pattern. Loops and maps are covered later in the course.

Use names that describe the intended component or variant. Dynamic selector generation can make it harder to find a rule by searching the source, so keep the naming rule simple and documented.

## 4. Interpolate into a property name

Interpolation can construct all or part of a declaration's property name.

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

A mixin can accept a property name when that makes a real pattern reusable:

~~~scss
@mixin set-spacing($property, $value) {
  #{$property}: $value;
}

.card {
  @include set-spacing(padding-inline, 1rem);
}
~~~

Generated CSS:

~~~css
.card {
  padding-inline: 1rem;
}
~~~

The property name is dynamic; the value remains a normal Sass expression. Keep dynamic property names limited to valid, expected CSS property names.

## 5. Use interpolation only where syntax is being constructed

For an ordinary property value, use a variable directly.

~~~scss
$accent: #175cd3;

.link {
  color: $accent;
}
~~~

Generated CSS:

~~~css
.link {
  color: #175cd3;
}
~~~

This is unnecessarily complicated:

~~~scss
.link {
  color: #{$accent};
}
~~~

Sass already evaluates the variable in a property value. Interpolating it converts the value to unquoted text before emitting it, which can remove Sass's type-aware handling.

Use interpolation when the value becomes part of a selector or property name. Use normal SassScript when supplying a declaration value.

## 6. Avoid turning numbers into strings

Interpolation with numbers can discard their numeric type and units. Do not use it to attach a unit to a number.

Avoid:

~~~scss
$gap: 12;

.grid {
  gap: #{$gap}px;
}
~~~

This builds text that looks like a length, rather than performing unit-aware Sass arithmetic.

Prefer to store the unit with the value:

~~~scss
$gap: 12px;

.grid {
  gap: $gap;
}
~~~

Or perform unit-aware arithmetic when the number is intentionally unitless:

~~~scss
$gap: 12;

.grid {
  gap: $gap * 1px;
}
~~~

Sass can check and calculate units when the value remains a number. Interpolation should not replace Sass's unit arithmetic.

## 7. Quoted strings lose their quotes in interpolation

When a quoted string is interpolated into another value, its surrounding quotation marks are removed.

~~~scss
$family: "Arial, sans-serif";

.heading {
  font-family: #{$family};
}
~~~

The interpolation result is unquoted text. For a CSS property value where quotes are meaningful, do not assume the original string quotes are preserved.

For ordinary property values, use the variable directly:

~~~scss
$family: "Arial, sans-serif";

.heading {
  font-family: $family;
}
~~~

Generated CSS:

~~~css
.heading {
  font-family: "Arial, sans-serif";
}
~~~

If a quoted value must be inserted through interpolation, Sass's meta.inspect() can serialize the Sass value with its quotes. This is an advanced workaround, not the default way to write declarations.

~~~scss
@use "sass:meta";

$family: "Arial, sans-serif";

:root {
  --font-family: #{meta.inspect($family)};
}
~~~

Generated CSS:

~~~css
:root {
  --font-family: "Arial, sans-serif";
}
~~~

## 8. CSS custom property values are a special case

CSS custom property values are passed through as CSS text so that the browser and JavaScript can use them. Sass does not evaluate a Sass variable written there like an ordinary declaration value. Interpolate the Sass value explicitly.

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

Writing --accent: $accent would leave the dollar-prefixed text in the emitted custom property value. For quoted Sass strings, use meta.inspect() when the quote characters must be retained.

Custom properties are covered in more detail in the CSS integration section of the roadmap.

## 9. Constructing a complete property name

Interpolation can form a complete property name when a shared mixin needs to choose one.

~~~scss
@mixin logical-spacing($property, $amount) {
  #{$property}: $amount;
}

.notice {
  @include logical-spacing(padding-block, 1rem);
  @include logical-spacing(padding-inline, 1.5rem);
}
~~~

Generated CSS:

~~~css
.notice {
  padding-block: 1rem;
  padding-inline: 1.5rem;
}
~~~

A mixin should receive known property names from its author or configuration. Avoid constructing property names from arbitrary user input; a typo can produce invalid or ineffective CSS without expressing the intended behavior.

## 10. Interpolation is resolved before selector parsing

Sass substitutes interpolated text and then parses the resulting selector. The generated selector must still be valid CSS.

~~~scss
$element: "input";

.form {
  #{$element}:focus {
    outline: 3px solid #84caff;
  }
}
~~~

Generated CSS:

~~~css
.form input:focus {
  outline: 3px solid #84caff;
}
~~~

If interpolation creates an invalid selector or property name, Sass reports an error or emits CSS that does not behave as intended. Treat selector construction as code, not as unvalidated string formatting.

## 11. Interpolation in selector lists

A selector can include more than one interpolated fragment:

~~~scss
$block: "notice";
$element: "icon";

.#{$block}__#{$element} {
  inline-size: 1rem;
}
~~~

Generated CSS:

~~~css
.notice__icon {
  inline-size: 1rem;
}
~~~

For a selector list, you can write each selector explicitly or use a list-driven pattern in a later lesson about Sass lists and control flow. Prefer an explicit selector list if only a few static selectors are needed.

## 12. Common mistakes

### Mistake: interpolating a normal property value

~~~scss
$color: #175cd3;

.link {
  color: #{$color};
}
~~~

Use color: $color instead. Sass already evaluates the variable in an ordinary property value.

### Mistake: adding units by string interpolation

~~~scss
$width: 8;

.box {
  width: #{$width}px;
}
~~~

Store a length such as 8px or use Sass arithmetic. This keeps the value numeric and unit-aware.

### Mistake: forgetting that quotes are removed

Interpolating a quoted string can remove the quotes that CSS requires. Use the variable normally for ordinary properties, or use meta.inspect() when a custom property needs the quoted serialization.

### Mistake: generating names that are difficult to trace

A dynamically generated selector or property name can be harder to find in the source. Keep the naming rule simple, and inspect the generated CSS when names are assembled from several fragments.

### Mistake: building Sass variable names dynamically

Sass does not support interpolating a variable name to look up another variable. Use a map when a name needs to select a value.

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

This keeps the relationship between names and values explicit.

## 13. Practical example: generated icon classes

Source SCSS:

~~~scss
$icon-name: "download";
$icon-size: 1.25rem;

.icon-#{$icon-name} {
  display: inline-block;
  inline-size: $icon-size;
  block-size: $icon-size;
  background-image: url("/icons/#{$icon-name}.svg");
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

Interpolation constructs the selector and URL text. The dimension uses a normal variable because it is already a Sass value with a unit.

## 14. Exercises

1. Use a string variable to create a class selector named .badge--success.
2. Use interpolation to create a property name from a logical side such as inline-start.
3. Rewrite a declaration that interpolates a color value so it uses normal SassScript instead.
4. Given a unitless number, explain why appending px with interpolation is unsafe and rewrite it using a Sass length.
5. Put a Sass variable into a CSS custom property value using interpolation.
6. Use meta.inspect() to preserve the quotes around a font stack in a custom property.
7. Build an icon class and image path from one Sass string variable.
8. Review a generated selector and property name. Confirm that each is valid CSS and easy to locate in the source.

## Summary

- Interpolation uses #{expression} to insert SassScript output into CSS syntax.
- It is useful in selectors and property names when those names must be constructed dynamically.
- Ordinary property values already evaluate Sass variables and expressions; interpolation is usually unnecessary there.
- Interpolation returns unquoted text, so quoted string values may lose their quotes.
- Avoid using interpolation to append units to numbers; use Sass numbers with units and unit-aware arithmetic.
- CSS custom property values are passed through as CSS text, so interpolate Sass values that need to be inserted.
- Keep dynamically generated names simple and inspect the resulting CSS.

## Official references

- [Sass interpolation](https://sass-lang.com/documentation/interpolation/)
- [Sass property declarations](https://sass-lang.com/documentation/style-rules/declarations/)
- [Sass values: numbers and units](https://sass-lang.com/documentation/values/numbers/)
- [Sass values: strings](https://sass-lang.com/documentation/values/strings/)
- [Sass variables and dynamic variable names](https://sass-lang.com/documentation/variables/)
- [Sass custom property syntax](https://sass-lang.com/documentation/breaking-changes/css-vars/)
