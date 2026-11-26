# 036 — Sass with CSS At-Rules

## Learning goals

After this lesson, you should be able to:

- Recognize which at-rules Sass evaluates and which it passes through as CSS.
- Nest CSS at-rules inside Sass style rules and understand how Sass bubbles them in the output.
- Use Sass values in media and supports conditions without unnecessary interpolation.
- Read and write nested media queries, and verify how Sass combines them.
- Use conditional at-rules such as `@media`, `@supports`, and `@container` with reusable mixins.
- Generate dynamic at-rule names and values safely with interpolation.
- Work with `@keyframes`, `@font-face`, and cascade layers.
- Use `@at-root` when a selector must escape Sass's normal nesting.
- Inspect generated CSS instead of assuming nested source maps directly to nested CSS.

## 1. Sass and CSS both use at-rules

CSS at-rules begin with `@`. Sass supports CSS at-rules and understands special behavior for a few common rules, while preserving general support for CSS rules it does not interpret specially.

A CSS at-rule can have a value, a block, or both:

~~~scss
@charset "UTF-8";

@font-face {
  font-family: "Field Sans";
  src: url("/fonts/field-sans.woff2") format("woff2");
}

@layer reset, base, components;

@custom-media --wide-screen (min-width: 60rem);
~~~

Sass will usually emit these as CSS. Sass also adds its own at-rules, such as `@use`, `@mixin`, `@include`, `@function`, and flow-control rules. These are evaluated by Sass and do not appear in the generated CSS as Sass instructions.

The first question when reading a rule is: “Is this a CSS rule Sass should preserve, or a Sass instruction that changes how Sass builds the output?”

## 2. CSS at-rules are generally passed through

Sass is designed to remain compatible with CSS as it evolves. It has general support for CSS at-rules such as `@container`, `@layer`, `@scope`, and `@starting-style`, even when Sass has no special evaluation behavior for them.

~~~scss
@container card (min-width: 30rem) {
  .card__body {
    display: grid;
    grid-template-columns: 8rem 1fr;
  }
}

@layer components {
  .notice {
    border-inline-start: 0.25rem solid rebeccapurple;
  }
}

@starting-style {
  dialog[open] {
    opacity: 0;
  }
}
~~~

These are CSS features. Sass emits the at-rules and compiles the Sass source inside them. Sass does not make a browser support a CSS feature; the browser decides how to apply the emitted CSS.

## 3. Nest an at-rule inside a style rule

A conditional rule can be written where it is relevant to a selector:

~~~scss
.card {
  padding: 1rem;

  @media (min-width: 48rem) {
    padding: 1.5rem;
  }
}
~~~

Sass moves the media rule to the top level in the generated CSS and places the selector inside it:

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

This is called bubbling. It keeps related declarations together in the Sass source while producing the standard CSS structure expected by browsers.

## 4. At-rules can be nested inside other CSS at-rules

CSS at-rules can also be nested where CSS allows a block:

~~~scss
@media screen {
  .card {
    @supports (display: grid) {
      display: grid;
      grid-template-columns: 1fr 2fr;
    }
  }
}
~~~

Sass preserves the surrounding conditions and bubbles the inner rule as needed. The compiled CSS remains a nested conditional structure:

~~~css
@media screen {
  @supports (display: grid) {
    .card {
      display: grid;
      grid-template-columns: 1fr 2fr;
    }
  }
}
~~~

Use nesting to express a real relationship between conditions. Deep stacks of conditional rules are harder to review and can make it harder to tell which browsers or layouts receive a declaration.

## 5. Use Sass variables directly in media feature values

The `@media` rule supports SassScript in media feature queries:

~~~scss
$compact-layout: 42rem;
$wide-layout: 68rem;

.page {
  padding-inline: 1rem;

  @media (min-width: $compact-layout) {
    padding-inline: 2rem;
  }

  @media (min-width: $wide-layout) {
    max-width: 72rem;
    margin-inline: auto;
  }
}
~~~

The variables are evaluated at compile time. The output contains concrete CSS values:

~~~css
.page {
  padding-inline: 1rem;
}

@media (min-width: 42rem) {
  .page {
    padding-inline: 2rem;
  }
}

@media (min-width: 68rem) {
  .page {
    max-width: 72rem;
    margin-inline: auto;
  }
}
~~~

When a variable is valid as a SassScript expression, write it directly. Interpolation is not needed just to insert a normal Sass value into a media feature.

## 6. Build a reusable media-query mixin

A mixin can standardize breakpoint usage across components:

~~~scss
@use "sass:map";

$breakpoints: (
  "medium": 48rem,
  "large": 72rem
);

@mixin above($name) {
  $breakpoint: map.get($breakpoints, $name);

  @if $breakpoint == null {
    @error "Unknown breakpoint: #{$name}";
  }

  @media (min-width: $breakpoint) {
    @content;
  }
}

