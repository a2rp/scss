# 011 - Lists and List Functions

## Learning goals

After this lesson, you should be able to:

- Recognize comma-separated, space-separated, and slash-separated Sass lists.
- Explain how brackets, parentheses, and nested lists affect a value.
- Read and access list items with Sass's one-based indexes.
- Iterate over values with @each.
- Inspect a list's length, separator, brackets, and contents.
- Append, join, and replace list values without expecting mutation.
- Use lists to generate maintainable CSS.
- Use the sass:list module and identify common list-related mistakes.

## 1. What is a Sass list?

A Sass list is an ordered sequence of values. Lists can contain values such as numbers, strings, colors, other lists, or even null. Sass recognizes comma-separated and space-separated sequences as lists without requiring special brackets.

~~~scss
$font-stack: "Inter", "Segoe UI", sans-serif;
$padding: 0.5rem 1rem;
$border: 1px solid #c8ced8;
~~~

These are lists used as ordinary CSS values:

~~~scss
body {
  font-family: $font-stack;
}

.button {
  padding: $padding;
  border: $border;
}
~~~

Compiled CSS:

~~~css
body {
  font-family: "Inter", "Segoe UI", sans-serif;
}

.button {
  padding: 0.5rem 1rem;
  border: 1px solid #c8ced8;
}
~~~

Sass lists are useful both for CSS values and for storing values that Sass code will inspect or iterate over.

## 2. Separators: spaces and commas

The separator is part of a list's structure.

### Space-separated list

A space-separated list is common for shorthand properties:

~~~scss
$padding: 0.5rem 1rem 0.75rem;

.card {
  padding: $padding;
}
~~~

### Comma-separated list

A comma-separated list is common for font stacks, shadows, and alternatives:

~~~scss
$font-stack: "Inter", "Segoe UI", sans-serif;
$shadows: 0 2px 8px rgb(0 0 0 / 12%), 0 12px 32px rgb(0 0 0 / 8%);

body {
  font-family: $font-stack;
}

.card {
  box-shadow: $shadows;
}
~~~

Use the separator that matches the CSS grammar you want to emit.

## 3. Parentheses, brackets, and nested lists

Parentheses can group a list or help express a nested list. Square brackets create a bracketed list.

~~~scss
$coordinates: (10px, 20px);
$grid-lines: [main-start, content-start, content-end, main-end];
~~~

A nested list has lists as its own items:

~~~scss
$breakpoints: (small 30rem), (medium 48rem), (large 72rem);
~~~

The outer list contains three two-item space-separated lists. Parentheses distinguish the inner pairs from the outer comma separator.

Bracketed lists are useful when the resulting CSS requires square brackets, including grid line-name syntax:

~~~scss
.grid {
  grid-template-columns: [content-start] 1fr [content-end];
}
~~~

Sass also allows zero-item and one-item lists. An empty list can be written as () or [], and a one-item list can be written with a trailing comma or square brackets. Most list functions already treat a single value as a one-item list, so explicit one-item lists are not needed often.

An unbracketed empty list is not a valid CSS property value.

## 4. Slash-separated lists

CSS uses slashes in values such as font-size/line-height and modern color syntax. Sass historically treated slash as division, so a slash-separated list should be constructed explicitly with list.slash().

~~~scss
@use "sass:list";

$font-size-and-line-height: list.slash(1rem, 1.5);

.text {
  font: $font-size-and-line-height "Inter", sans-serif;
}
~~~

The resulting value is 1rem/1.5 "Inter", sans-serif.

Use math.div() for mathematical division. Use list.slash() when the slash is part of the CSS value's syntax.

~~~scss
@use "sass:list";
@use "sass:math";

$half: math.div(10px, 2);
$ratio: list.slash(1, 2);
~~~

These values have different meanings: $half is a calculated number, while $ratio is a slash-separated list.

## 5. Load the sass:list module

Use Sass's built-in list module for clear, namespaced list operations:

~~~scss
@use "sass:list";

