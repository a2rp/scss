# 011 — Lists and List Functions: Examples

Each example is intended to be read as a small, focused Sass snippet. If you combine them into one stylesheet, keep one @use statement for each module at the top.

## 1. Use lists directly as CSS values

A list can represent the values of a CSS property.

~~~scss
$font-stack: "Inter", "Segoe UI", sans-serif;
$padding: 0.5rem 1rem;
$border: 1px solid #c8ced8;

body {
  font-family: $font-stack;
}

.card {
  padding: $padding;
  border: $border;
}
~~~

Compiled CSS:

~~~css
body {
  font-family: "Inter", "Segoe UI", sans-serif;
}

.card {
  padding: 0.5rem 1rem;
  border: 1px solid #c8ced8;
}
~~~

## 2. Compare separators and brackets

Use list functions to inspect how Sass parsed a value.

~~~scss
@use "sass:list";

$space-list: 1rem 2rem 3rem;
$comma-list: red, green, blue;
$bracketed-list: [line-start line-end];

@debug list.separator($space-list); // space
@debug list.separator($comma-list); // comma
@debug list.is-bracketed($space-list); // false
@debug list.is-bracketed($bracketed-list); // true
~~~

Debug output is printed by the Sass compiler and does not become part of the CSS file.

## 3. Create nested lists

Parentheses keep inner list values together.

~~~scss
$breakpoints:
  (compact, 30rem),
  (wide, 60rem);

@debug $breakpoints;
~~~

This outer comma-separated list contains two inner comma-separated lists. Each inner list has a name and a width.

## 4. Read items with positive and negative indexes

Indexes start at 1. Negative indexes count backward from the end.

~~~scss
@use "sass:list";

$space-scale: 0.25rem 0.5rem 1rem 1.5rem;

$smallest: list.nth($space-scale, 1);
$second: list.nth($space-scale, 2);
$largest: list.nth($space-scale, -1);
$before-largest: list.nth($space-scale, -2);

@debug $smallest; // 0.25rem
@debug $second; // 0.5rem
@debug $largest; // 1.5rem
@debug $before-largest; // 1rem
~~~

## 5. Inspect list size and membership

list.length() counts items. list.index() returns an index or null when no match exists.

~~~scss
@use "sass:list";

$allowed-sides: top, right, bottom, left;

@debug list.length($allowed-sides); // 4
@debug list.index($allowed-sides, bottom); // 3
@debug list.index($allowed-sides, center); // null
~~~

## 6. Generate classes with @each

A simple list can drive a set of related CSS rules.

~~~scss
$levels: low, medium, high;

@each $level in $levels {
  .priority-#{$level} {
    display: block;
  }
}
~~~

Compiled CSS:

~~~css
.priority-low {
  display: block;
}

.priority-medium {
  display: block;
}

.priority-high {
  display: block;
}
~~~

The level is interpolated because it becomes part of the selector. Ordinary property values can use Sass variables directly.

## 7. Destructure nested lists in @each

Store a small record as a nested list, then assign its values to multiple loop variables.