.sidebar {
  display: none;

  @include above("medium") {
    display: block;
  }

  @include above("large") {
    position: sticky;
    top: 1rem;
  }
}
~~~

In production code, load the appropriate built-in module if the mixin uses a namespaced built-in. This example focuses on how a mixin wraps the conditional CSS and yields its content inside the media rule.

A breakpoint mixin should receive values that match the project's layout system. Avoid making a mixin so generic that callers can create accidental, inconsistent queries.

## 7. Nest and combine media queries carefully

Sass can merge media queries nested within other media queries when it can represent the combined condition:

~~~scss
@media screen {
  .menu {
    @media (min-width: 48rem) {
      display: flex;
    }
  }
}
~~~

A combined output can be represented as:

~~~css
@media screen and (min-width: 48rem) {
  .menu {
    display: flex;
  }
}
~~~

The output is still governed by CSS media-query rules. When nested conditions cannot be combined cleanly, Sass may preserve nested media rules instead. Inspect the compiled CSS for the actual conditions and order rather than relying on a hand-written mental rewrite.

For simple responsive styles, keep the media condition close to the selector it changes. For complex combinations, write the condition clearly and verify the generated result.

## 8. Use @supports for feature queries

`@supports` lets CSS conditionally apply declarations based on browser feature support. Sass compiles Sass values in declaration queries and preserves the CSS feature test:

~~~scss
$preferred-display: grid;

.layout {
  display: flex;

  @supports (display: $preferred-display) {
    display: $preferred-display;
    grid-template-columns: 12rem 1fr;
  }
}
~~~

Generated CSS:

~~~css
.layout {
  display: flex;
}

@supports (display: grid) {
  .layout {
    display: grid;
    grid-template-columns: 12rem 1fr;
  }
}
~~~

This example provides a fallback and an enhanced layout. A supports query should describe a browser capability, not a Sass condition. Use Sass `@if` when the decision is known during compilation; use `@supports` when the browser should decide at runtime.

## 9. Combine media and supports conditions

A selector can respond to both viewport size and feature support:

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

Output:

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

The declarations in the nested block require both conditions. Keep a usable fallback outside the conditional block when older or less capable browsers need one.

## 10. Use container queries as CSS at-rules

Sass passes through container query syntax and compiles nested Sass selectors:

~~~scss
.product-card {
  container: product-card / inline-size;

  &__content {
    display: block;
  }

  @container product-card (min-width: 32rem) {
    .product-card__content {
      display: grid;
      grid-template-columns: 7rem 1fr;
      gap: 1rem;
    }
  }
}
~~~

Because the conditional block is nested inside `.product-card`, Sass bubbles it into CSS. The selector inside the container block is written explicitly here so it is easy to read; use nesting consistently with the selector structure in your codebase.

Container queries are evaluated by the browser using the named container. A Sass variable can generate a CSS value, but Sass does not evaluate the browser's container size.

## 11. Write keyframes and dynamic animation names

`@keyframes` has special child syntax. Its children are keyframe selectors such as `from`, `to`, or percentages rather than ordinary nested selectors:

~~~scss
@keyframes fade-in {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.dialog {
  animation: fade-in 180ms ease-out;
}
~~~

Sass can interpolate a generated name when a mixin creates both the keyframes and the declaration that refers to them:

~~~scss
@use "sass:color";

@mixin pulse-animation($name, $color) {
  @keyframes #{$name} {
    from {
      box-shadow: 0 0 0 0 color.adjust($color, $alpha: -1);
    }

    to {
      box-shadow: 0 0 0 0.75rem color.adjust($color, $alpha: -0.75);
    }
  }

  animation-name: $name;
  animation-duration: 700ms;
  animation-iteration-count: 2;
}

.status-dot {
  @include pulse-animation("status-pulse", #1769aa);
}
~~~

Interpolation is needed in the at-rule name because the keyframe identifier is dynamic. Keep the generated identifier valid CSS and avoid defining the same named keyframes with conflicting bodies.

## 12. Use interpolation for dynamic at-rule names and values

At-rule names and values can contain interpolation:

~~~scss
$vendor: "webkit";

@#{$vendor}-keyframes slide-in {
  from {
    opacity: 0;
    transform: translateX(1rem);
  }

  to {
    opacity: 1;
    transform: translateX(0);
  }
}
~~~

This generates a vendor-prefixed at-rule name. Vendor prefixes should generally come from a maintained compatibility strategy rather than being added by hand without a browser-support reason.

Interpolation is also useful for building an at-rule value from Sass variables:

~~~scss
$layer-name: "components";

@layer #{$layer-name} {
  .button {
    border-radius: 0.5rem;
  }
}
~~~

