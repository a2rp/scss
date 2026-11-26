# 008 — Numbers, Units, and Arithmetic

## Learning goals

After this lesson, you should be able to:

- Read a Sass number as a numeric value plus its unit.
- Use addition, subtraction, multiplication, modulo, and unary signs.
- Explain how Sass converts compatible units and rejects incompatible ones.
- Use unitless values deliberately instead of accidentally losing a unit.
- Use the modern math.div() function for numeric division.
- Keep number calculations numeric instead of turning them into strings with interpolation.

## 1. A Sass number has a value and units

A Sass number has a numeric part and zero or more units:

~~~scss
$items: 4;
$gap: 1.25rem;
$turn: 90deg;
$duration: 250ms;
~~~

In 1.25rem, the number is 1.25 and the unit is rem. Units are part of the value; they are not decorations that Sass ignores.

A number can be unitless, such as 4, or carry one unit, such as 1.25rem. Sass can also represent compound units during calculations, such as pixels per second. Compound units are useful for Sass math, but ordinary CSS properties generally expect units suited to that property.

Sass number values can be decimals. Sass does not round a numeric result to a whole number just because the source values were whole numbers.

~~~scss
@use "sass:math";

@debug math.div(5, 2); // 2.5
~~~

## 2. Units describe what a number measures

Common CSS units include:

| Category | Examples | Typical meaning |
| --- | --- | --- |
| Length | px, rem, em, %, vw, vh, cm | Size, spacing, or position |
| Time | s, ms | Duration or delay |
| Angle | deg, rad, turn | Rotation or direction |
| Frequency | Hz, kHz | Frequency values |
| Resolution | dpi, dpcm, dppx | Output resolution |
| Unitless | 0, 1.5, 400 | Ratios, counts, weights, and scalars |

The category alone does not guarantee Sass can convert two units. Sass needs a known conversion relationship. For example, px and in are compatible CSS absolute lengths. Sass knows that one inch is 96 pixels.

~~~scss
@debug 1in == 96px; // true
~~~

By contrast, rem depends on the root font size and em depends on the relevant font size. Sass cannot know their actual pixel sizes at compile time, so it cannot generally convert rem to px or em to rem.

~~~scss
// This is an incompatible-unit calculation:
// @debug 1rem + 4px;
~~~

Do not combine different units just because they are both used for lengths. Choose units that Sass can compare or convert, or use a CSS calculation when the browser must resolve context-dependent values.

## 3. Addition and subtraction

Addition and subtraction combine values that have compatible units:

~~~scss
$base-gap: 1rem;
$extra-gap: 0.5rem;

$large-gap: $base-gap + $extra-gap;
$compact-gap: $base-gap - $extra-gap;

.card {
  padding: $large-gap;
  margin-top: $compact-gap;
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1.5rem;
  margin-top: 0.5rem;
}
~~~

Sass can convert compatible units before adding or subtracting them. For example, 1in and 96px represent equal lengths. The unit Sass uses to display a result can depend on the Sass implementation and operand order, so focus on the represented quantity rather than assuming one preferred spelling.

Incompatible units produce an error for addition and subtraction. Adding a duration to a length has no meaningful result:

~~~scss
// Error: seconds and pixels are incompatible.
// $invalid: 1s + 1px;
~~~

### Unitless values in addition and subtraction

Sass allows a unitless number to be combined with a number that has units. The result carries the dimensioned operand's unit:

~~~scss
$width: 100px + 50; // 150px
~~~

This can be useful when the unitless value is intentionally a count in that calculation, but it can also hide a missing unit. If 50 is meant to be a length, write the length unit explicitly. Consistent units make mistakes easier for Sass to catch.

## 4. Multiplication

Multiplication combines numeric values and multiplies their units:

~~~scss
$gap: 0.75rem;
$double-gap: $gap * 2;

.panel {
  padding: $double-gap;
}
~~~

Generated CSS:

~~~css
.panel {
  padding: 1.5rem;
}
~~~

A unitless multiplier scales the number while preserving its unit. Multiplying two unit-bearing values combines their units:

~~~scss
@debug 3 * 4px; // 12px
@debug 3px * 4px; // 12px*px
~~~

The first result is a normal length. The second has square-pixel units. That compound value is not a normal border width or padding value. When a result has an unexpected compound unit, check whether the calculation is dimensionally correct.

## 5. Modulo and remainder

The percent operator between numbers performs modulo: it returns the remainder after division.

~~~scss
@debug 10 % 3; // 1
@debug 10px % 3px; // 1px
~~~

The units must be compatible. Do not confuse modulo with a percentage unit. The number 50% is a percentage-valued number; the expression 10 % 3 is a remainder calculation.

A design-system use might be checking whether a count falls on a repeating interval:

~~~scss
@use "sass:math";

@debug math.div(12, 4); // 3
@debug 12 % 4; // 0
~~~

Modulo is less common in everyday styling than addition or multiplication, but it can be useful in generated patterns and loops.

## 6. Modern numeric division with math.div()

In current Sass, use math.div() when you mean numeric division:

~~~scss
@use "sass:math";

$half-gap: math.div(1rem, 2);
$column-width: math.div(100%, 3);
$ratio: math.div(12, 4);

.layout {
  gap: $half-gap;
  width: $column-width;
}
~~~

The unit on the result follows the unit calculation:

- Dividing a length by a unitless number keeps the length unit.
- Dividing compatible units cancels them and produces a unitless result.
- Dividing by zero is an error.

