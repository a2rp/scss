# 036 — Sass with CSS At-Rules: Examples

This chapter's examples focus on how Sass source becomes CSS when CSS at-rules are nested, parameterized, or emitted from mixins.

## 1. Bubble a media query out of a selector

### Sass source

~~~scss
.card {
  padding: 1rem;

  @media (min-width: 48rem) {
    padding: 1.5rem;
  }
}
~~~

### Compiled CSS

~~~css
.card {
  padding: 1rem;
}

@media (min-width: 48rem) {
  .card {
    padding: 1.5rem;
  }
}
~~~

The media rule is at the CSS root, and the selector remains inside the condition.

## 2. Use a compile-time breakpoint value

### Sass source

~~~scss
$wide-layout: 68rem;

.page {
  padding-inline: 1rem;

  @media (min-width: $wide-layout) {
    max-width: 72rem;
    margin-inline: auto;
    padding-inline: 2rem;
  }
}
~~~

### Compiled CSS

~~~css
.page {
  padding-inline: 1rem;
}

@media (min-width: 68rem) {
  .page {
    max-width: 72rem;
    margin-inline: auto;
    padding-inline: 2rem;
  }
}
~~~

Sass substitutes the value during compilation. The browser receives an ordinary CSS media query.

## 3. Build a breakpoint mixin with a map

~~~scss
@use "sass:map";

$breakpoints: (
  "medium": 48rem,
  "wide": 68rem
);

@mixin above($name) {
  $value: map.get($breakpoints, $name);

  @if $value == null {
    @error "Unknown breakpoint: #{$name}";
  }

  @media (min-width: $value) {
    @content;
  }
}

.sidebar {
  display: none;

  @include above("medium") {
    display: block;
  }

  @include above("wide") {
    position: sticky;
    top: 1rem;
  }
}
~~~

The mixin provides one breakpoint source and lets each caller supply the declarations that belong inside the media condition.

## 4. Combine nested media queries

### Sass source

~~~scss
@media screen {
  .navigation {
    @media (min-width: 48rem) {
      display: flex;
    }
  }
}
~~~

### Combined CSS

~~~css
@media screen and (min-width: 48rem) {
  .navigation {
    display: flex;
  }
}
~~~

Sass combines the conditions when it can express them together. Always inspect complex nested-query output.

## 5. Add a feature-query enhancement with a fallback

~~~scss
$grid-display: grid;

.gallery {
  display: flex;
  flex-wrap: wrap;

  @supports (display: $grid-display) {
    display: $grid-display;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 1rem;
  }
}
~~~

Expected CSS:

~~~css
.gallery {
  display: flex;
  flex-wrap: wrap;
}

@supports (display: grid) {
  .gallery {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 1rem;
  }
}
~~~

Browsers that do not match the supports query keep the fallback declarations.

## 6. Require both a viewport and feature condition

~~~scss
.dashboard {
  display: block;

  @media (min-width: 60rem) {
    @supports (display: grid) {
      display: grid;
      grid-template-columns: 16rem minmax(0, 1fr);
      gap: 2rem;
    }
  }
}
~~~

Expected CSS:

~~~css
.dashboard {
  display: block;
}

@media (min-width: 60rem) {
  @supports (display: grid) {
    .dashboard {
      display: grid;
      grid-template-columns: 16rem minmax(0, 1fr);
      gap: 2rem;
    }
  }
}
~~~

The grid declarations are applied only when both conditions pass.

## 7. Wrap a motion preference in a reusable mixin

~~~scss
@mixin motion-safe {
  @media (prefers-reduced-motion: no-preference) {
    @content;
  }
}

.toast {
  opacity: 1;

  @include motion-safe {
    transition: opacity 180ms ease-out;
  }
}
~~~

Expected CSS:

~~~css
.toast {
  opacity: 1;
}

@media (prefers-reduced-motion: no-preference) {
  .toast {
    transition: opacity 180ms ease-out;
  }
}
~~~

The content block is emitted only inside the media condition defined by the mixin.

## 8. Add a container query inside a component

~~~scss
.product-card {
  container: product-card / inline-size;

  .product-card__content {
    display: block;
  }

  @container product-card (min-width: 32rem) {
    .product-card__content {
      display: grid;
      grid-template-columns: 7rem minmax(0, 1fr);
      gap: 1rem;
    }
  }
}
~~~

Sass bubbles the container rule while retaining its selector relationship. The browser evaluates the named container query at runtime; Sass only emits the CSS.

## 9. Use range syntax in a media query

~~~scss
$compact-limit: 42rem;

@media (width <= $compact-limit) {
  .toolbar__label {
    display: none;
  }
}
~~~

This is modern CSS media-feature range syntax. Sass compiles the Sass value into the query. Check browser support against the project's target browsers before choosing this syntax.

## 10. Emit keyframes from a mixin

~~~scss
@mixin fade-in($name, $duration) {
  @keyframes #{$name} {
    from {
      opacity: 0;
      transform: translateY(0.5rem);
    }

    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  animation-name: $name;
  animation-duration: $duration;
  animation-timing-function: ease-out;
}

.dialog {
  @include fade-in("dialog-entry", 180ms);
}
~~~

The interpolation creates a dynamic keyframe identifier. The same argument is used for the animation declaration.

## 11. Generate a CSS at-rule name with interpolation

~~~scss
$rule-name: "layer";

@#{$rule-name} utilities {
  .visually-hidden {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip-path: inset(50%);
  }
}
~~~

The generated CSS uses an @layer rule. Dynamic rule names are useful only when a real abstraction needs them; ordinary static at-rule names are easier to read.

## 12. Interpolate a named container value

~~~scss
$container-name: product-card;
$container-limit: 32rem;

