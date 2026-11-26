# 009 - Slash Division and Calculations

## Learning goals

After this lesson, you should be able to:

- Distinguish a slash-separated CSS value from numeric division.
- Use math.div() whenever Sass should divide numbers.
- Build a slash-separated list explicitly with sass:list when needed.
- Explain how calc(), min(), max(), and clamp() work as Sass calculations.
- Understand when Sass simplifies a calculation and when the browser must resolve it.
- Avoid calculation syntax that prevents Sass from checking or simplifying values.

## 1. Why slash has two meanings

The slash character is used in CSS as a separator. Examples include:

~~~scss
font: 16px/1.5 "Inter", sans-serif;
aspect-ratio: 16 / 9;
border-radius: 24px / 12px;
~~~

In the font shorthand, 16px/1.5 means a font size followed by a line-height. For aspect-ratio, 16 / 9 describes the ratio of width to height. For border-radius, the slash separates horizontal radii from vertical radii.

Programming languages often use slash for division too. Sass historically did this, which made expressions ambiguous: should 16px / 2 be calculated, or should the slash be preserved for CSS output?

Use different syntax for each intent:

- Use math.div() for Sass numeric division.
- Use a slash-separated value when CSS syntax needs a slash separator.
- Use slash inside calc() when writing a CSS calculation; inside a calculation, slash always means division.

## 2. Use math.div() for numeric division

Load Sass's math module and call math.div():

~~~scss
@use "sass:math";

$half: math.div(16px, 2);
$ratio: math.div(16, 9);

@debug $half;  // 8px
@debug $ratio; // 1.7777777778
~~~

The unit behavior follows number arithmetic:

- Dividing a length by a unitless number preserves the length unit.
- Dividing two compatible lengths cancels the units and returns a unitless number.
- Dividing by zero is an error.

~~~scss
@use "sass:math";

@debug math.div(24px, 3);     // 8px
@debug math.div(24px, 6px);  // 4
~~~

When Sass interprets a slash in ordinary SassScript as arithmetic, that legacy division behavior is deprecated. Its context-dependent parsing rules are easy to misread, so do not depend on them. math.div() states the intent clearly and avoids conflicts with CSS slash syntax.

## 3. Slash-separated values are not quotients

A slash-separated list preserves the slash in its output. It does not calculate a ratio.

~~~scss
@debug 16px / 2; // 16px/2: a slash-separated value
~~~

The exact interpretation of slash syntax in ordinary SassScript has historically depended on the surrounding expression. That legacy ambiguity is why new code should use math.div() for division.

Use slash-separated values when the target CSS property expects that syntax. Common examples include aspect ratios, font shorthand, and elliptical border radii.

~~~scss
.hero {
  aspect-ratio: 16 / 9;
  border-radius: 24px / 12px;
}
~~~

Generated CSS:

~~~css
.hero {
  aspect-ratio: 16 / 9;
  border-radius: 24px / 12px;
}
~~~

Here the slash is part of the CSS value. Sass is preserving a separator, not dividing 16 by 9 or 24px by 12px.

## 4. Construct a slash list explicitly

When a slash-separated list is assembled from Sass variables or expressions, use sass:list:

~~~scss
@use "sass:list";

$width: 16;
$height: 9;
$ratio-value: list.slash($width, $height);

.video {
  aspect-ratio: $ratio-value;
}
~~~

Generated CSS:

~~~css
.video {
  aspect-ratio: 16 / 9;
}
~~~

list.slash() is the current explicit helper for constructing a slash-separated list. It avoids treating the slash as division. Sass documents it as a temporary solution while slash lists are not written with ordinary literal syntax in SassScript.

Do not use list.slash() to perform arithmetic. To calculate a quotient, use math.div().

## 5. CSS calculations are a separate expression system

Sass supports CSS calculation functions such as calc(), min(), max(), and clamp(). These represent calculations that CSS can preserve for the browser.

~~~scss
$gutter: 1rem;

.content {
  width: calc(100% - $gutter * 2);
}
~~~

Generated CSS:

~~~css
.content {
  width: calc(100% - 2rem);
}
~~~

Sass knows the value of $gutter, so it can simplify the part it can calculate. It cannot resolve 100% at build time because the percentage depends on the browser's layout context. The browser finishes that calculation later.

Other examples can remain as CSS calculations:

~~~scss
.card {
  width: min(100%, 72rem);
  padding-inline: clamp(1rem, 3vw, 2.5rem);
}
~~~

These values depend on browser layout and viewport dimensions, so Sass leaves the calculation for CSS.

## 6. Sass simplifies calculations when it can

Sass evaluates known operations inside calculations:

~~~scss
@debug calc(400px / 2); // 200px
@debug calc(5s * 2);     // 10s
~~~

When a calculation mixes values that cannot be reduced to one compile-time number, Sass keeps a CSS calculation:

~~~scss
@debug calc(400px + 10%); // calc(400px + 10%)
~~~

The browser knows the containing block size used by the percentage. Sass does not, so the full result remains for runtime.

A calculation does not always remain wrapped in calc(). If Sass can produce a plain number, it may return that value directly.

~~~scss
$fixed-width: calc(300px + 20px);

.panel {
  width: $fixed-width;
}
~~~

Generated CSS:

~~~css
.panel {
  width: 320px;
}
~~~

The compiler simplified the calculation because both operands were known compatible lengths.

## 7. Slash inside calc() always means division

Inside a CSS calculation, slash is always the division operator:

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

The browser supplies the custom property value at runtime, so Sass preserves the division calculation. By contrast, Sass can simplify calc(100% / 3) to 33.3333333333% because dividing a known percentage by a unitless number is compile-time arithmetic. Do not confuse slash inside a calculation with a slash-separated list in ordinary CSS value syntax.

