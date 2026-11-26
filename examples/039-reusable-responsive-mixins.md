# 039 — Reusable Responsive Mixins: Examples

The snippets focus on Dart Sass and compile-time mixins. Breakpoint values are illustrative; choose values where your own content needs a layout change.

## 1. Define and include a simple mixin

~~~scss
@mixin reset-list {
  margin: 0;
  padding: 0;
  list-style: none;
}

.navigation ul {
  @include reset-list;
}
~~~

Output:

~~~css
.navigation ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
~~~

The mixin emits declarations at its include site.

## 2. Parameterize a reusable group of declarations

~~~scss
@mixin inline-gap($gap) {
  display: flex;
  gap: $gap;
}

.toolbar {
  @include inline-gap(1rem);
}

.breadcrumbs {
  @include inline-gap(0.5rem);
}
~~~

Each include supplies a different value for the required argument.

## 3. Add a default argument and use a keyword argument

~~~scss
@mixin stack($gap: 1rem, $alignment: stretch) {
  display: flex;
  flex-direction: column;
  align-items: $alignment;
  gap: $gap;
}

.article {
  @include stack;
}

.article__actions {
  @include stack($gap: 0.5rem, $alignment: flex-end);
}
~~~

The default keeps the common call short. The keyword call makes each optional choice visible.

## 4. Create a validated breakpoint map

~~~scss
@use "sass:map";

$breakpoints: (
  "medium": 48rem,
  "wide": 72rem,
  "ultra": 90rem
);

@mixin from($name) {
  @if not map.has-key($breakpoints, $name) {
    @error "Unknown breakpoint: #{$name}.";
  }

  $min-width: map.get($breakpoints, $name);

  @media (min-width: $min-width) {
    @content;
  }
}
~~~

The validation catches misspelled names before unusable CSS is shipped.

## 5. Include the breakpoint mixin in a component

This snippet uses the from mixin from the preceding example.

~~~scss
.page-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;

  @include from("medium") {
    grid-template-columns: 16rem minmax(0, 1fr);
  }

  @include from("wide") {
    grid-template-columns: 18rem minmax(0, 1fr);
    gap: 2rem;
  }
}
~~~

Compiled CSS:

~~~css
.page-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

@media (min-width: 48rem) {
  .page-grid {
    grid-template-columns: 16rem minmax(0, 1fr);
  }
}

@media (min-width: 72rem) {
  .page-grid {
    grid-template-columns: 18rem minmax(0, 1fr);
    gap: 2rem;
  }
}
~~~

The base style handles narrow viewports; the media queries enhance the layout at wider thresholds.

## 6. See the useful failure for an unknown breakpoint

~~~scss
@include from("medum") {
  .sidebar {
    display: block;
  }
}
~~~

The mixin raises a Sass error similar to:

~~~text
Error: Unknown breakpoint: medum.
~~~

Without validation, a bad name could lead to an invalid or confusing media query.

## 7. Keep one-off rules as plain media queries

~~~scss
.article-intro {
  max-width: 42rem;

  @media (min-width: 68rem) {
    max-width: 50rem;
  }
}
~~~

A mixin is not mandatory. If a condition appears once and the plain query is clear, keep the rule local.

## 8. Put mobile-first base styles before enhancements

