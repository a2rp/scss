# 039 — Reusable Responsive Mixins

## Learning goals

After this lesson, you should be able to:

- Explain what a Sass mixin adds to responsive styles.
- Define and include mixins with required, optional, and keyword arguments.
- Use a breakpoint map to keep viewport thresholds in one place.
- Write a mobile-first mixin that wraps caller styles in a media query.
- Validate breakpoint names and report useful Sass errors.
- Pass information from a mixin to its content block when an API needs it.
- Distinguish viewport media queries from container queries and preference queries.
- Decide when a mixin helps and when ordinary CSS is clearer.

## 1. What a mixin does

A Sass mixin is a reusable piece of Sass that can emit declarations, nested rules, or at-rules where it is included. A mixin can accept arguments so each inclusion can adapt the output.

~~~scss
@mixin visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

.skip-link__text {
  @include visually-hidden;
}
~~~

A mixin is useful when a group of styles is repeated and benefits from a shared interface. It is not a browser feature: Sass expands the mixin while compiling, and the browser receives the resulting CSS.

## 2. Keep responsive CSS mobile-first

A mobile-first rule defines the base layout for narrow viewports, then adds changes as more space becomes available.

~~~scss
.page-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

@media (min-width: 48rem) {
  .page-grid {
    grid-template-columns: 16rem minmax(0, 1fr);
    gap: 1.5rem;
  }
}
~~~

The narrow layout is the default. The media query adds a second column only when the viewport is at least the chosen width. Choose thresholds when content needs them, rather than matching a particular phone or tablet model.

## 3. Put breakpoint values in one Sass map

~~~scss
$breakpoints: (
  "medium": 48rem,
  "wide": 72rem,
  "ultra": 90rem
);
~~~

Names should describe layout transitions in your project. They do not need to match device categories. A different site may need different thresholds or fewer breakpoints.

Sass variables and maps are compile-time values. They cannot be changed by the browser after CSS has been generated.

## 4. Build a breakpoint mixin

A reusable breakpoint mixin looks up a named minimum width and wraps the caller's styles in a media query.

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

Include it where responsive styles belong:

~~~scss
.page-grid {
  display: grid;
  grid-template-columns: 1fr;

  @include from("medium") {
    grid-template-columns: 16rem minmax(0, 1fr);
  }

  @include from("wide") {
    grid-template-columns: 18rem minmax(0, 1fr);
    gap: 2rem;
  }
}
~~~

Dart Sass compiles the nested includes to CSS with media queries around the .page-grid rules.

## 5. Understand the generated CSS

The preceding example produces CSS with the same behavior as:

