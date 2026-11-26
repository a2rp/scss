# 009 — Slash Division and Calculations: Examples

Each snippet uses SCSS. Slash-separated values preserve CSS syntax; math.div() calculates Sass numbers; CSS calculations preserve browser-dependent work.

## 1. CSS font shorthand uses slash as a separator

~~~scss
body {
  font: 16px / 1.5 "Inter", sans-serif;
}
~~~

Generated CSS:

~~~css
body {
  font: 16px / 1.5 "Inter", sans-serif;
}
~~~

In this shorthand, the slash separates font size from line-height. It does not divide 16px by 1.5.

## 2. CSS aspect-ratio uses slash as a separator

~~~scss
.video {
  aspect-ratio: 16 / 9;
}
~~~

Generated CSS:

~~~css
.video {
  aspect-ratio: 16 / 9;
}
~~~

The browser interprets the two numbers as the width-to-height ratio.

## 3. CSS border-radius can use a slash separator

~~~scss
.avatar {
  border-radius: 24px / 12px;
}
~~~

Generated CSS:

~~~css
.avatar {
  border-radius: 24px / 12px;
}
~~~

The first radius describes the horizontal radii; the second describes the vertical radii.

## 4. Calculate a quotient with math.div()

~~~scss
@use "sass:math";

$half-size: math.div(16px, 2);
$ratio: math.div(16, 9);

.card {
  padding: $half-size;
}
~~~

Generated CSS:

~~~css
.card {
  padding: 8px;
}
~~~

The quotient in $ratio is approximately 1.7777777778. It is a unitless number, not the CSS aspect-ratio notation 16 / 9.

## 5. Compare division with a slash-separated list

~~~scss
@use "sass:list";
@use "sass:math";

$quotient: math.div(16, 9);
$css-ratio: list.slash(16, 9);

@debug $quotient;  // 1.7777777778
@debug $css-ratio; // 16 / 9

.video {
  aspect-ratio: $css-ratio;
}
~~~

Generated CSS:

~~~css
.video {
  aspect-ratio: 16 / 9;
}
~~~

Use math.div() when Sass should calculate a numeric result. Use list.slash() when the CSS output must preserve a slash-separated value.

## 6. Build a ratio from variables

~~~scss
@use "sass:list";

$video-width: 16;
$video-height: 9;
$video-ratio: list.slash($video-width, $video-height);

.video {
  aspect-ratio: $video-ratio;
}
~~~

Generated CSS:

~~~css
.video {
  aspect-ratio: 16 / 9;
}
~~~

list.slash() creates a list with slash separation. It does not calculate the ratio.

## 7. Sass simplifies known arithmetic inside calc()

~~~scss
@debug calc(24px / 3); // 8px
@debug calc(100% / 2);  // 50%
~~~

Both calculations can be simplified by Sass. The percentage value is still relative in CSS, but dividing 100% by the unitless number 2 is compile-time arithmetic; it becomes 50%.

For a rule:

~~~scss
.half-width {
  width: calc(100% / 2);
}
~~~

Generated CSS:

~~~css
.half-width {
  width: 50%;
}
~~~

## 8. The browser resolves a CSS custom property calculation

~~~scss
.box {
  width: calc(var(--available-width) / 3);
}
~~~

Generated CSS:

~~~css
.box {
  width: calc(var(--available-width) / 3);
}
~~~

Sass cannot know the value of the CSS custom property. The browser resolves the division after the custom property is substituted.

## 9. Sass variables can be passed directly into calc()

~~~scss
$sidebar-width: 18rem;
$gutter: 1rem;

.main {
  width: calc(100% - $sidebar-width - $gutter * 2);
}
~~~

Generated CSS:

~~~css
.main {
  width: calc(100% - 18rem - 2rem);
}
~~~

Sass calculates the known gutter arithmetic. The browser resolves the remaining percentage against layout.

## 10. Use spaces around plus and minus inside calculations

~~~scss
$extra-space: 1rem;

.card {
  width: calc(100% - $extra-space);
  padding: calc(0.5rem + $extra-space);
}
~~~

Generated CSS:

~~~css
.card {
  width: calc(100% - 1rem);
  padding: calc(0.5rem + 1rem);
}
~~~

In CSS calculations, include spaces around binary plus and minus operators. This is part of the CSS calculation syntax.

## 11. Sass can remove calc() when the result is fully known

~~~scss
$width: calc(300px + 20px);

.panel {
  width: $width;
}
~~~

Generated CSS:

~~~css
.panel {
  width: 320px;
}
~~~

Both values are known compatible lengths, so Sass simplifies the calculation to a plain number.

## 12. Keep viewport-dependent values for the browser