~~~scss
$themes:
  (light, #ffffff, #172033),
  (dark, #172033, #f5f7fb);

@each $name, $background, $foreground in $themes {
  .theme-#{$name} {
    background-color: $background;
    color: $foreground;
  }
}
~~~

The loop produces:

~~~css
.theme-light {
  background-color: #ffffff;
  color: #172033;
}

.theme-dark {
  background-color: #172033;
  color: #f5f7fb;
}
~~~

## 8. Validate a list value before generating a property

Use list.index() to check a supported value. Report invalid input with @error.

~~~scss
@use "sass:list";

$valid-sides: top, right, bottom, left;

@mixin side-border($side, $width, $color) {
  @if not list.index($valid-sides, $side) {
    @error "Unsupported side: #{$side}. Choose one of #{$valid-sides}.";
  }

  border-#{$side}: $width solid $color;
}

.card {
  @include side-border(left, 0.25rem, royalblue);
}
~~~

The generated declaration is border-left. Passing center causes an informative compilation error.

## 9. Append one value

append() adds a value to the end of a new list. The original value is unchanged.

~~~scss
@use "sass:list";

$base: 0.25rem 0.5rem;
$expanded: list.append($base, 1rem);

@debug $base; // 0.25rem 0.5rem
@debug $expanded; // 0.25rem 0.5rem 1rem
~~~

## 10. Append a nested list versus join two lists

append() preserves a list argument as one nested item. join() combines the items from both lists.

~~~scss
@use "sass:list";

$first-group: (red, green);
$second-group: (blue, coral);

$appended: list.append($first-group, $second-group);
$joined: list.join($first-group, $second-group);

@debug $appended; // red, green, (blue, coral)
@debug $joined; // red, green, blue, coral
~~~

Choose append() when the new value should remain a single item. Choose join() when both lists should become one flat sequence.

## 11. Choose a separator when joining

The separator option controls the resulting list's separator.

~~~scss
@use "sass:list";

$first: 1rem 2rem;
$second: 3rem 4rem;

$space-result: list.join($first, $second);
$comma-result: list.join($first, $second, $separator: comma);

@debug list.separator($space-result); // space
@debug list.separator($comma-result); // comma
@debug $comma-result; // 1rem, 2rem, 3rem, 4rem
~~~

## 12. Replace an item with set-nth()

set-nth() returns a new list with one value replaced. Negative indexes work too.

~~~scss
@use "sass:list";

$colors: red, green, blue;
$accessible-colors: list.set-nth($colors, 2, darkgreen);
$last-updated: list.set-nth($colors, -1, navy);

@debug $colors; // red, green, blue
@debug $accessible-colors; // red, darkgreen, blue
@debug $last-updated; // red, green, navy
~~~

## 13. Zip parallel lists

Use zip() to combine values at matching positions.

~~~scss
@use "sass:list";

$names: compact, standard, wide;
$widths: 30rem, 48rem, 72rem;
$breakpoints: list.zip($names, $widths);

@debug $breakpoints;
~~~

The result is a comma-separated list of paired name-and-width lists:

~~~text
compact 30rem, standard 48rem, wide 72rem
~~~

Keep the source lists aligned so each name stays paired with its intended width.

## 14. Preserve bracketed CSS list syntax

A bracketed list can represent grid line names.

~~~scss
$main-track: [content-start content-end];

.page {
  display: grid;
  grid-template-columns: 1fr $main-track 1fr;
}
~~~

The bracketed line-name group remains bracketed in the emitted CSS.

## 15. Construct a slash-separated list

Use list.slash() when the slash belongs to the CSS value.

~~~scss
@use "sass:list";

$font-size-line-height: list.slash(1rem, 1.5);

.article {
  font: $font-size-line-height "Inter", sans-serif;
}
~~~

The font shorthand contains the size and line-height separated by a slash.

For arithmetic division, use math.div():

~~~scss
@use "sass:math";

$half-width: math.div(12rem, 2);

.sidebar {
  width: $half-width;
}
~~~

## 16. Pass a comma-separated list as one function argument

Parentheses make the full font stack a single argument to nth().

~~~scss
@use "sass:list";

$font-stack: ("Inter", "Segoe UI", sans-serif);
$fallback: list.nth($font-stack, 2);

@debug $fallback; // "Segoe UI"
~~~

## 17. A single value works as a one-item list

A scalar value can be passed directly to list functions.

~~~scss
@use "sass:list";

@debug list.length(2rem); // 1
@debug list.nth(2rem, 1); // 2rem
~~~

## 18. Use maps for named lookups

A map is useful when code needs values by key. Use sass:map to retrieve values by name.

~~~scss
@use "sass:map";

$breakpoints: (
  compact: 30rem,
  wide: 72rem
);

$wide-breakpoint: map.get($breakpoints, wide);

@media (min-width: $wide-breakpoint) {
  .layout {
    grid-template-columns: 16rem 1fr;
  }
}
~~~

Maps also have list-like properties in Sass, but map.get() expresses a named lookup more clearly than choosing an item by position.

## 19. A worked spacing scale

Use a map when each spacing number names a corresponding size. Iterate over its key/value pairs to generate utilities.

~~~scss
$spacing-scale: (
  1: 0.25rem,
  2: 0.5rem,
  3: 0.75rem,
  4: 1rem
);

@each $step, $size in $spacing-scale {
  .gap-#{$step} {
    gap: $size;
  }
}
~~~

Representative output:

~~~css
.gap-1 {
  gap: 0.25rem;
}

.gap-2 {
  gap: 0.5rem;
}

.gap-3 {
  gap: 0.75rem;
}

.gap-4 {
  gap: 1rem;
}
~~~

## 20. Practice tasks

1. Create a comma-separated list of three fallback font families and output it through font-family.
2. Generate a class for each value in compact, comfortable, and spacious.
3. Read the last item from a four-item list.
4. Check whether dashed appears in a border-style list.
5. Add one value to a list, then combine that result with another list.
6. Replace the second item of a list without changing the original variable.
7. Create a slash-separated font-size and line-height pair.
8. Use two parallel lists to pair breakpoint names and widths.

### Sample answers

~~~scss
@use "sass:list";

$font-stack: "Inter", "Segoe UI", sans-serif;
$densities: compact, comfortable, spacious;
$border-styles: solid, dotted, dashed;
$original: 1px 2px 3px;
$appended: list.append($original, 4px);
$combined: list.join($appended, 5px 6px);
$updated: list.set-nth($original, 2, 8px);
$font-metrics: list.slash(1rem, 1.5);
$names: compact, standard, wide;
$widths: 30rem, 48rem, 72rem;
$pairs: list.zip($names, $widths);

body {
  font-family: $font-stack;
}

@each $density in $densities {
  .density-#{$density} {
    display: block;
  }
}

@debug list.nth($original, -1);
@debug list.index($border-styles, dashed);
@debug $original;
@debug $appended;
@debug $combined;
@debug $updated;
@debug $font-metrics;
@debug $pairs;
~~~

The final item is 3px. The index of dashed is 3. $original remains unchanged; $updated is 1px 8px 3px.

## Further reading

- [Sass lists](https://sass-lang.com/documentation/values/lists/)
- [sass:list module](https://sass-lang.com/documentation/modules/list/)
- [Sass @each rule](https://sass-lang.com/documentation/at-rules/control/each/)
- [Sass map module](https://sass-lang.com/documentation/modules/map/)