Do not interpolate numbers unnecessarily. Interpolation creates text and can bypass Sass's unit checks. Prefer normal Sass expressions where the grammar accepts them.

## 13. Put reusable rules inside @font-face and other declaration at-rules

At-rules such as `@font-face` contain declarations rather than style selectors:

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

Sass emits this as a CSS at-rule. Interpolation can be used to build a URL or other value when necessary. If the value can be written as a normal Sass expression, prefer that over interpolation.

Other CSS at-rules may have their own block grammar. For example, `@keyframes` expects keyframe blocks, while `@font-face` expects declarations. Sass does not make invalid CSS block structures valid.

## 14. Use cascade layers to organize emitted styles

Sass can organize styles inside CSS cascade layers:

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
    padding: 0.75rem 1rem;
  }
}
~~~

The layer order statement and the layer blocks are emitted as CSS. Sass does not decide the CSS cascade priority; the browser applies the cascade-layer rules.

If styles are emitted from several Sass modules, make sure the layer order is established in an intentional entrypoint and that modules place their CSS in the expected layers.

## 15. Use @at-root to escape selector nesting deliberately

CSS at-rules such as `@media` bubble automatically. Sometimes a Sass-generated selector needs to escape an outer selector. `@at-root` tells Sass to emit the nested rule at the root:

~~~scss
.card {
  color: #263238;

  @at-root .theme-dark & {
    color: white;
    background: #20242a;
  }
}
~~~

Output:

~~~css
.card {
  color: #263238;
}

.theme-dark .card {
  color: white;
  background: #20242a;
}
~~~

Without `@at-root`, ordinary selector nesting would keep the new selector under `.card`. Use it only when the resulting selector should no longer be nested under the current rule.

`@at-root` also supports filters for controlling which at-rules remain around emitted CSS. Use those filters only when you understand the exact CSS structure they produce.

## 16. Put conditional output in a mixin with @content

A mixin can provide a reusable conditional wrapper:

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

Output:

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

This makes the conditional boundary visible at the call site. Keep the mixin name and condition aligned with the actual CSS behavior.

## 17. Distinguish a CSS condition from a Sass condition

Use CSS at-rules when the browser should decide whether a rule applies:

~~~scss
@media (min-width: 48rem) {
  .navigation {
    display: flex;
  }
}

@supports (display: grid) {
  .gallery {
    display: grid;
  }
}
~~~

Use Sass flow control when Sass should choose which CSS to generate during compilation:

~~~scss
$high-contrast-theme: false;

@if $high-contrast-theme {
  .notice {
    border: 2px solid currentColor;
  }
}
~~~

The Sass condition produces either CSS or no CSS at build time. The CSS condition is emitted and evaluated by the browser.

## 18. Common mistakes

### Expecting source nesting to remain in the same shape in CSS

Sass may bubble an at-rule out of a style rule or combine nested media queries. Inspect compiled CSS.

### Using @supports for a compile-time choice

A browser support rule is evaluated by the browser. Use Sass `@if` for build-time choices.

### Assuming every at-rule has the same block grammar

`@keyframes` uses keyframe selectors; `@font-face` uses declarations; `@media` contains CSS rules. Sass preserves CSS syntax but cannot make invalid structures valid.

### Interpolating values that Sass can already understand

Use a variable directly in a supported media feature or declaration. Save interpolation for dynamic identifiers, names, and grammar positions that require text insertion.

### Adding vendor-prefixed at-rules without a support requirement

Prefer the project's build tooling and documented browser support policy. Hand-maintained prefixes can create duplicate or stale output.

### Forgetting the base or fallback style

A conditional enhancement should be paired with a usable default where the design requires one.

### Nesting conditional rules too deeply

Deeply nested media, supports, and container rules make the applied conditions difficult to see. Keep the CSS condition structure readable and review the output.

## 19. Summary

Sass accepts CSS at-rules and adds useful behavior around nesting, SassScript values, and interpolation. It bubbles conditional rules out of style selectors, can combine nested media queries when possible, and preserves general CSS at-rules for the browser. Use `@media`, `@supports`, and `@container` for browser-evaluated conditions; use Sass flow control for compile-time decisions. Use interpolation only when a dynamic name or value needs to become CSS text, and use `@at-root` when a generated selector must escape its parent.

## Further reading

- [Sass: CSS at-rules](https://sass-lang.com/documentation/at-rules/css/)
- [Sass: Interpolation](https://sass-lang.com/documentation/interpolation/)
- [Sass: @at-root](https://sass-lang.com/documentation/at-rules/at-root/)
- [Sass: Style rules and nesting](https://sass-lang.com/documentation/style-rules/)
- [Sass: @extend within at-rules](https://sass-lang.com/documentation/at-rules/extend/)
- [Chapter 035: Migrating to Sass modules](035-migrating-to-sass-modules.md)
