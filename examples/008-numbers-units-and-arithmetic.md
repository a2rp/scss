# 008 — Numbers, Units, and Arithmetic: Examples

Each example uses SCSS and can be copied into its own .scss file. Sass evaluates arithmetic at compile time when the values and units allow it.

## 1. Add and subtract compatible lengths

~~~scss
$base-space: 1rem;
$extra-space: 0.5rem;
$large-space: $base-space + $extra-space;
$small-space: $base-space - $extra-space;

.card {
  padding: $large-space;
  margin-block-start: $small-space;
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1.5rem;
  margin-block-start: 0.5rem;
}
~~~

Addition and subtraction preserve the compatible length unit.

## 2. Scale a length with multiplication

~~~scss
$unit: 0.25rem;
$space-1: $unit * 1;
$space-2: $unit * 2;
$space-4: $unit * 4;

.stack {
  gap: $space-2;
  padding: $space-4;
}
~~~

Generated CSS:

~~~css
.stack {
  gap: 0.5rem;
  padding: 1rem;
}
~~~

Multiplying by a unitless scalar keeps the original unit.

## 3. Work with compatible absolute units

~~~scss
@debug 1in == 96px; // true
@debug 1in + 96px;  // Represents 2in; Sass may write it as 192px.
~~~

Sass knows the conversion between CSS absolute length units such as inches and pixels. Equivalent calculations may be represented using a different compatible unit by a different Sass implementation.

Use this kind of conversion only when Sass has a known conversion rule. Context-dependent units such as rem and em cannot generally be converted to px during Sass compilation.

## 4. Avoid incompatible-unit addition

This expression intentionally fails to compile:

~~~scss
// Seconds and pixels measure different things.
// @debug 1s + 1px;
~~~

Sass reports an incompatible-unit error. Do not add unlike quantities just because both are numbers.

This also cannot generally be resolved by Sass:

~~~scss
// Sass does not know the runtime relationship between rem and px.
// @debug 1rem + 4px;
~~~

When a relationship depends on browser context, use a CSS calculation instead, as shown later.

## 5. Understand unitless arithmetic

Sass allows a unitless number to combine with a unit-bearing number in some operations:

~~~scss
$base-width: 100px;
$extra-count: 50;
$combined: $base-width + $extra-count;

.box {
  width: $combined;
}
~~~

Generated CSS:

~~~css
.box {
  width: 150px;
}
~~~

This behavior can be useful when the unitless value intentionally represents a count in the calculation. If 50 is supposed to represent a physical length, add the correct unit instead of relying on this behavior.

## 6. Calculate a remainder with modulo

~~~scss
$remainder: 10px % 3px;

.badge {
  margin-inline-start: $remainder;
}
~~~

Generated CSS:

~~~css
.badge {
  margin-inline-start: 1px;
}
~~~

The modulo operator returns the remainder. It is different from writing a percentage unit such as 50%.

~~~scss
@debug 10 % 3;     // 1
@debug 10px % 3px; // 1px
~~~

## 7. Divide numbers with math.div()

~~~scss
@use "sass:math";

$half-gap: math.div(1rem, 2);
$third-width: math.div(100%, 3);
$ratio: math.div(12, 4);

.columns {
  gap: $half-gap;
  width: $third-width;
}
~~~

Generated CSS:

~~~css
.columns {
  gap: 0.5rem;
  width: 33.3333333333%;
}
~~~

The results are:

- A length divided by a unitless scalar remains a length.
- A percentage divided by a unitless scalar remains a percentage.
- A unitless number divided by a unitless number remains unitless.

Use math.div() for numeric division. A slash has CSS meaning and is covered in the next chapter.

## 8. Cancel compatible units to make a ratio

~~~scss
@use "sass:math";

$container-width: 1200px;
$column-width: 300px;
$column-ratio: math.div($column-width, $container-width);
$computed-width: $container-width * $column-ratio;

.column {
  width: $computed-width;
}
~~~

Generated CSS:

~~~css
.column {
  width: 300px;
}
~~~

