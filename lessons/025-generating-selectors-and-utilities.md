# 025 - Generating Selectors and Utility Classes

## Learning goals

After this lesson, you should be able to:

- Use interpolation to insert Sass values into selectors.
- Generate utility classes from lists and maps.
- Use map keys for readable class names and map values for declarations.
- Generate property names with interpolation when a utility needs dynamic properties.
- Build component modifier selectors with the parent selector.
- Generate responsive utility rules without mixing up Sass compilation and browser evaluation.
- Keep generated class names valid and the resulting CSS focused.

## 1. What does selector generation mean?

Sass can combine loops, variables, maps, and interpolation to create repeated selectors. This is useful for a family of related utility classes or component modifiers.

A utility class usually applies one clear style, such as a spacing value, text color, or display mode. Rather than copy similar rules by hand, describe the values in Sass and let a loop generate the CSS.

~~~scss
$displays: block, flex, inline-flex;

@each $display in $displays {
  .u-display-#{$display} {
    display: $display;
  }
}
~~~

Sass generates a rule for each display value. The browser receives ordinary CSS selectors and declarations; it does not receive the Sass loop.

## 2. Interpolate values into a selector

Interpolation uses **#{...}** to insert the result of a Sass expression into selector text.

~~~scss
$name: "primary";

.button-#{$name} {
  background-color: #2457c5;
}
~~~

Generated CSS:

~~~css
.button-primary {
  background-color: #2457c5;
}
~~~

Interpolation is needed here because the Sass value is part of the selector name. In a declaration value, a Sass variable can usually be used directly without interpolation:

~~~scss
$color: #2457c5;

.button {
  background-color: $color;
}
~~~

Prefer direct variable use for values. Interpolation turns expressions into unquoted strings and is most useful where Sass needs to construct text, such as a selector or property name.

## 3. Generate utilities from a named map

A map can associate a readable class suffix with the CSS value to emit.

~~~scss
$spacing: (
  "small": 0.5rem,
  "medium": 1rem,
  "large": 2rem
);

@each $name, $value in $spacing {
  .u-p-#{$name} {
    padding: $value;
  }
}
~~~

Generated CSS:

~~~css
.u-p-small {
  padding: 0.5rem;
}

.u-p-medium {
  padding: 1rem;
}

.u-p-large {
  padding: 2rem;
}
~~~

The map key becomes part of the class name. The map value becomes the declaration value. Named keys keep class names readable and avoid putting punctuation such as decimal points into selectors.

## 4. Generate more than one related utility

The same token map can produce more than one family of utilities.

~~~scss
$spacing: (
  "small": 0.5rem,
  "medium": 1rem,
  "large": 2rem
);

@each $name, $value in $spacing {
  .u-p-#{$name} {
    padding: $value;
  }

  .u-m-#{$name} {
    margin: $value;
  }
}
~~~

For each spacing token, Sass emits one padding class and one margin class. If the list of properties grows, keep the code easy to scan; a very generic generator can become harder to understand than explicit Sass.

## 5. Generate component modifiers with the parent selector

The parent selector **&** refers to the current outer selector. In a nested selector, it can add a suffix to form a component modifier.

~~~scss
$button-colors: (
  "primary": #2457c5,
  "danger": #c43232
);

.button {
  padding: 0.75rem 1rem;
  color: #ffffff;

  @each $name, $color in $button-colors {
    &--#{$name} {
      background-color: $color;
    }
  }
}
~~~

Generated selectors:

~~~css
.button {
  padding: 0.75rem 1rem;
  color: #ffffff;
}

.button--primary {
  background-color: #2457c5;
}

.button--danger {
  background-color: #c43232;
}
~~~

The nested **&--#{$name}** appends a BEM-style modifier suffix to **.button**. Sass combines the parent and generated suffix while compiling.

## 6. Generate property names carefully

Interpolation can also build a declaration property name. This is useful when a map records a relationship between a short token name and a CSS property.

~~~scss
$logical-margins: (
  "block-start": margin-block-start,
  "block-end": margin-block-end,
  "inline-start": margin-inline-start,
  "inline-end": margin-inline-end
);

@each $name, $property in $logical-margins {
  .u-m-#{$name} {
    #{$property}: 1rem;
  }
}
~~~

A generated rule looks like this:

~~~css
.u-m-block-start {
  margin-block-start: 1rem;
}
~~~

Use interpolation only for the property name. The declaration value remains a normal Sass value. Review generated properties carefully because an unexpected key can produce an invalid declaration.