~~~css
.page-grid {
  display: grid;
  grid-template-columns: 1fr;
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

The mixin does not make a responsive decision in the browser by itself. Sass writes the media query, and the browser evaluates that CSS condition.

## 6. Validate mixin arguments

A misspelled breakpoint should fail with a message that points to the problem.

~~~scss
@include from("medum") {
  .sidebar {
    display: block;
  }
}
~~~

Compilation stops with an error similar to:

~~~text
Error: Unknown breakpoint: medum.
~~~

An explicit error is more useful than silently generating a query with a missing or invalid width. The exact error formatting can vary between Sass versions.

## 7. Use required, default, and keyword arguments

~~~scss
@mixin horizontal-space($gap, $align: center) {
  display: flex;
  align-items: $align;
  gap: $gap;
}

.toolbar {
  @include horizontal-space(1rem);
}

.toolbar--top-aligned {
  @include horizontal-space(1rem, $align: flex-start);
}
~~~

Here, the gap is required and the alignment has a default. The second call uses a keyword argument so the chosen option is clear.

Keyword arguments are part of a mixin's interface. Renaming an argument can break callers that use its name, even if positional calls continue to work.

## 8. Give optional arguments clear meanings

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

Avoid APIs with many boolean arguments such as @include layout(true, false, true). Named options or a small number of focused mixins are easier to understand.

## 9. Content blocks make a mixin caller-controlled

The snippets below reuse the breakpoint map and sass:map module from Section 4; they focus on how the content block works.

A mixin that contains @content accepts a block of Sass from the caller. The mixin can place that block inside an at-rule or selector.

~~~scss
@mixin from($name) {
  $min-width: map.get($breakpoints, $name);

  @media (min-width: $min-width) {
    @content;
  }
}

.card {
  @include from("medium") {
    display: grid;
    grid-template-columns: 12rem minmax(0, 1fr);
  }
}
~~~

The caller chooses which styles belong at that breakpoint. A content block is lexically scoped: it can see variables in the caller's scope, but it cannot directly see local variables declared inside the mixin.

## 10. Pass a value to a content block when useful

This example also reuses the breakpoint map and sass:map module from Section 4. Dart Sass lets a mixin pass arguments to the block through @content(...). The caller declares the received values with using (...).

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

Only add this interface if the caller needs the value. Passing content arguments means callers must declare compatible using parameters, so changing that contract can be a breaking change.

## 11. Keep the map and mixin in a Sass module

~~~scss
// _responsive.scss
@use "sass:map";

$breakpoints: (
  "medium": 48rem,
  "wide": 72rem
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

Use the mixin through the module namespace:

~~~scss
// card.scss
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

The namespace shows where the mixin comes from and helps avoid accidental name collisions. If a module emits CSS as well as utilities, loading it with @use includes that CSS once per compilation.

## 12. Configure a breakpoint module when needed

A module can expose a default map that a consuming stylesheet configures before loading it.

~~~scss
// _responsive.scss
$breakpoints: (
  "medium": 48rem,
  "wide": 72rem
) !default;
~~~

~~~scss
// app.scss
@use "responsive" with (
  $breakpoints: (
    "medium": 50rem,
    "wide": 76rem
  )
);
~~~

Module configuration is a build-time choice and must happen the first time the module is loaded. A simple project may be clearer with one centrally edited map and no configurable module API.

## 13. Keep separate mixins for separate media features

Not every responsive adjustment is a viewport-width breakpoint. User preferences and output modes can use their own media queries.

~~~scss
@mixin reduced-motion {
  @media (prefers-reduced-motion: reduce) {
    @content;
  }
}

@mixin print {
  @media print {
    @content;
  }
}

.animated-panel {
  transition: transform 180ms ease;

  @include reduced-motion {
    transition: none;
  }

  @include print {
    box-shadow: none;
  }
}
~~~

A name such as reduced-motion communicates intent better than a generic media mixin that accepts arbitrary strings.

## 14. Use container queries when the component's container matters

A viewport breakpoint responds to the viewport. A container query responds to a queried ancestor container, which can better fit reusable components in sidebars, dialogs, or grids.

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

A small mixin can package the repeated condition:

~~~scss
@mixin container-from($min-width) {
  @container (min-width: $min-width) {
    @content;
  }
}

.card {
  @include container-from(36rem) {
    display: grid;
    grid-template-columns: 10rem minmax(0, 1fr);
  }
}
~~~

The ancestor must establish an appropriate query container. A container query is not interchangeable with a viewport media query; choose the one that matches the layout dependency.

## 15. Avoid brittle device-named breakpoints

~~~scss
$breakpoints: (
  "content-wide": 68rem,
  "navigation-expanded": 52rem
);
~~~

A breakpoint should describe a layout threshold, not a promise about a specific device. A tablet, small laptop, or zoomed desktop can have different available CSS viewport widths.

## 16. Avoid generating every breakpoint for every selector

Mixins emit CSS at each include. A large set of breakpoints and broad includes can make the stylesheet harder to follow and increase output.

~~~scss
.article {
  max-width: 42rem;

  @include from("wide") {
    max-width: 52rem;
  }
}
~~~

Use only responsive changes the design requires. Do not include an empty breakpoint block or create a mixin call when one plain @media rule would be clearer.

## 17. Do not use a mixin to hide a poor abstraction

A mixin can reduce duplication, but it can also obscure where styles came from or create an API with too many options. Ask whether the rule appears in multiple places, needs parameters, and emits predictable CSS.

Keep one-off responsive styles local when that makes the code easier to read. Read the compiled CSS when a mixin call's output is not obvious.

## 18. Mixins and other Sass features

| Feature | Use it when |
| --- | --- |
| Mixin | You need reusable declarations, nested rules, or a configurable content block |
| Function | You need to calculate and return one Sass value |
| Placeholder and @extend | Selectors share a semantic relationship and a combined selector is appropriate |
| CSS class | Markup should explicitly select a reusable behavior or component |
| Plain @media | One local responsive rule is clearer without an abstraction |

A mixin copies its emitted rules at each inclusion. Do not choose between mixins and @extend by assuming one always makes smaller files; choose the feature that best describes the relationship and output you want.

## 19. Common mistakes

### Using device names as a promise

A breakpoint called tablet can imply device detection. CSS evaluates a width or another media condition. Prefer a name that describes the layout or content transition.

### Forgetting that a mixin expands at compile time

A mixin does not make a reusable runtime CSS function. Its contents are included in generated CSS at each call.

### Reversing the mobile-first base

If the base rule assumes a wide layout, small viewports may overflow. Start with a layout that works in constrained space, then add enhancements.

### Pairing inclusive boundaries carelessly

Both min-width and max-width queries include their boundary. If paired at the same value, both can match at that exact width. A min-width-only mobile-first progression avoids an artificial one-pixel gap.

### Making a generic mixin accept raw media strings

A free-form string API can accept invalid or inconsistent conditions. Prefer small named mixins for common conditions and plain CSS for a unique query.

### Allowing invalid names to produce broken queries

Validate names in maps and raise a helpful @error instead of silently generating unusable CSS.

## 20. A practical workflow

1. Write a working narrow-viewport layout first.
2. Resize the viewport while checking when the content stops fitting comfortably.
3. Add a named threshold for a real layout change.
4. Keep threshold values together in a Sass map.
5. Build a mixin that validates its breakpoint name.
6. Include only the declarations that change.
7. Use a container query when the component depends on its container width.
8. Use separate preference mixins for motion, print, or input capabilities.
9. Inspect the compiled CSS to understand the output.
10. Remove abstractions that make a one-off rule harder to read.

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

- Use mobile-first base styles and add enhancements with minimum-width queries.
- Keep breakpoint values and names in one map.
- Use content blocks when the caller should supply responsive styles.
- Validate names and arguments at compilation time.
- Choose viewport media queries, container queries, or preference queries based on what drives the change.

## Related material

- [Previous lesson: Runtime theming and design tokens](038-runtime-theming-and-design-tokens.md)
- [Q&A: Reusable responsive mixins](../qnas/039-reusable-responsive-mixins.md)
- [Examples: Reusable responsive mixins](../examples/039-reusable-responsive-mixins.md)
- [Next lesson: Maps for design systems](040-maps-for-design-systems.md)
- [Sass: Mixins and content blocks](https://sass-lang.com/documentation/at-rules/mixin/)
- [Sass: Module loading with @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: Map functions](https://sass-lang.com/documentation/modules/map/)
- [Sass: @error](https://sass-lang.com/documentation/at-rules/error/)
- [CSS Media Queries Level 5](https://www.w3.org/TR/mediaqueries-5/)