The two compatible length units cancel in the ratio, leaving 0.25. Multiplying the container width by that unitless ratio produces a length again.

## 9. Divide compatible units to get a unitless result

~~~scss
@use "sass:math";

@debug math.div(36px, 12px); // 3
@debug math.div(36px, 3);    // 12px
~~~

In the first calculation, px cancels with px. In the second, the denominator is unitless, so the result keeps px.

## 10. Use a negative value and subtraction clearly

~~~scss
$container-padding: 2rem;
$icon-overlap: -0.25rem;
$content-space: $container-padding - 0.5rem;

.card {
  padding: $content-space;
}

.card__icon {
  margin-inline-end: $icon-overlap;
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1.5rem;
}

.card__icon {
  margin-inline-end: -0.25rem;
}
~~~

Spaces around subtraction make the operator clear. Keep a negative sign next to its number.

## 11. Put a negative value inside a space-separated list

~~~scss
$offsets: 1rem (-0.5rem) 2rem;

.example {
  margin: $offsets;
}
~~~

Generated CSS:

~~~css
.example {
  margin: 1rem -0.5rem 2rem;
}
~~~

Parentheses make it clear that -0.5rem is one negative list item rather than a subtraction expression.

## 12. Keep a number numeric instead of interpolating a unit

~~~scss
$size: 12;
$looks-like-a-number: "#{$size}px";
$real-length: $size * 1px;

@debug $looks-like-a-number; // "12px"
@debug $real-length;         // 12px
~~~

The first value is text. The second is a Sass number with a px unit and can participate in arithmetic:

~~~scss
$double-length: $real-length * 2;

.box {
  width: $double-length;
}
~~~

Generated CSS:

~~~css
.box {
  width: 24px;
}
~~~

If an input is intended to be a length, keep it numeric by supplying its unit or multiplying by a unit-bearing value.

## 13. Let the browser resolve runtime layout

~~~scss
$inline-gutter: 1rem;
$total-gutter: $inline-gutter * 2;

.content {
  width: calc(100% - #{$total-gutter});
}
~~~

Generated CSS:

~~~css
.content {
  width: calc(100% - 2rem);
}
~~~

The browser knows the final containing-block width at layout time. Sass can insert a known Sass value, while CSS calc() preserves the runtime relationship for the browser.

## 14. Combine arithmetic in a small component

~~~scss
@use "sass:math";

$base-size: 1rem;
$control-padding: $base-size * 0.75;
$control-gap: math.div($base-size, 2);
$border-width: 1px;
$focus-width: $border-width * 2;

.control {
  padding: $control-padding $base-size;
  gap: $control-gap;
  border: $border-width solid #87909e;

  &:focus-visible {
    outline: $focus-width solid #2457a7;
  }
}
~~~

Generated CSS:

~~~css
.control {
  padding: 0.75rem 1rem;
  gap: 0.5rem;
  border: 1px solid #87909e;
}

.control:focus-visible {
  outline: 2px solid #2457a7;
}
~~~

Each calculation keeps meaningful units, and Sass emits the evaluated dimensions.

## 15. Practice exercises

Try these before reading the answers:

1. What is 0.5rem multiplied by 6?
2. What does math.div(24px, 6) return?
3. What does math.div(24px, 6px) return?
4. What is 11px modulo 4px?
5. Why does Sass reject 2s + 8px?
6. Why should you not build 16px by interpolating 16 and px?
7. Which part must be resolved by the browser in calc(100% - 2rem)?

### Answers

1. 3rem.
2. 4px.
3. 4, a unitless number.
4. 3px.
5. Seconds and pixels are incompatible units.
6. Interpolation creates a string, not a numeric Sass value.
7. The browser resolves the percentage against the runtime layout width.

## References

- [Sass numbers and units](https://sass-lang.com/documentation/values/numbers/)
- [Sass numeric operators](https://sass-lang.com/documentation/operators/numeric/)
- [Sass math module](https://sass-lang.com/documentation/modules/math/)
