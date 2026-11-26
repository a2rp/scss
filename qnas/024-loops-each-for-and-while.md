# 024 - Loops: each, for, and while: Q&A

## Core concepts

### 1. What is a Sass loop?

A loop evaluates a Sass block repeatedly. Sass runs the loop during compilation and writes the resulting CSS. The browser receives the generated styles, not the loop.

### 2. What is @each used for?

**@each** processes every item in a known list or map. It is a good fit for repeated styles that vary by token, such as a theme color or display value.

~~~scss
$displays: block, flex, inline-flex;

@each $display in $displays {
  .display-#{$display} {
    display: $display;
  }
}
~~~

### 3. How does @each assign a list value?

With one loop variable, Sass assigns the current list item to that variable for each iteration. The body runs once per item.

### 4. How does @each iterate over a map?

Use two loop variables. Sass assigns the map key to the first variable and the corresponding value to the second.

~~~scss
$colors: ("primary": #2457c5, "danger": #c43232);

@each $name, $color in $colors {
  .text-#{$name} {
    color: $color;
  }
}
~~~

### 5. What is list destructuring in @each?

When the list contains inner lists, **@each** can assign the inner values to multiple variables. Each entry should have the structure the loop expects.

~~~scss
$sizes: ("small" 0.75rem, "large" 1.25rem);

@each $name, $size in $sizes {
  .text-#{$name} {
    font-size: $size;
  }
}
~~~

### 6. What is @for used for?

**@for** iterates through a numeric range. Use it when the number sequence is known, such as generating numbered utility classes.

~~~scss
@for $i from 1 through 3 {
  .order-#{$i} {
    order: $i;
  }
}
~~~

This creates three rules for **1**, **2**, and **3**.

### 7. What is the difference between to and through?

**to** excludes the ending number. **through** includes it.

~~~scss
@for $i from 1 to 4 {
  // $i is 1, 2, then 3.
}
~~~

~~~scss
@for $i from 1 through 4 {
  // $i is 1, 2, 3, then 4.
}
~~~

Choose carefully to avoid an extra or missing iteration.

### 8. Can @for count down?

Yes. Set the starting value higher than the ending value.

~~~scss
@for $layer from 3 through 1 {
  .layer-#{$layer} {
    z-index: $layer;
  }
}
~~~

The loop uses **3**, **2**, and **1**.

### 9. What does @while do?

**@while** repeats its body while its condition is truthy. The loop must update state so the condition eventually becomes false.

~~~scss
$i: 1;

@while $i <= 3 {
  .step-#{$i} {
    order: $i;
  }

  $i: $i + 1;
}
~~~

### 10. How can a @while loop become infinite?

It can keep running if the condition never becomes false. Common causes include forgetting to update the loop variable or updating it in the wrong direction. Initialize and update every value used by the condition.

### 11. Which loop should I choose?

- Use **@each** for each item in a known list or map.
- Use **@for** for a known numeric range.
- Use **@while** when repetition depends on a condition that changes over time.

Prefer **@each** or **@for** when either expresses the task clearly; they make the number or source of iterations easier to understand.

## Selectors and output

### 12. Why use interpolation in a generated selector?

Interpolation inserts a Sass value into selector text.

~~~scss
@each $name in "primary", "danger" {
  .button-#{$name} {
    display: inline-flex;
  }
}
~~~

This creates **.button-primary** and **.button-danger**.

### 13. Should raw CSS values always be inserted into class names?

No. Values containing punctuation, such as decimal lengths, can create awkward or invalid class names. Use meaningful token names in a map when you need readable selectors.

~~~scss
$spaces: ("small": 0.5rem, "large": 2rem);

@each $name, $space in $spaces {
  .gap-#{$name} {
    gap: $space;
  }
}
~~~

### 14. Does the loop remain in the compiled CSS?

No. Sass expands it during compilation. The generated CSS contains the repeated rules or declarations.

### 15. Can a loop emit declarations as well as selectors?

Yes. A loop can contain any valid Sass statements for its context. It can repeat declarations, nested selectors, or style rules.

### 16. Does every loop iteration need to generate a new selector?

No. A loop can repeat declarations or other Sass statements inside an existing rule. Create selectors only when each iteration needs a distinct rule.

### 17. Can a loop generate too much CSS?

Yes. Every iteration can add output. Keep the list or numeric range focused on styles the project needs, and avoid generating large sets of unused utilities.

## Choosing and debugging

### 18. Why might a @for loop produce one too few rules?

The range may use **to**, which excludes the ending value. Change it to **through** if the endpoint should be included.

### 19. Why does my @while loop not stop?

Check whether the loop changes the variable used in the condition and whether the update moves it toward the stopping condition.

### 20. Why is my generated selector missing the loop value?

Use interpolation around the variable in selector text, such as **.item-#{$name}**.

### 21. Is @while the best choice for a known list?

Usually not. **@each** communicates that the loop processes a collection and is generally clearer for a known list or map.

### 22. Are Sass loops evaluated by the browser?

No. Sass loops are compile-time constructs. If behavior should react to viewport size or other browser state, use CSS features such as media queries rather than a Sass loop.

## Quick review

- **@each** iterates over known values in lists, maps, and nested lists.
- **@for** counts through numeric ranges.
- **to** excludes the endpoint; **through** includes it.
- **@while** repeats until its condition becomes false, so update its state.
- Use interpolation when inserting values into selectors.
- Sass loops generate CSS during compilation and can increase output size.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="023-conditionals-and-boolean-logic.md">023 - Conditionals and Boolean Logic: Q&amp;A</a></td><td><a href="../lessons/024-loops-each-for-and-while.md">Lesson</a></td><td><a href="../examples/024-loops-each-for-and-while.md">Example</a></td><td><a href="025-generating-selectors-and-utilities.md">025 - Generating Selectors and Utility Classes: Q&amp;A</a></td></tr>
  </tbody>
</table>