~~~scss
.heading {
  font-size: clamp(1rem, 4vw, 3rem);
}

.container {
  width: min(100%, 72rem);
}

.safe-area {
  padding-inline-start: max(1rem, env(safe-area-inset-left));
}
~~~

Generated CSS:

~~~css
.heading {
  font-size: clamp(1rem, 4vw, 3rem);
}

.container {
  width: min(100%, 72rem);
}

.safe-area {
  padding-inline-start: max(1rem, env(safe-area-inset-left));
}
~~~

The viewport unit, percentage, and environment variable require browser context, so Sass preserves these CSS functions.

## 13. Sass can simplify min() when all inputs are compatible

~~~scss
.fixed {
  width: min(300px, 24px);
}
~~~

Generated CSS:

~~~css
.fixed {
  width: 24px;
}
~~~

Both inputs are known compatible lengths. Sass can choose the smaller value at compile time. Do not substitute 24rem here: Sass cannot generally convert rem to px because the relationship depends on browser font sizing.

## 14. Nested calculations can be simplified

~~~scss
$fluid-width: calc(100% + 10px);

.sidebar {
  width: $fluid-width;
  padding-inline-start: calc($fluid-width / 4);
}
~~~

Generated CSS:

~~~css
.sidebar {
  width: calc(100% + 10px);
  padding-inline-start: calc((100% + 10px) / 4);
}
~~~

The inner calculation is reused, and Sass preserves the percentage relationship for the browser.

## 15. Avoid interpolation inside calc()

~~~scss
$gap: 1rem;

// Avoid this:
// .bad-example {
//   width: calc(100% - #{$gap});
// }

.good-example {
  width: calc(100% - $gap);
}
~~~

Generated CSS:

~~~css
.good-example {
  width: calc(100% - 1rem);
}
~~~

Direct Sass variables let Sass understand and simplify the calculation. Interpolation turns a value into text and prevents Sass from checking or simplifying operations that include that text.

## 16. Do not apply plain Sass arithmetic to a calculation

~~~scss
$fluid-width: calc(100% + 10px);

// This is an error because $fluid-width is a calculation, not a plain number:
// $invalid-width: $fluid-width * 2;

// Keep the operation inside calculation syntax:
$scaled-width: calc($fluid-width * 2);

.panel {
  width: $scaled-width;
}
~~~

Generated CSS:

~~~css
.panel {
  width: calc((100% + 10px) * 2);
}
~~~

A Sass calculation is not interchangeable with a plain numeric value. Keep math on a calculation inside calc() or another calculation function.

## 17. Use math.div() for a design token ratio

~~~scss
@use "sass:math";

$full-spacing: 2rem;
$half-spacing: math.div($full-spacing, 2);
$third-spacing: math.div($full-spacing, 3);

.stack {
  gap: $half-spacing;
}

.compact-stack {
  gap: $third-spacing;
}
~~~

Generated CSS:

~~~css
.stack {
  gap: 1rem;
}

.compact-stack {
  gap: 0.6666666667rem;
}
~~~

These are compile-time numeric calculations. No slash-separated list is involved.

## 18. Common output predictions

### What does this preserve?

~~~scss
@use "sass:list";

$ratio: list.slash(4, 3);

.media {
  aspect-ratio: $ratio;
}
~~~

It preserves 4 / 3 in the emitted CSS.

### What does this calculate?

~~~scss
@use "sass:math";

@debug math.div(4, 3);
~~~

It calculates approximately 1.3333333333.

### What can Sass simplify?

~~~scss
.fixed {
  width: calc(200px + 40px);
}
~~~

Sass can emit width: 240px because both operands are known compatible lengths.

### What must the browser resolve?

~~~scss
.flexible {
  width: calc(100% - var(--sidebar-width));
}
~~~

The browser resolves the percentage and custom property when it computes layout.

## Try it yourself

1. Replace math.div(16, 9) with list.slash(16, 9). How does the output differ?
2. Change calc(100% / 2) to calc(100% / 4). What percentage can Sass emit?
3. Replace var(--available-width) with a known Sass value. Which parts can Sass simplify?
4. Change min(300px, 24px) to min(300px, 240px). Which value should Sass emit?
5. Why does calc(100% - 2rem) remain a calculation?
6. Why should a Sass variable be passed directly into calc() rather than interpolated?

## References

- [Sass numeric operators and slash-separated values](https://sass-lang.com/documentation/operators/numeric/)
- [Sass calculations](https://sass-lang.com/documentation/values/calculations/)
- [Sass math module](https://sass-lang.com/documentation/modules/math/)
- [Sass list module](https://sass-lang.com/documentation/modules/list/)