@container #{$container-name} (min-width: $container-limit) {
  .product-card__content {
    display: grid;
    grid-template-columns: 7rem minmax(0, 1fr);
  }
}
~~~

Interpolation inserts the Sass value into the at-rule prelude. Sass does not evaluate whether an element with that container name exists.

## 13. Define a font face using a Sass value

~~~scss
$font-file: "/fonts/field-sans.woff2";

@font-face {
  font-family: "Field Sans";
  src: url(#{$font-file}) format("woff2");
  font-style: normal;
  font-weight: 100 900;
  font-display: swap;
}
~~~

Expected CSS:

~~~css
@font-face {
  font-family: "Field Sans";
  src: url(/fonts/field-sans.woff2) format("woff2");
  font-style: normal;
  font-weight: 100 900;
  font-display: swap;
}
~~~

This at-rule contains font descriptors rather than selector rules. Its block should follow the CSS @font-face grammar.

## 14. Declare layer order and add layered rules

~~~scss
@layer reset, base, components, utilities;

@layer base {
  body {
    margin: 0;
    font-family: system-ui, sans-serif;
  }
}

@layer components {
  .button {
    border: 0;
    border-radius: 0.5rem;
    padding: 0.75rem 1rem;
  }
}
~~~

Sass emits the layer statement and blocks. The browser applies CSS cascade-layer priority.

## 15. Escape selector nesting with @at-root

~~~scss
.card {
  color: #263238;

  @at-root .theme-dark & {
    color: white;
    background: #20242a;
  }
}
~~~

Expected CSS:

~~~css
.card {
  color: #263238;
}

.theme-dark .card {
  color: white;
  background: #20242a;
}
~~~

The theme selector is emitted at the root instead of as a descendant of .card.

## 16. Use @at-root with a selector function

~~~scss
@use "sass:selector";

@mixin parent-and-child($child) {
  @at-root #{selector.unify(&, $child)} {
    @content;
  }
}

.form-field {
  @include parent-and-child("input") {
    border-color: #1769aa;
  }
}
~~~

One possible output is:

~~~css
input.form-field {
  border-color: #1769aa;
}
~~~

The selector module combines the current selector with the requested child selector. The at-root rule prevents Sass from automatically nesting the generated selector a second time.

## 17. Compare a Sass condition with a CSS condition

### Sass decides during compilation

~~~scss
$high-contrast: false;

@if $high-contrast {
  .notice {
    border: 2px solid currentColor;
  }
}
~~~

When $high-contrast is false, Sass emits no notice rule from this block.

### The browser decides at runtime

~~~scss
@media (forced-colors: active) {
  .notice {
    border: 2px solid currentColor;
  }
}
~~~

The media rule is emitted. The browser applies the declaration when the condition matches.

## 18. Use a generic CSS at-rule

~~~scss
@starting-style {
  .popover[popover]:popover-open {
    opacity: 0;
    transform: translateY(0.5rem);
  }
}
~~~

Sass preserves the at-rule and compiles the nested style rule. Browser support and behavior are determined by CSS implementations, not by Sass.

## 19. See how multiple rules bubble

### Sass source

~~~scss
.alert {
  color: #7a1d1d;

  @media (min-width: 48rem) {
    display: grid;
  }

  @supports (color: color-mix(in srgb, red, white)) {
    border-color: color-mix(in srgb, #7a1d1d, white 35%);
  }
}
~~~

### Output shape

~~~css
.alert {
  color: #7a1d1d;
}

@media (min-width: 48rem) {
  .alert {
    display: grid;
  }
}

@supports (color: color-mix(in srgb, red, white)) {
  .alert {
    border-color: color-mix(in srgb, #7a1d1d, white 35%);
  }
}
~~~

The style rule is repeated under each condition in the generated CSS. Sass does not combine unrelated conditional blocks just because they share a selector.

## 20. Practice tasks

1. Add a media rule inside a component and compare the source nesting with the emitted CSS.
2. Replace a repeated breakpoint value with a Sass variable.
3. Build a breakpoint mixin that accepts a content block.
4. Add a fallback declaration and an @supports enhancement.
5. Combine a media condition and a supports condition, then explain when the declarations apply.
6. Define a named container and write a container query for its contents.
7. Generate one keyframe identifier with interpolation and use it in animation-name.
8. Add component CSS to a named cascade layer.
9. Use @at-root to generate a theme selector outside a component's normal nesting.
10. Compare the result of a Sass @if with the output of a browser-evaluated CSS condition.

## Quick reference

~~~scss
$wide: 68rem;

.component {
  @media (min-width: $wide) {
    padding: 2rem;
  }

  @supports (display: grid) {
    display: grid;
  }
}

@layer components {
  .button {
    border-radius: 0.5rem;
  }
}
~~~

- Sass evaluates its own directives.
- Sass compiles and emits CSS at-rules.
- Conditional at-rules can bubble out of style rules.
- Media queries can be combined when Sass can express the result.
- Interpolation creates dynamic CSS text.
- @at-root changes Sass style-rule nesting; its filters control enclosing at-rules.

## Related material

- [Lesson: Sass with CSS at-rules](../lessons/036-sass-and-css-at-rules.md)
- [Q&A: Sass with CSS at-rules](../qnas/036-sass-and-css-at-rules.md)
- [Sass: CSS at-rules](https://sass-lang.com/documentation/at-rules/css/)
- [Sass: Interpolation](https://sass-lang.com/documentation/interpolation/)
- [Sass: @at-root](https://sass-lang.com/documentation/at-rules/at-root/)
- [Sass: Media query behavior](https://sass-lang.com/documentation/breaking-changes/media-logic/)