$spacing: 0.25rem 0.5rem 1rem;

$first: list.nth($spacing, 1);
$count: list.length($spacing);
~~~

The @use rule makes the functions available under the list namespace. This lesson uses the module form rather than the older global function names.

## 6. List indexes start at 1

Sass list indexes are one-based:

- 1 refers to the first item.
- 2 refers to the second item.
- -1 refers to the last item.
- -2 refers to the second-to-last item.

~~~scss
@use "sass:list";

$spacing: 0.25rem 0.5rem 1rem 1.5rem;

$first: list.nth($spacing, 1);
$last: list.nth($spacing, -1);
~~~

Here $first is 0.25rem and $last is 1.5rem.

list.nth() returns the value at the requested index. An index outside the list's valid range is an error, so use list.length() or list.index() when the position might vary.

## 7. Measure and inspect a list

list.length() returns the number of items. list.separator() returns space, comma, or slash. list.is-bracketed() reports whether the list uses square brackets.

~~~scss
@use "sass:list";

$font-stack: "Inter", "Segoe UI", sans-serif;
$grid-lines: [main-start, content-start];

@debug list.length($font-stack); // 3
@debug list.separator($font-stack); // comma
@debug list.is-bracketed($font-stack); // false
@debug list.is-bracketed($grid-lines); // true
~~~

These functions are useful when writing reusable mixins and functions that accept lists from callers.

A single value also behaves like a one-item list for list functions:

~~~scss
@use "sass:list";

@debug list.length(1rem); // 1
@debug list.nth(1rem, 1); // 1rem
~~~

## 8. Find a value with list.index()

list.index() returns the one-based index of the first matching value. It returns null when there is no match.

~~~scss
@use "sass:list";

$valid-sides: top, right, bottom, left;

@debug list.index($valid-sides, bottom); // 3
@debug list.index($valid-sides, center); // null
~~~

Because null is falsey, list.index() can validate an argument:

~~~scss
@use "sass:list";

$valid-sides: top, right, bottom, left;

@mixin offset-border($side, $width, $color) {
  @if not list.index($valid-sides, $side) {
    @error "Unknown side: #{$side}. Use one of #{$valid-sides}.";
  }

  border-#{$side}: $width solid $color;
}

.card {
  @include offset-border(left, 0.25rem, royalblue);
}
~~~

If a caller passes center, compilation stops with a useful error rather than silently emitting an invalid side-specific property.

## 9. Loop over a list with @each

@each runs its block once for every item in a list.

~~~scss
$sizes: small, medium, large;

@each $size in $sizes {
  .badge-#{$size} {
    display: inline-block;
  }
}
~~~

The loop emits .badge-small, .badge-medium, and .badge-large.

Use the value itself in declarations when possible; use interpolation when the value becomes part of selector syntax.

### Destructure nested list items

When each item is itself a list, @each can assign its values to multiple variables:

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

Each inner list contains a theme name, background color, and foreground color.

## 10. Append one value with list.append()

list.append() returns a new list with one value added at the end.

~~~scss
@use "sass:list";

$base-spacing: 0.25rem 0.5rem;
$expanded-spacing: list.append($base-spacing, 1rem);

@debug $base-spacing; // 0.25rem 0.5rem
@debug $expanded-spacing; // 0.25rem 0.5rem 1rem
~~~

Use append() when adding one item. If the value being added is itself a list, append() keeps that list as one nested item.

~~~scss
@use "sass:list";

$groups: list.append((a, b), (c, d));

@debug $groups; // a, b, (c, d)
~~~

## 11. Combine lists with list.join()

list.join() combines the items from two lists into one new list.

~~~scss
@use "sass:list";

$horizontal: 0.5rem 1rem;
$vertical: 0.25rem 0.75rem;
$all-spacing: list.join($horizontal, $vertical);

@debug $all-spacing; // 0.5rem 1rem 0.25rem 0.75rem
~~~

The optional separator and bracketed arguments control the result's structure:

~~~scss
@use "sass:list";