~~~scss
@use "sass:math";

@debug math.div(12px, 3); // 4px
@debug math.div(12px, 3px); // 4
@debug math.div(5, 2); // 2.5
~~~

The old expression 12px / 3 is ambiguous because CSS uses slash-separated values, such as line-height in font shorthand. Sass's slash-division behavior is deprecated. Use math.div() for math; the next chapter focuses on slash-separated values and the distinction between Sass division and CSS slash syntax.

## 7. Unit cancellation and ratios

Division can cancel compatible units. A ratio of two lengths is unitless:

~~~scss
@use "sass:math";

$container: 1200px;
$column: 300px;
$column-ratio: math.div($column, $container);

@debug $column-ratio; // 0.25
~~~

A unitless ratio can then scale a dimension:

~~~scss
$container: 1200px;
$column-ratio: 0.25;
$column-width: $container * $column-ratio;

.grid-column {
  width: $column-width;
}
~~~

Generated CSS:

~~~css
.grid-column {
  width: 300px;
}
~~~

A ratio should be unitless when it represents a proportion. Do not remove units with string manipulation; let Sass's number arithmetic cancel compatible units.

## 8. Positive and negative numbers

A leading plus or minus can be a unary operator:

~~~scss
$offset: -0.5rem;
$positive-offset: +$offset;

.badge {
  margin-left: $offset;
}
~~~

Generated CSS:

~~~css
.badge {
  margin-left: -0.5rem;
}
~~~

For subtraction, put spaces around the minus sign. For a negative number, keep the minus next to the number:

~~~scss
$remaining: 5rem - 2rem;
$negative: -2rem;
~~~

This spacing makes the expression easier to read and prevents confusion between subtraction and a negative value, especially in space-separated lists.

~~~scss
$values: 1rem (-2rem) 3rem;
~~~

Parentheses make it clear that -2rem is one negative list item.

## 9. Keep numeric values numeric

Avoid creating units by interpolating a number into text:

~~~scss
$size: 12;

// Avoid this pattern:
// $width: #{$size}px;
~~~

Interpolation produces a string that looks like a number with a unit. A string cannot participate in Sass number operations or be checked for unit compatibility.

Prefer assigning a real number with units:

~~~scss
$width: 12px;
~~~

If a unitless input intentionally represents a count of pixels, multiply it by a unit-bearing one:

~~~scss
$size: 12;
$width: $size * 1px;
~~~

The result is a numeric Sass value with a px unit, so it can continue to participate in arithmetic.

## 10. Sass math versus CSS calculations

Sass arithmetic runs during compilation when Sass has enough information to calculate the result. Some values depend on the browser's layout or runtime environment, such as a percentage combined with a viewport-dependent value. Sass cannot always reduce those to a single number.

CSS has calculation functions such as calc() for values the browser resolves. Sass's arithmetic and CSS calculations are related, but they are not identical:

- Sass number arithmetic works with values known to the compiler.
- CSS calculations can preserve relationships for the browser to resolve later.
- Use a CSS calculation when the result depends on runtime layout or browser-known values.

Dedicated CSS math functions are covered later in this roadmap.

## 11. Common mistakes

### Adding incompatible units

A length and a duration cannot be added. Keep units meaningful and compatible.

### Assuming rem and px are interchangeable at compile time

Their relationship depends on browser font sizing. Sass cannot generally convert between them.

### Accidentally creating a compound unit

Multiplying two values that both have units multiplies their units too. Check whether the result makes sense for the CSS property.

### Using unitless values without checking their meaning

Sass allows some operations between unitless and unit-bearing values. That convenience can hide a missing unit, so use unitless inputs only when they represent a scalar or count intentionally.

### Writing slash when numeric division is intended

Slash has CSS meaning and Sass's slash division is deprecated. Use math.div() for numeric division. The next lesson explains slash behavior in detail.

### Interpolating a unit onto a number

Interpolation creates a string. Keep the value numeric by assigning a unit or multiplying by a unit-bearing value.

## 12. Worked example: fluid spacing tokens

~~~scss
@use "sass:math";

$space-unit: 0.25rem;
$space-1: $space-unit * 1;
$space-2: $space-unit * 2;
$space-4: $space-unit * 4;
$space-half: math.div($space-unit, 2);

.card {
  padding: $space-4;
  gap: $space-2;
}

.card__icon {
  margin-inline-end: $space-half;
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1rem;
  gap: 0.5rem;
}

.card__icon {
  margin-inline-end: 0.125rem;
}
~~~

The values stay numeric throughout. Sass scales the base unit, divides it with math.div(), and emits ordinary CSS lengths.

## 13. Practice checklist

Before moving on, make sure you can answer these questions:

1. What are the two main parts of a Sass number?
2. Why can Sass convert px to in but not generally rem to px?
3. What unit does multiplying a length by a unitless scalar preserve?
4. What kind of result can multiplying two unit-bearing values create?
5. How do you divide numbers using modern Sass?
6. What does dividing compatible units produce?
7. Why should unit interpolation be avoided?
8. When should the browser resolve a CSS calculation instead of Sass calculating a value?

## Further reading

- [Sass numbers and units](https://sass-lang.com/documentation/values/numbers/)
- [Sass numeric operators](https://sass-lang.com/documentation/operators/numeric/)
- [Sass math module](https://sass-lang.com/documentation/modules/math/)
