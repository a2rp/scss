# 007 — Sass Value Types and Null: Q&A

This Q&A reviews the value types introduced in the lesson, with extra focus on Sass truthiness and how null affects generated CSS.

## Core concepts

### 1. What is a Sass value?

A Sass value is the result of an expression. Variables store values, and Sass passes values through calculations, functions, control rules, and declarations. During compilation, Sass evaluates them and emits CSS where appropriate.

### 2. What value types does Sass support?

The main types are numbers, strings, colors, lists, booleans, null, maps, and function references. Some correspond directly to CSS values. Maps and function references are primarily Sass-side structures.

### 3. Which value types map most directly to CSS?

Numbers, strings, colors, and lists have direct CSS counterparts. Sass can also use those values in calculations before writing the final CSS.

### 4. Are Sass maps themselves CSS declarations?

No. A map associates keys with values for Sass code. Sass can look up a value from a map and use that value in a declaration, but a map is not normally emitted as a CSS property value.

### 5. What is the difference between a Sass value and generated CSS?

A Sass value exists while Sass compiles the stylesheet. Generated CSS is the output Sass emits for the browser. Some values affect that output; others are consumed by Sass logic and never appear as CSS text.

### 6. What does @debug do?

The @debug rule prints an evaluated Sass value to the compiler's diagnostic output. It is a development aid and does not write a declaration to the generated stylesheet.

## Booleans and truthiness

### 7. What are the two boolean values?

They are true and false. Use them for an explicit yes-or-no state, such as whether a component option is enabled.

### 8. Which values are falsey in Sass?

Only false and null are falsey. Values such as 0, an empty string, and an empty list are truthy.

### 9. Will this condition run?

~~~scss
@if 0 {
  .answer {
    display: block;
  }
}
~~~

Yes. Zero is truthy in Sass, so the rule is emitted.

### 10. Will an empty string make an @if block run?

Yes. An empty string is truthy in Sass. Do not treat Sass conditionals as if they used JavaScript's truthiness rules.

### 11. How should I represent a feature that is explicitly turned off?

Use false. For example, a $show-icon: false configuration clearly communicates that the feature is disabled.

### 12. What boolean operators does Sass provide?

Sass provides not, and, and or. For example, true and false evaluates to false, while not false evaluates to true.

~~~scss
@debug true and false; // false
@debug not false;      // true
~~~

## Null and CSS output

### 13. What does null mean in Sass?

Null represents the absence of a value. It is Sass's only value of the null type and can also be returned by a function when there is no result.

### 14. Is null the same as false?

No. Both are falsey, but false means an explicit negative boolean answer. Null means that a value is absent. Use each according to its meaning.

### 15. What happens when a CSS property has a null value?

Sass omits the entire declaration.

~~~scss
$shadow: null;

.card {
  box-shadow: $shadow;
}
~~~

The output contains no box-shadow declaration. Sass does not emit box-shadow: null.

### 16. What happens to null inside a list used in CSS?

The null item is omitted while the other list items remain.

~~~scss
$fonts: "Inter", null, sans-serif;

body {
  font-family: $fonts;
}
~~~

The resulting value is equivalent to "Inter", sans-serif.

### 17. Why is null useful in a mixin?

It can represent an optional value. A mixin may set a declaration from an argument whose default is null; when the caller does not supply a value, Sass omits that declaration.

~~~scss
@mixin panel($shadow: null) {
  border: 1px solid #d8dde5;
  box-shadow: $shadow;
}
~~~

A caller can include the mixin without a shadow or pass a real shadow value when needed.

### 18. Can null be used in a condition?

Yes. Null is falsey, so an @if block conditioned on a null value is skipped. This can enable an optional style only after a value is provided.

### 19. Do all falsey Sass values disappear from CSS?

No. The special omission behavior discussed here belongs to null. False is a boolean and is not a general replacement for an absent property value. Use null when the value is absent.

### 20. What might a missing map lookup return?

A map lookup for a key that does not exist returns null. Code can use that result to recognize that there was no value for that key.

## Predict the result

### 21. What does this compile to?

~~~scss
$border-color: null;

.notice {
  color: #20242b;
  border-color: $border-color;
}
~~~

The color declaration is emitted. The border-color declaration is omitted because its value is null.

### 22. Which outline is emitted?

~~~scss
$enabled: false;

@if $enabled {
  .enabled {
    outline: 2px solid blue;
  }
}

@if not $enabled {
  .disabled {
    outline: 2px solid gray;
  }
}
~~~

Only the .disabled rule is emitted. The first condition is false; the second is true.

### 23. Which declarations are emitted here?

~~~scss
$accent: null;

.link {
  color: #2457a7;

  @if $accent {
    border-bottom: 2px solid $accent;
  }
}
~~~

Only color is emitted. The border rule is skipped because $accent is null and therefore falsey.

### 24. What does @debug show here?

~~~scss
$gap: 1rem 1.5rem;
$maybe-value: null;

@debug $gap;
@debug $maybe-value;
~~~

The compiler diagnostic output shows the list value 1rem 1.5rem and then null. Neither @debug statement becomes CSS.

## Common mistakes

### 25. Why should I not use JavaScript intuition for Sass conditions?

The languages have different truthiness rules. In Sass, zero, empty strings, and empty lists are truthy; only false and null are falsey. A condition copied from JavaScript may therefore behave differently.

### 26. If an optional value is missing, should I use an empty string?

Usually no. An empty string is truthy in Sass and can produce confusing output. Use null when the intended meaning is “no value.”

### 27. Why did my CSS property disappear?

Check whether its Sass value is null. Sass intentionally omits properties whose values are null. This is often expected behavior for optional declarations.

### 28. Why did my list have fewer items in CSS than in my Sass variable?

One or more list entries may have been null. Sass omits null list items from generated CSS.

### 29. Does a Sass map automatically become a CSS custom property or object?

No. Sass maps are Sass data structures. You need Sass logic to read or transform their contents into CSS declarations or values.

### 30. What is a good debugging sequence when output is unexpected?

1. Use @debug on the relevant variable or expression.
2. Check whether its value is null, false, or another falsey-sensitive value.
3. Check the generated CSS to see whether Sass omitted a declaration or a list item.
4. Confirm the condition uses a value with the intended meaning.

## Quick review

- A Sass expression produces a value.
- Variables store values.
- Sass supports CSS-like values and Sass-specific values.
- Only false and null are falsey.
- A null property value causes Sass to omit the declaration.
- Null items are omitted from generated CSS lists.
- False means “no”; null means “no value.”
- @debug writes to compiler diagnostics, not CSS.

## References

- [Sass values](https://sass-lang.com/documentation/values/)
- [Sass null](https://sass-lang.com/documentation/values/null/)
- [Sass booleans](https://sass-lang.com/documentation/values/booleans/)
- [Sass lists](https://sass-lang.com/documentation/values/lists/)
- [Sass maps](https://sass-lang.com/documentation/values/maps/)
