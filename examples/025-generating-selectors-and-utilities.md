# 025 — Generating Selectors and Utility Classes: Examples

These examples use Sass maps, loops, interpolation, and the parent selector to generate CSS selectors. Sass expands the loops during compilation.

## 1. Generate display utilities from a list

~~~scss
$displays: block, flex, inline-flex;

@each $display in $displays {
  .u-display-#{$display} {
    display: $display;
  }
}
~~~

Generated CSS:

~~~css
.u-display-block {
  display: block;
}

.u-display-flex {
  display: flex;
}

.u-display-inline-flex {
  display: inline-flex;
}
~~~

The loop value is interpolated into each selector and used directly as the declaration value.

## 2. Generate padding utilities from named spacing tokens

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

The readable key forms the class suffix, while the Sass value provides the padding.

## 3. Generate more than one utility for each token

~~~scss
$spacing: (
  "small": 0.5rem,
  "medium": 1rem
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

This creates two utility families:

~~~css
.u-p-small {
  padding: 0.5rem;
}

.u-m-small {
  margin: 0.5rem;
}

.u-p-medium {
  padding: 1rem;
}

.u-m-medium {
  margin: 1rem;
}
~~~

Each map entry produces both rules.

## 4. Generate text color utilities

~~~scss
$text-colors: (
  "primary": #2457c5,
  "muted": #687385,
  "danger": #c43232
);

@each $name, $color in $text-colors {
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

.text-muted {
  color: #687385;
}

.text-danger {
  color: #c43232;
}
~~~

Class names come from the map keys, not from interpolating the color values.

## 5. Generate BEM-style component modifiers

~~~scss
$button-colors: (
  "primary": #2457c5,
  "danger": #c43232
);

.button {
  display: inline-flex;
  padding: 0.75rem 1rem;
  color: white;

  @each $name, $color in $button-colors {
    &--#{$name} {
      background-color: $color;
    }
  }
}
~~~

Generated CSS:

~~~css
.button {
  display: inline-flex;
  padding: 0.75rem 1rem;
  color: white;
}

.button--primary {
  background-color: #2457c5;
}

.button--danger {
  background-color: #c43232;
}
~~~

The **&** is replaced by the parent selector. The generated modifiers are siblings of **.button**, not descendants such as **.button .button--primary**.

## 6. Generate logical margin properties

A map can connect readable direction suffixes to CSS property names. Interpolate the property because Sass is constructing the declaration name.

~~~scss
$logical-margins: (
  "block-start": margin-block-start,
  "block-end": margin-block-end,
  "inline-start": margin-inline-start,
  "inline-end": margin-inline-end
);

@each $name, $property in $logical-margins {
  .u-margin-#{$name} {
    #{$property}: 1rem;
  }
}
~~~

Generated CSS begins:

~~~css
.u-margin-block-start {
  margin-block-start: 1rem;
}

.u-margin-block-end {
  margin-block-end: 1rem;
}

.u-margin-inline-start {
  margin-inline-start: 1rem;
}

.u-margin-inline-end {
  margin-inline-end: 1rem;
}
~~~

Keep generated property names in a controlled map so the output remains predictable.

## 7. Generate responsive utilities

The loop creates one media query and class for each named breakpoint. The browser evaluates the media query at runtime.

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

Sass generates the repeated media query structure. The breakpoint check remains in CSS for the browser.

## 8. Use variables directly in declaration values

Interpolation is useful for the class name, but is unnecessary for a normal declaration value.

~~~scss
$widths: (
  "narrow": 24rem,
  "wide": 48rem
);

@each $name, $width in $widths {
  .u-max-#{$name} {
    max-width: $width;
  }
}
~~~

Generated CSS:

~~~css
.u-max-narrow {
  max-width: 24rem;
}

.u-max-wide {
  max-width: 48rem;
}
~~~

The class suffix needs interpolation. The width should use the Sass variable directly so Sass keeps its number and unit behavior.

## 9. A small utility set

~~~scss
$spaces: (
  "compact": 0.5rem,
  "comfortable": 1rem,
  "spacious": 1.5rem
);

$text-colors: (
  "primary": #2457c5,
  "muted": #687385
);

@each $name, $space in $spaces {
  .u-gap-#{$name} {
    gap: $space;
  }
}

@each $name, $color in $text-colors {
  .u-text-#{$name} {
    color: $color;
  }
}
~~~

This generates named gap and text color utilities. The class names remain stable even if the underlying token values change.

## 10. Practice

### Practice A: create named gap utilities

~~~scss
$gaps: (
  "tight": 0.25rem,
  "regular": 0.75rem,
  "loose": 1.5rem
);

@each $name, $value in $gaps {
  .u-gap-#{$name} {
    gap: $value;
  }
}
~~~

### Practice B: add a component modifier

~~~scss
$badge-colors: (
  "info": #2457c5,
  "warning": #a45b00
);

.badge {
  padding: 0.25rem 0.5rem;

  @each $name, $color in $badge-colors {
    &--#{$name} {
      background-color: $color;
    }
  }
}
~~~

This generates **.badge--info** and **.badge--warning** alongside the base **.badge** rule.

### Practice C: choose class names

For a spacing value of **0.75rem**, prefer a key such as **regular** or **medium** rather than inserting **0.75rem** into the selector. Keep the token name in the map and the length in its value.

## Compile an example

Save a snippet as **utilities.scss** and compile it with Dart Sass:

~~~sh
sass utilities.scss utilities.css
~~~

Inspect the compiled CSS to confirm that each selector is valid, the values match the intended token, and the generated output is no larger than needed.

## Key takeaways

- Use interpolation to construct selectors and property names.
- Store readable class suffixes separately from CSS values.
- Use **&** for component modifier selectors.
- Use direct Sass variables in declaration values.
- Responsive utility media queries remain for the browser to evaluate.
- Keep generated utility APIs intentional and concise.
