# 024 - Loops: each, for, and while: Examples

These examples show list, map, numeric-range, and condition-based loops. Sass expands each loop during compilation; the browser receives only the resulting CSS.

## 1. @each over a list

Loop over display values to create one utility class per value.

~~~scss
$displays: block, flex, inline-flex;

@each $display in $displays {
  .display-#{$display} {
    display: $display;
  }
}
~~~

Generated CSS:

~~~css
.display-block {
  display: block;
}

.display-flex {
  display: flex;
}

.display-inline-flex {
  display: inline-flex;
}
~~~

The loop variable contains the current list item. Interpolation inserts that value into the class name.

## 2. @each over a map

Use a map when each generated rule needs a stable name and a value.

~~~scss
$colors: (
  "primary": #2457c5,
  "success": #16803c,
  "danger": #c43232
);

@each $name, $color in $colors {
  .text-#{$name} {
    color: $color;
  }
}
~~~

Generated CSS:

~~~css
.text-primary {
  color: #2457c5;
}

.text-success {
  color: #16803c;
}

.text-danger {
  color: #c43232;
}
~~~

The first loop variable receives the key, and the second receives the associated value.

## 3. @each over named spacing tokens

Named tokens keep generated selector names readable and valid.

~~~scss
$spaces: (
  "small": 0.5rem,
  "medium": 1rem,
  "large": 2rem
);

@each $name, $space in $spaces {
  .gap-#{$name} {
    gap: $space;
  }
}
~~~

Generated CSS:

~~~css
.gap-small {
  gap: 0.5rem;
}

.gap-medium {
  gap: 1rem;
}

.gap-large {
  gap: 2rem;
}
~~~

The selector uses the map key, while the declaration uses the corresponding value.

## 4. @each with nested lists

Each map-like entry can instead be represented as a small list of related values.

~~~scss
$icon-sizes: (
  "search" 1rem,
  "menu" 1.25rem,
  "close" 0.875rem
);

@each $name, $size in $icon-sizes {
  .icon-#{$name} {
    inline-size: $size;
    block-size: $size;
  }
}
~~~

Generated CSS:

~~~css
.icon-search {
  inline-size: 1rem;
  block-size: 1rem;
}

.icon-menu {
  inline-size: 1.25rem;
  block-size: 1.25rem;
}

.icon-close {
  inline-size: 0.875rem;
  block-size: 0.875rem;
}
~~~

Each inner list is destructured into **$name** and **$size**.

## 5. @for with an included endpoint

Use **through** when the ending number should be included.

~~~scss
@for $step from 1 through 4 {
  .space-#{$step} {
    margin-block-start: $step * 0.25rem;
  }
}
~~~

Generated CSS:

~~~css
.space-1 {
  margin-block-start: 0.25rem;
}

.space-2 {
  margin-block-start: 0.5rem;
}

.space-3 {
  margin-block-start: 0.75rem;
}

.space-4 {
  margin-block-start: 1rem;
}
~~~

The loop runs four times because both **1** and **4** are included.

## 6. @for with an excluded endpoint

Use **to** to stop before the ending number.

~~~scss
@for $layer from 1 to 4 {
  .layer-#{$layer} {
    z-index: $layer;
  }
}
~~~

Generated CSS:

~~~css
.layer-1 {
  z-index: 1;
}

.layer-2 {
  z-index: 2;
}

.layer-3 {
  z-index: 3;
}
~~~

There is no **.layer-4** because **to 4** excludes the endpoint.

## 7. Count down with @for

A **@for** loop can count down to a lower endpoint.

~~~scss
@for $layer from 3 through 1 {
  .layer-#{$layer} {
    z-index: $layer;
  }
}
~~~

Generated CSS:

~~~css
.layer-3 {
  z-index: 3;
}

.layer-2 {
  z-index: 2;
}

.layer-1 {
  z-index: 1;
}
~~~

Because the range uses **through**, the final value **1** is included.

