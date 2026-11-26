# 008 — Numbers, Units, and Arithmetic: Q&A

This Q&A reviews Sass number values, units, arithmetic, and the use of math.div() for numeric division.

## Number fundamentals

### 1. What are the parts of a Sass number?

A Sass number has a numeric value and units. For example, in 16px, 16 is the numeric value and px is its unit. A number can also be unitless, such as 16.

### 2. Are units just text attached to a number?

No. Units are part of the Sass number value. They affect which operations are valid, what the result means, and whether values can be converted.

### 3. Can Sass numbers have decimal results?

Yes. Sass keeps fractional results. For example, dividing 5 by 2 with math.div() produces 2.5, not 2.

~~~scss
@use "sass:math";

@debug math.div(5, 2); // 2.5
~~~

### 4. Name some common unit categories.

Examples include lengths such as px and rem, times such as s and ms, angles such as deg and turn, frequencies such as Hz, and resolutions such as dpi. Sass numbers may also be unitless.

## Unit compatibility and conversions

### 5. What does it mean for two units to be compatible?

Sass can convert one unit to the other using a known conversion relationship. For example, CSS defines 1in as 96px, so Sass can compare and calculate with those absolute length units.

~~~scss
@debug 1in == 96px; // true
~~~

### 6. Why can Sass not generally convert rem to px?

The pixel size of rem depends on the browser's root font size. Sass compiles before the browser resolves that context, so it cannot generally calculate the rem-to-px conversion.

### 7. What happens if I add a length and a duration?

Sass reports an incompatible-unit error. A value such as 1px + 1s has no meaningful numeric result.

### 8. Can unitless numbers be used with unit-bearing numbers?

Yes. Sass allows unitless numbers in certain operations with a dimensioned value. For example, 100px + 50 produces 150px. This can be convenient, but it can also hide a missing unit. Use a unitless value only when that is intentional.

### 9. What happens if I add 1rem and 4px?

Sass cannot generally convert rem to px because their relationship depends on runtime font sizing. Treat this as an incompatible-unit calculation. If the browser must resolve the relationship, use an appropriate CSS calculation.

## Arithmetic operators

### 10. Which arithmetic operators does Sass support for numbers?

Sass supports addition (+), subtraction (-), multiplication (*), and modulo (%). Unary plus and minus can mark a value positive or negative. Use math.div() for numeric division.

### 11. How does addition affect units?

Addition preserves a compatible dimension. Sass converts compatible units as needed. For example, adding two rem values produces a rem value.

~~~scss
$spacing: 1rem + 0.5rem;
// $spacing is 1.5rem
~~~

### 12. How does subtraction affect units?

Subtraction also requires compatible units and preserves the resulting dimension.

~~~scss
$remaining: 5rem - 2rem;
// $remaining is 3rem
~~~

### 13. What should I do to make subtraction easy to read?

Put spaces around the subtraction operator.

~~~scss
$remaining: 5rem - 2rem;
~~~

Keep a minus sign attached to a negative number:

~~~scss
$offset: -2rem;
~~~

This makes arithmetic clearer, especially in a space-separated list.

### 14. What happens when I multiply a length by a unitless number?

The unitless number scales the length while the length unit remains.

~~~scss
$double-gap: 0.75rem * 2;
// $double-gap is 1.5rem
~~~

### 15. What happens when I multiply two values that both have units?

Their units are multiplied too. Multiplying 3px by 4px produces a value with square-pixel units, not an ordinary length. Check that compound units make sense before using a result in a CSS property.

### 16. What does the modulo operator do?

The percent operator between Sass numbers returns a remainder.

~~~scss
@debug 10 % 3; // 1
@debug 10px % 3px; // 1px
~~~

The number 50% uses the percent unit; it is not the same thing as the modulo expression 10 % 3.

## Division and the math module

### 17. What is the current Sass way to divide numbers?

Load the built-in math module and call math.div().

~~~scss
@use "sass:math";

$half: math.div(1rem, 2);
~~~

### 18. Why should I avoid writing a slash for numeric division?

Slash is part of CSS syntax for slash-separated values, including font shorthand and modern color syntax. Sass's use of slash for division is deprecated. The next chapter covers slash-separated values and the distinction in more detail.