## 7. Generate responsive utilities

Sass can generate a class inside each media query. The media query remains in the CSS, so the browser evaluates the breakpoint at runtime.

~~~scss
$breakpoints: (
  "tablet": 48rem,
  "desktop": 64rem
);

@each $name, $width in $breakpoints {
  @media (min-width: $width) {
    .#{$name}-u-hidden {
      display: none;
    }
  }
}
~~~

Generated CSS:

~~~css
@media (min-width: 48rem) {
  .tablet-u-hidden {
    display: none;
  }
}

@media (min-width: 64rem) {
  .desktop-u-hidden {
    display: none;
  }
}
~~~

The loop creates the repeated media query structure during compilation. Each media query still responds to the browser viewport.

## 8. Keep class names valid

Class names generated from Sass values must still follow CSS selector syntax. Use simple names made from letters, digits in safe positions, and hyphens.

Good map keys for selector names include **small**, **primary**, **block-start**, and **tablet**. A raw value such as **0.5rem** contains punctuation that does not make a reliable class suffix. Prefer a named key and store the raw CSS value separately.

~~~scss
// Prefer a descriptive key and a separate CSS value.
$spacing: (
  "compact": 0.5rem,
  "comfortable": 1rem
);
~~~

If a class name must include special characters, CSS escaping becomes relevant. Most utility systems are simpler when their generated class names use predictable tokens instead.

## 9. Avoid unnecessary interpolation

Interpolation is not needed just to place a Sass variable into a declaration value.

~~~scss
$width: 12px;

.correct {
  width: $width;
}
~~~

Avoid converting a number to a string to attach a unit:

~~~scss
// Avoid constructing a number with interpolation.
$width: 12;
$bad-width: #{$width}px;
~~~

Instead, store a number with its unit or use unit arithmetic:

~~~scss
$width: 12px;

.correct {
  width: $width;
}
~~~

Sass can then preserve and check units during calculations.

## 10. Keep generated CSS maintainable

Generated utilities become part of the project's class API. Before adding a generator, decide:

- Which class names should authors use?
- Which values should be available?
- Are the utilities compatible with the project's design tokens?
- Could two generated classes set the same property in confusing ways?
- Is the amount of generated CSS reasonable?

A short loop over a small token map is usually easy to maintain. A generator with many options and branches may be harder to understand than the repeated CSS it replaces.

## 11. Common mistakes

### Forgetting interpolation in selector text

Write **.u-#{$name}** when inserting a Sass variable into the class name. Without interpolation, Sass will not construct the intended selector.

### Inserting raw decimal values into class names

A suffix such as **0.5rem** contains punctuation. Use a descriptive map key such as **small** and keep **0.5rem** as the declaration value.

### Interpolating declaration values unnecessarily

Use a variable directly in a declaration. Interpolation is intended for constructing text and converts the result to an unquoted string.

### Forgetting that nested selectors include their parent

Inside **.button**, a selector such as **&--primary** generates **.button--primary**. Without **&**, ordinary nesting would create a descendant selector.

### Generating an accidental amount of CSS

Each token or breakpoint can add output. Keep the data set small and useful.

## 12. Practice

1. Create a map of named spacing tokens and generate **.u-gap-...** classes.
2. Create a map of named text colors and generate **.text-...** classes.
3. Add **primary** and **secondary** modifier selectors to a nested **.button** rule.
4. Generate a logical margin utility for each map entry.
5. Add a **tablet** breakpoint utility inside a media query.
6. Explain why a named key such as **small** is safer in a class name than a raw value such as **0.5rem**.
7. Explain why a declaration value should usually use **$value** directly instead of **#{$value}**.

## Key takeaways

- Interpolation inserts Sass values into selector and property-name text.
- Use map keys for stable, readable class names and map values for CSS declarations.
- Use **&** to create component modifiers from an outer selector.
- Keep selector tokens valid and avoid raw decimals or other punctuation in generated names.
- Sass creates the utility rules at compile time; browser media queries remain runtime conditions.
- Generate only the utilities the project needs.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="024-loops-each-for-and-while.md">024 - Loops: each, for, and while</a></td><td><a href="../qnas/025-generating-selectors-and-utilities.md">Q&amp;A</a></td><td><a href="../examples/025-generating-selectors-and-utilities.md">Example</a></td><td><a href="026-extend-placeholder-selectors.md">026 - @extend and Placeholder Selectors</a></td></tr>
  </tbody>
</table>