## 8. @while with a changing counter

A **@while** loop needs an initial value, a condition, and an update that eventually makes the condition false.

~~~scss
$step: 1;

@while $step <= 3 {
  .level-#{$step} {
    padding: $step * 0.5rem;
  }

  $step: $step + 1;
}
~~~

Generated CSS:

~~~css
.level-1 {
  padding: 0.5rem;
}

.level-2 {
  padding: 1rem;
}

.level-3 {
  padding: 1.5rem;
}
~~~

After each iteration Sass increments **$step**. When it reaches **4**, the condition is false and the loop stops.

## 9. Combine tokens with a loop in a component system

A map can generate a family of named spacing classes without putting raw decimal values into selectors.

~~~scss
$inset-tokens: (
  "compact": 0.5rem,
  "comfortable": 1rem,
  "spacious": 1.5rem
);

@each $name, $value in $inset-tokens {
  .inset-#{$name} {
    padding: $value;
  }
}
~~~

Generated CSS:

~~~css
.inset-compact {
  padding: 0.5rem;
}

.inset-comfortable {
  padding: 1rem;
}

.inset-spacious {
  padding: 1.5rem;
}
~~~

The token names become part of the public class API. Choose names that are useful to the people who will write the HTML.

## 10. Compare @each, @for, and @while

### Use @each for known choices

~~~scss
$positions: start, center, end;

@each $position in $positions {
  .align-#{$position} {
    text-align: $position;
  }
}
~~~

### Use @for for a known count

~~~scss
@for $column from 1 through 3 {
  .column-#{$column} {
    grid-column: $column;
  }
}
~~~

### Use @while when the stopping condition is the important part

~~~scss
$scale: 1;

@while $scale < 5 {
  .scale-#{$scale} {
    order: $scale;
  }

  $scale: $scale * 2;
}
~~~

This **@while** loop emits values **1**, **2**, and **4**. It stops when the updated value becomes **8**, because **8 < 5** is false.

If the iterations are simply a known list or range, prefer **@each** or **@for**. Use **@while** for a condition-driven sequence like this one.

## 11. Practice and solutions

### Practice A: generate display classes

~~~scss
$displays: block, flex, grid;

@each $display in $displays {
  .display-#{$display} {
    display: $display;
  }
}
~~~

This generates **.display-block**, **.display-flex**, and **.display-grid**.

### Practice B: generate three widths

~~~scss
@for $column from 1 through 3 {
  .column-#{$column} {
    width: $column * 10%;
  }
}
~~~

The widths are **10%**, **20%**, and **30%**.

### Practice C: make a @while loop stop

~~~scss
$count: 1;

@while $count <= 3 {
  .count-#{$count} {
    order: $count;
  }

  $count: $count + 1;
}
~~~

The counter update ensures the condition eventually becomes false.

## Compile the examples

Save a snippet as **loops.scss**, then compile it with Dart Sass:

~~~sh
sass loops.scss loops.css
~~~

Compare the generated selectors and declarations with the expected output. When changing a range, check whether its endpoint should be included.

## Key takeaways

- Use **@each** for values from lists, maps, or nested lists.
- Use **@for** for numeric ranges and check whether the endpoint uses **to** or **through**.
- Use **@while** only when a changing condition determines when to stop.
- Interpolate loop values in generated selectors.
- Use descriptive tokens instead of raw values for class names.
- Every Sass loop is expanded during compilation and contributes to the generated CSS.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="023-conditionals-and-boolean-logic.md">023 - Conditionals and Boolean Logic: Examples</a></td><td><a href="../lessons/024-loops-each-for-and-while.md">Lesson</a></td><td><a href="../qnas/024-loops-each-for-and-while.md">Q&amp;A</a></td><td><a href="025-generating-selectors-and-utilities.md">025 - Generating Selectors and Utility Classes: Examples</a></td></tr>
  </tbody>
</table>