$colors: list.join((navy, teal), (gold, coral), $separator: comma);
$grid-lines: list.join([content-start], [content-end]);

@debug $colors; // navy, teal, gold, coral
@debug $grid-lines; // [content-start content-end]
~~~

Use join() to combine lists. Use append() to add one value. If you append a list with join(), that list's items are flattened into the result instead of being kept as a nested item.

## 12. Replace an item with list.set-nth()

list.set-nth() returns a copy of the list with one item replaced. It does not change the original list.

~~~scss
@use "sass:list";

$colors: red, green, blue;
$accessible-colors: list.set-nth($colors, 2, darkgreen);

@debug $colors; // red, green, blue
@debug $accessible-colors; // red, darkgreen, blue
~~~

Negative indexes count from the end, just like list.nth().

## 13. Combine corresponding positions with list.zip()

list.zip() combines items at corresponding positions from multiple lists into nested lists.

~~~scss
@use "sass:list";

$names: small, medium, large;
$widths: 30rem, 48rem, 72rem;
$breakpoints: list.zip($names, $widths);

@debug $breakpoints;
// small 30rem, medium 48rem, large 72rem
~~~

This is useful when two related lists store parallel values. Keep their lengths aligned so each name is paired with the intended width.

## 14. Lists are immutable

Sass list values are immutable. A list function does not edit its input; it returns a new list. Assign the returned value if you want to use the updated list.

~~~scss
@use "sass:list";

$steps: 1, 2, 3;
$steps: list.append($steps, 4);

@debug $steps; // 1, 2, 3, 4
~~~

Without the reassignment, $steps still refers to the original list.

## 15. Maps are also list-like

Sass maps are key/value collections, but several list functions also treat maps as lists of key/value pairs. For map-specific operations, use the sass:map module. Use a list when order and position are the main ideas; use a map when values are looked up by a key.

~~~scss
$breakpoint-map: (
  small: 30rem,
  large: 72rem
);
~~~

For example, list.length() can count map entries, but list.nth() is not a clear way to retrieve a named breakpoint. A map lookup communicates that purpose better.

## 16. Common list mistakes

### Using a zero-based index

The first Sass list item is at index 1, not 0. Use -1 for the final item.

### Confusing append() with join()

append() adds one value and keeps a list value nested. join() combines the contents of two lists.

### Forgetting that list functions return new values

Sass does not mutate the original list. Save the return value if you need the updated list.

### Forgetting grouping parentheses

A comma can separate function arguments as well as list elements. Use parentheses to pass a comma-separated list as one function argument:

~~~scss
@use "sass:list";

$font-stack: ("Inter", "Segoe UI", sans-serif);
$second-font: list.nth($font-stack, 2);
~~~

### Treating a string or number as a special case

Sass list functions treat a single value as a one-item list. A color, number, or string can be passed to list.length() or list.nth().

### Writing slash division in new Sass code

Use math.div() for arithmetic and list.slash() for a slash-separated CSS list.

## 17. Worked example: spacing utilities

A list and @each can generate a consistent set of spacing classes:

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

This emits .gap-1 through .gap-4. The Sass value remains a number with a length unit in the declaration.

## 18. Practice checklist

Before moving on, make sure you can answer these questions:

1. What are the common Sass list separators?
2. What does a square-bracketed list preserve?
3. Which index refers to the first item? Which refers to the last?
4. What does list.index() return when it finds no match?
5. When should you use append() instead of join()?
6. Why must the result of a list function be assigned when you want to keep it?
7. How can @each iterate over nested two-item lists?
8. When should you use list.slash() and when should you use math.div()?
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="010-strings-and-interpolation.md">010 - Strings and Interpolation</a></td><td><a href="../qnas/011-lists-and-list-functions.md">Q&amp;A</a></td><td><a href="../examples/011-lists-and-list-functions.md">Example</a></td><td><a href="012-maps-and-map-functions.md">012 - Maps and Map Functions</a></td></tr>
  </tbody>
</table>