~~~scss
.product-list {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

.product-list__item {
  min-width: 0;
}

@include from("medium") {
  .product-list {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@include from("wide") {
  .product-list {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}
~~~

Here the media queries wrap the full .product-list selector because the mixin is included at the top level. Including a mixin inside the selector instead would keep the responsive code next to the base rule.

## 9. Use a content block to keep related rules together

~~~scss
.card {
  display: block;
  padding: 1rem;

  @include from("medium") {
    display: grid;
    grid-template-columns: 12rem minmax(0, 1fr);
    padding: 1.5rem;
  }
}
~~~

The mixin supplies the media-query wrapper, while the component supplies the declarations. The content block is inserted where @content appears in the mixin.

## 10. Use a map value inside a content block

~~~scss
@mixin from-with-value($name) {
  $min-width: map.get($breakpoints, $name);

  @media (min-width: $min-width) {
    @content($min-width);
  }
}

.card {
  @include from-with-value("medium") using ($width) {
    max-width: calc(100% - $width);
  }
}
~~~

This example also assumes the map module and breakpoint map from Example 4. Use a content argument only when the caller needs the looked-up width.

## 11. Place utilities in a module

responsive.scss:

~~~scss
@use "sass:map";

$breakpoints: (
  "medium": 48rem,
  "wide": 72rem
);

@mixin from($name) {
  @if not map.has-key($breakpoints, $name) {
    @error "Unknown breakpoint: #{$name}.";
  }

  @media (min-width: map.get($breakpoints, $name)) {
    @content;
  }
}
~~~

card.scss:

~~~scss
@use "responsive";

.card {
  padding: 1rem;

  @include responsive.from("medium") {
    display: grid;
    grid-template-columns: 12rem minmax(0, 1fr);
    padding: 1.5rem;
  }
}
~~~

The responsive namespace makes the mixin's source clear. Sass resolves the module and its map while compiling card.scss.

## 12. Configure breakpoints for one build

responsive.scss:

~~~scss
@use "sass:map";

$breakpoints: (
  "medium": 48rem,
  "wide": 72rem
) !default;

@mixin from($name) {
  @if not map.has-key($breakpoints, $name) {
    @error "Unknown breakpoint: #{$name}.";
  }

  @media (min-width: map.get($breakpoints, $name)) {
    @content;
  }
}
~~~

app.scss:

~~~scss
@use "responsive" with (
  $breakpoints: (
    "medium": 50rem,
    "wide": 76rem
  )
);

.panel {
  @include responsive.from("medium") {
    display: grid;
    grid-template-columns: 14rem minmax(0, 1fr);
  }
}
~~~

Module configuration happens at build time and must be provided the first time the module is loaded. If only one set of values is needed, editing one central map is usually simpler.

## 13. Use a named reduced-motion mixin

~~~scss
@mixin reduced-motion {
  @media (prefers-reduced-motion: reduce) {
    @content;
  }
}

.drawer {
  transition: transform 180ms ease;

  @include reduced-motion {
    transition: none;
  }
}
~~~

The browser evaluates the preference query. The mixin only makes the repeated Sass pattern convenient.

## 14. Add print-specific output

~~~scss
@mixin print {
  @media print {
    @content;
  }
}

.navigation {
  @include print {
    display: none;
  }
}

.article {
  @include print {
    color: black;
    background: white;
    box-shadow: none;
  }
}
~~~

Print rules concern an output medium, not viewport width. Keep them separate from breakpoint helpers.

## 15. Compare viewport and container queries

A viewport-based mixin responds to the viewport:

~~~scss
@include from("medium") {
  .card {
    grid-template-columns: 12rem minmax(0, 1fr);
  }
}
~~~

A container-based rule responds to a query container:

~~~scss
.card-list {
  container-type: inline-size;
}

.card {
  display: block;
}

@container (min-width: 36rem) {
  .card {
    display: grid;
    grid-template-columns: 10rem minmax(0, 1fr);
  }
}
~~~

Use the viewport when the overall page width drives the change. Use a container when the component's own available space drives it.

## 16. Wrap a container query in a mixin

~~~scss
@mixin container-from($min-width) {
  @container (min-width: $min-width) {
    @content;
  }
}

.card-list {
  container-type: inline-size;
}

.card {
  display: block;

  @include container-from(36rem) {
    display: grid;
    grid-template-columns: 10rem minmax(0, 1fr);
  }
}
~~~

The card-list element establishes an inline-size query container. The card responds when that eligible ancestor's inline size reaches the threshold.

## 17. Avoid a mixin that takes arbitrary media strings

~~~scss
// Hard to validate and easy to call inconsistently:
@mixin media($condition) {
  @media #{$condition} {
    @content;
  }
}
~~~

A focused mixin is easier to use and review:

The focused version below assumes the validated breakpoint map and sass:map module from Example 4.

~~~scss
@mixin from($name) {
  $min-width: map.get($breakpoints, $name);

  @media (min-width: $min-width) {
    @content;
  }
}
~~~

Use a generic media wrapper only when the project has a strong reason for that flexible API and validates its inputs.

## 18. Avoid paired inclusive thresholds with a gap

~~~scss
@media (max-width: 48rem) {
  .layout {
    grid-template-columns: 1fr;
  }
}

@media (min-width: 48rem) {
  .layout {
    grid-template-columns: 12rem 1fr;
  }
}
~~~

At exactly 48rem both conditions match. In this particular example, the later declaration wins for the same property, but relying on that overlap can make styles harder to reason about.

A mobile-first progression is simpler:

~~~scss
.layout {
  grid-template-columns: 1fr;
}

@media (min-width: 48rem) {
  .layout {
    grid-template-columns: 12rem 1fr;
  }
}
~~~

## 19. Keep breakpoint names about layout needs

~~~scss
$breakpoints: (
  "navigation-expanded": 52rem,
  "content-wide": 68rem
);
~~~

These names say what changes in the design. They do not promise that a particular device will match the condition.

## 20. Inspect the compiled output when debugging

Given this mixin call:

~~~scss
.notice {
  padding: 0.75rem;

  @include from("medium") {
    padding: 1rem;
  }
}
~~~

Confirm that the compiled result contains the base rule and a media query with the intended selector and width. Sass source shows the reusable abstraction; the compiled CSS shows the browser-facing result.

## 21. Practice tasks

1. Create a mixin that accepts a gap value and applies it to a flex row.
2. Build a breakpoint map with two layout-oriented names.
3. Write a min-width mixin that validates the requested map key.
4. Use the mixin inside a component rule to change its grid at a wider width.
5. Add an optional alignment argument with a sensible default.
6. Create a reduced-motion mixin and use it to remove a nonessential transition.
7. Make a container-query example and ensure the ancestor establishes a query container.
8. Deliberately misspell a breakpoint and inspect the Sass error.
9. Compare the compiled CSS with the Sass mixin source.
10. Replace a one-use mixin with a plain media query if the abstraction makes the code less clear.

## Quick reference

~~~scss
@use "sass:map";

$breakpoints: (
  "medium": 48rem,
  "wide": 72rem
);

@mixin from($name) {
  @if not map.has-key($breakpoints, $name) {
    @error "Unknown breakpoint: #{$name}.";
  }

  @media (min-width: map.get($breakpoints, $name)) {
    @content;
  }
}

.panel {
  display: block;

  @include from("medium") {
    display: grid;
    grid-template-columns: 14rem minmax(0, 1fr);
  }
}
~~~

## Related material

- [Lesson: Reusable responsive mixins](../lessons/039-reusable-responsive-mixins.md)
- [Q&A: Reusable responsive mixins](../qnas/039-reusable-responsive-mixins.md)
- [Sass: Mixins and content blocks](https://sass-lang.com/documentation/at-rules/mixin/)
- [Sass: Module loading with @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: Map functions](https://sass-lang.com/documentation/modules/map/)
- [Sass: @error](https://sass-lang.com/documentation/at-rules/error/)
- [CSS Media Queries Level 5](https://www.w3.org/TR/mediaqueries-5/)