### 19. What unit does division by a unitless number produce?

The numerator's unit remains:

~~~scss
@use "sass:math";

@debug math.div(12px, 3); // 4px
~~~

### 20. What happens when compatible units are divided?

The units cancel and the result is unitless:

~~~scss
@use "sass:math";

@debug math.div(12px, 3px); // 4
~~~

### 21. What happens if I divide by zero?

Sass reports an error. A zero denominator cannot produce a valid finite numeric result.

### 22. Why are ratios often unitless?

A ratio compares quantities with compatible units. Those units cancel, leaving a proportion that can scale another compatible value.

~~~scss
@use "sass:math";

$ratio: math.div(300px, 1200px); // 0.25
$column-width: 1200px * $ratio;  // 300px
~~~

## Interpolation and CSS calculations

### 23. Why is interpolation a poor way to append a unit?

Interpolation turns the result into text. A string such as 12px may look like a number, but it no longer participates in Sass number arithmetic or unit checking.

~~~scss
$size: 12;

// Avoid creating a number-looking string:
// $width: #{$size}px;
~~~

If the input is intended to be pixels, keep it numeric:

~~~scss
$width: $size * 1px;
~~~

### 24. When should the browser perform the calculation?

Use a CSS calculation when the result depends on runtime layout or values known by the browser, such as viewport dimensions or contextual percentages. Sass can calculate values known at compile time; the browser resolves CSS calculations later.

### 25. Is a compound unit always valid for a CSS property?

No. Sass can represent compound units for calculations, but an ordinary CSS property may not accept such a value. Use a compound unit when the calculation calls for it, and cancel units where the intended result is a standard CSS dimension.

## Predict the result

### 26. What does this value equal?

~~~scss
$gap: 0.5rem * 3;
~~~

It equals 1.5rem. Multiplication by a unitless scalar preserves the rem unit.

### 27. What does this division return?

~~~scss
@use "sass:math";

@debug math.div(18px, 3);
~~~

It returns 6px. The denominator is unitless, so the numerator's px unit remains.

### 28. What does this division return?

~~~scss
@use "sass:math";

@debug math.div(18px, 3px);
~~~

It returns 6, a unitless number. The compatible px units cancel.

### 29. What happens here?

~~~scss
$invalid: 2s + 10px;
~~~

Sass reports an incompatible-unit error. Seconds and pixels measure different kinds of quantities.

### 30. What does the CSS become?

~~~scss
$gutter: 1rem;
$double-gutter: $gutter * 2;

.layout {
  gap: $double-gutter;
}
~~~

The CSS contains a gap of 2rem:

~~~css
.layout {
  gap: 2rem;
}
~~~

## Common mistakes

### 31. Why did Sass reject my addition?

The operands may have incompatible units. Check that they measure compatible quantities and that Sass knows how to convert them.

### 32. Why did a multiplication produce a strange unit?

Both operands may have had units, so Sass multiplied the units as well as the numeric values. Check the dimensions of both inputs.

### 33. Can I always remove a unit by dividing by a number?

No. Dividing a unit-bearing number by a unitless scalar preserves the unit. To cancel units, divide by a compatible unit-bearing value.

### 34. Should I use interpolation to add units?

No. Interpolation creates a string. Assign a unit to the value directly or multiply by a unit-bearing one.

### 35. How do I distinguish modulo from a percentage?

Modulo uses % between two numbers as an operator, as in 10 % 3. A percentage unit is attached to a number, as in 50%.

## Quick review

- Sass numbers contain a numeric value and units.
- Sass converts units only when it knows a compatible conversion.
- Addition, subtraction, and modulo require compatible units, except Sass can combine unitless numbers with a dimensioned value.
- Multiplication combines the operands' units.
- math.div() is the modern function for numeric division.
- Compatible units cancel in division.
- Interpolation creates text, not a numeric Sass value.
- Use CSS calculations when the browser must resolve runtime context.

## References

- [Sass numbers and units](https://sass-lang.com/documentation/values/numbers/)
- [Sass numeric operators](https://sass-lang.com/documentation/operators/numeric/)
- [Sass math module](https://sass-lang.com/documentation/modules/math/)
