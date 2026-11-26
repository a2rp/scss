# 009 - Slash Division and Calculations: Q&A

This Q&A explains when slash is a CSS separator, when it represents numeric division, and how Sass calculations are simplified or passed through to the browser.

## Slash: separator or division?

### 1. Why does slash have more than one meaning in Sass?

CSS already uses slash inside values such as font shorthand, aspect-ratio, and elliptical border-radius. Sass also historically used slash for numeric division. Sass must distinguish preserving a CSS separator from calculating a quotient.

### 2. What is the recommended syntax for Sass numeric division?

Use math.div() from the built-in math module.

~~~scss
@use "sass:math";

$half: math.div(16px, 2);
~~~

This makes it explicit that a numeric quotient is wanted.

### 3. Is slash division in ordinary SassScript still recommended?

No. Sass's legacy slash-division behavior is deprecated and context-dependent. Use math.div() for Sass numeric division and reserve slash-separated values for CSS syntax.

### 4. What does math.div(16px, 2) return?

It returns 8px. The denominator is unitless, so the result keeps the numerator's px unit.

### 5. What does math.div(16, 9) return?

It returns a unitless quotient of approximately 1.7777777778. It does not preserve a slash.

### 6. Does a slash-separated list calculate a quotient?

No. It keeps both values with a slash separator.

~~~scss
@use "sass:list";

$ratio: list.slash(16, 9);
~~~

The value is emitted as 16 / 9 rather than being evaluated to approximately 1.78.

### 7. How do I build a slash-separated list from Sass variables?

Load sass:list and call list.slash():

~~~scss
@use "sass:list";

$width: 16;
$height: 9;

.video {
  aspect-ratio: list.slash($width, $height);
}
~~~

### 8. Is list.slash() the same as math.div()?

No.

- list.slash() creates a slash-separated list for CSS output.
- math.div() calculates a quotient as a Sass number.

Pick the function based on the desired result.

### 9. What does the slash mean in font shorthand?

It separates the font size from the line-height. For example, 16px/1.5 means a 16px font size and a 1.5 line-height. It is CSS syntax, not a request to divide 16px by 1.5.

### 10. What does the slash mean in aspect-ratio: 16 / 9?

It separates the width ratio from the height ratio. CSS interprets the two values as an aspect ratio. Sass should preserve that slash-separated value.

### 11. What does the slash mean in border-radius: 24px / 12px?

It separates horizontal radii from vertical radii for elliptical corners. It is a CSS separator.

## CSS calculations

### 12. What is a Sass calculation?

A calculation is a Sass value that represents CSS math syntax such as calc(), min(), max(), and clamp(). It can preserve browser-dependent math and can also include Sass variables and Sass functions.

### 13. What happens to calc(24px / 3)?

Sass can calculate the known values, so it simplifies the expression to 8px.

~~~scss
@debug calc(24px / 3); // 8px
~~~

### 14. What happens to calc(100% - 2rem)?

Sass generally preserves this as a CSS calculation because the percentage depends on the browser's layout context. The browser resolves the final result when it lays out the page.

### 15. What does slash always mean inside calc()?

Inside a CSS calculation, slash always means division. This is different from an ordinary slash-separated CSS value.

### 16. Can Sass simplify parts of a calculation?

Yes. Sass can simplify operations with values known at compile time and leave unresolved pieces for the browser.

~~~scss
$gutter: 1rem;

.content {
  width: calc(100% - $gutter * 2);
}
~~~

Sass can reduce the known gutter math to 2rem, while the browser resolves 100%.

### 17. Does calc() always remain in the emitted CSS?

No. If Sass can reduce the full calculation to a plain number, it may emit only that result.

~~~scss
$width: calc(300px + 20px);

.panel {
  width: $width;
}
~~~

The width can compile to 320px.

### 18. When might min(), max(), or clamp() remain in CSS?

They remain when their result depends on browser context, such as percentages, viewport units, or environment values. If all values are known and compatible, Sass may simplify the function during compilation.

### 19. Why should I pass Sass variables directly into calculations?

Sass can recognize their types, simplify known operations, and check units. For example, write calc(10px + $gap) rather than inserting the variable through interpolation.

### 20. Why is interpolation inside calc() discouraged?

Interpolation converts the value to text. Sass can no longer type-check or simplify operations involving that interpolated text, and the result may be verbose or invalid CSS.

### 21. Can I multiply a calculation value outside calc()?

Not with ordinary SassScript arithmetic. A calculation is not a regular Sass number.

~~~scss
$fluid-width: calc(100% + 10px);

// This is invalid Sass arithmetic:
// $doubled: $fluid-width * 2;

// Keep the operation in calculation syntax:
$doubled: calc($fluid-width * 2);
~~~

### 22. Can a calculation be used wherever a Sass number can?

No. Calculations are accepted in CSS calculation contexts, but they cannot be passed to every Sass math function or used where Sass requires a plain number.

## Predict the result

### 23. What does this produce?

~~~scss
@use "sass:math";

@debug math.div(18px, 3);
~~~

It produces 6px.

### 24. What does this produce?

~~~scss
@use "sass:list";

@debug list.slash(18px, 3);
~~~

It produces a slash-separated value: 18px / 3. It does not divide the values.

### 25. What happens to this declaration?

~~~scss
.box {
  width: calc(600px / 2);
}
~~~

Sass can simplify it to width: 300px.

### 26. What happens to this declaration?

~~~scss
.box {
  width: calc(var(--available-width) / 2);
}
~~~

The calculation remains in CSS because the browser supplies the custom property value at runtime.

### 27. Which value should be used for an aspect ratio?

Use a slash-separated value such as 16 / 9. If assembling it from Sass variables, use list.slash(). Do not use math.div(), because that returns a decimal quotient rather than the ratio notation CSS expects.

## Common mistakes and troubleshooting

### 28. My slash expression unexpectedly stayed as a slash. Why?

Sass preserves slash-separated values in CSS contexts and may preserve a slash in ordinary SassScript depending on the expression. Since legacy division is ambiguous and deprecated, use math.div() for arithmetic.

### 29. My slash expression unexpectedly became a decimal. How do I prevent that?

If the intended output is a slash-separated value and you are constructing it from Sass values, build it explicitly with list.slash().

### 30. Why did Sass remove calc() from my output?

Sass simplified the entire calculation to a plain number because all of its inputs were known and compatible at compile time. This is expected.

### 31. Why is my min() or clamp() still in the generated CSS?

The function may depend on browser context, such as a viewport width, percentage, or environment variable. The browser must resolve it after compilation.

### 32. Why did my calculation fail after interpolation?

Interpolation may have converted a numeric Sass value into text. Remove the interpolation and pass the Sass value directly into the calculation.

## Quick review

- Use math.div() for numeric Sass division.
- Use list.slash() to construct a slash-separated list.
- A slash in CSS properties may be a separator, not division.
- A slash inside calc() is always division.
- Sass simplifies calculation parts it can evaluate at compile time.
- The browser resolves layout-dependent calculation parts.
- Avoid interpolation inside calculations.
- Calculation values are not plain Sass numbers for ordinary arithmetic.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="008-numbers-units-and-arithmetic.md">008 - Numbers, Units, and Arithmetic: Q&amp;A</a></td><td><a href="../lessons/009-slash-division-and-calculations.md">Lesson</a></td><td><a href="../examples/009-slash-division-and-calculations.md">Example</a></td><td><a href="010-strings-and-interpolation.md">010 - Strings and Interpolation: Q&amp;A</a></td></tr>
  </tbody>
</table>