For ordinary SassScript numeric division, prefer math.div(). Inside calc(), use CSS calculation syntax.

## 8. Keep calculation syntax visible to Sass

Sass calculations support Sass variables and Sass functions. Pass a variable directly into the calculation:

~~~scss
$base-width: 400px;

.sidebar {
  width: calc($base-width + 10%);
}
~~~

This allows Sass to recognize and simplify known parts of the expression.

Avoid interpolation inside calculations:

~~~scss
$base-width: 400px;

// Avoid interpolating the value inside calc():
// width: calc(#{$base-width} + 10%);
~~~

Interpolation turns the value into text. Sass cannot type-check or simplify operations that involve that interpolated text, which can lead to verbose or invalid CSS. Prefer calc(10px + $value) over calc(10px + #{$value}).

## 9. Calculation values are not ordinary Sass numbers

A value such as calc(100% + 10px) is a Sass calculation value, not a regular numeric Sass value. It can be used in compatible CSS calculations, but it cannot be passed to every Sass number operation.

~~~scss
$fluid-width: calc(100% + 10px);

// This is an error: a calculation is not a regular number.
// $doubled-width: $fluid-width * 2;

// Put the operation inside a calculation instead:
$scaled-fluid-width: calc($fluid-width * 2);
~~~

Sass can combine nested calculations and simplify known parts while preserving the browser-dependent parts.

## 10. Calculation functions: min(), max(), and clamp()

These CSS functions choose or constrain values:

- min() returns the smallest argument.
- max() returns the largest argument.
- clamp() limits a preferred value between a minimum and maximum.

~~~scss
$minimum: 16px;

.heading {
  font-size: clamp($minimum, 4vw, 48px);
}

.container {
  width: min(100% - 2rem, 72rem);
}

.safe-space {
  padding-left: max(1rem, env(safe-area-inset-left));
}
~~~

Generated CSS:

~~~css
.heading {
  font-size: clamp(16px, 4vw, 48px);
}

.container {
  width: min(100% - 2rem, 72rem);
}

.safe-space {
  padding-left: max(1rem, env(safe-area-inset-left));
}
~~~

The browser resolves the functions because viewport units, percentages, and environment variables depend on browser context.

When every argument can be calculated at build time, Sass may simplify a function to one value:

~~~scss
.fixed {
  width: min(300px, 24px);
}
~~~

Because both values are compatible lengths, Sass can choose the smaller value during compilation. The exact printed unit can depend on how Sass represents the result.

## 11. Keep Sass functions and CSS calculations distinct

Sass has a math module for compile-time numeric work. CSS calculations describe expressions that may need to remain for the browser.

Use math.div() when a Sass number should become a quotient:

~~~scss
@use "sass:math";

$third: math.div(100, 3);
~~~

Use calc() when the browser must combine values at runtime:

~~~scss
$sidebar: 18rem;

.main {
  width: calc(100% - $sidebar);
}
~~~

Use min(), max(), or clamp() when the browser should choose or constrain responsive values. Sass may simplify the calculation when the operands are fully known, but browser-dependent pieces remain in the emitted CSS.

## 12. Common mistakes

### Using slash for Sass arithmetic

The old Sass division behavior is deprecated and ambiguous. Use math.div() for numeric division.

### Assuming every slash means division

In CSS, slash often separates values. A slash-separated value should remain a slash-separated value.

### Using a slash list when you need a quotient

A slash list preserves its values and separator. Use math.div() when you need a numeric result.

### Interpolating variables inside calc()

Interpolation converts a value to text and prevents Sass from type-checking or simplifying the operation. Pass Sass variables directly into calculations.

### Applying ordinary arithmetic to a calculation value

A calc() expression is not interchangeable with a plain Sass number. Keep operations on calculations inside calc() or another calculation function.

### Expecting Sass to know browser layout

Percentages, viewport units, CSS variables, and environment values may need browser context. Sass can simplify known pieces, but the browser resolves the rest.

## 13. Worked example: responsive content width

Prefer direct Sass variables without interpolation in calculations:

~~~scss
$sidebar-width: 18rem;
$minimum-gutter: 1rem;
$maximum-content: 72rem;

.page-content {
  width: min(calc(100% - $sidebar-width), $maximum-content);
  margin-inline: auto;
  padding-inline: clamp($minimum-gutter, 3vw, 2rem);
}
~~~

Generated CSS:

~~~css
.page-content {
  width: min(100% - 18rem, 72rem);
  margin-inline: auto;
  padding-inline: clamp(1rem, 3vw, 2rem);
}
~~~

Sass substitutes known Sass variables and simplifies compatible calculations. The browser resolves the remaining percentage, viewport, and min/clamp behavior against the rendered page.

## 14. Practice checklist

Before moving on, make sure you can answer these questions:

1. When should you use math.div()?
2. What does a slash-separated list do?
3. How can you build a slash-separated list from variables?
4. What does slash mean inside calc()?
5. Which parts of calc(100% - 2rem) can Sass calculate?
6. Why should interpolation be avoided inside calculations?
7. Can a calc() value be multiplied using ordinary SassScript outside calc()?
8. Why might Sass simplify min() or clamp() before emitting CSS?
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="008-numbers-units-and-arithmetic.md">008 - Numbers, Units, and Arithmetic</a></td><td><a href="../qnas/009-slash-division-and-calculations.md">Q&amp;A</a></td><td><a href="../examples/009-slash-division-and-calculations.md">Example</a></td><td><a href="010-strings-and-interpolation.md">010 - Strings and Interpolation</a></td></tr>
  </tbody>
</table>
