# 017 - Interpolation in Selectors and Property Names: Q&A

## Interpolation basics

### 1. What is Sass interpolation?

Interpolation inserts the result of a SassScript expression into CSS syntax. It uses the form #{expression}.

### 2. What does interpolation produce?

It inserts the expression's serialized text into the surrounding CSS syntax. Interpolated output is unquoted text.

### 3. Where is interpolation useful in this lesson?

It is useful for constructing selectors and property names from Sass variables or expressions.

### 4. Is interpolation required whenever I use a Sass variable?

No. Sass variables already work directly in ordinary property values. Use interpolation when the value is part of CSS syntax such as a selector or property name.

### 5. What is the difference between .#{$name} and color: $color?

The first inserts the name into a selector, so interpolation is needed. The second supplies an ordinary property value, so Sass evaluates the variable directly.

## Selectors and property names

### 6. How can a variable form a class selector?

Place interpolation in the selector, such as .#{$name}. If the variable is alert, Sass emits .alert.

### 7. Can interpolation be used in only part of a selector?

Yes. It can construct part of a class name or other selector token, for example .button--#{$state}.

### 8. Does interpolation add a descendant relationship?

No. Interpolation inserts text. Selector relationships such as descendant, child, and sibling relationships come from the actual selector syntax and whitespace.

### 9. How can interpolation form a property name?

Place it in the declaration name, such as margin-#{$side}. With side set to inline-start, Sass emits margin-inline-start.

### 10. Can interpolation create an entire property name?

Yes. A property name can consist entirely of interpolation, as in #{$property}: $value. Use this only when the dynamic property pattern is useful and its inputs are controlled.

### 11. Does Sass parse an interpolated selector?

Yes. Sass resolves interpolation and parses the resulting selector. The final selector must still be valid CSS.

### 12. Can a typo in a dynamic property name cause a problem?

Yes. A generated name that is not a meaningful CSS property may be ignored by the browser or fail to express the intended behavior. Keep possible names explicit and inspect the output.

## Values, numbers, and strings

### 13. Should I interpolate a variable in an ordinary color declaration?

Usually no. Write color: $color. Sass already evaluates the variable as a declaration value.

### 14. Why should I avoid interpolation with numbers?

Interpolation turns a numeric Sass value into unquoted text. This loses numeric type handling, so Sass can no longer reliably perform unit arithmetic or catch unit mistakes.

### 15. How should I add a unit to a number?

Prefer to store a value with its unit, such as 12px. If the number is intentionally unitless, use Sass arithmetic such as $value * 1px instead of concatenating text.

### 16. What happens to quotes around an interpolated string?

Interpolation removes the string's surrounding quotation marks. This can change the meaning of a CSS value for which quotes are significant.

### 17. How do I preserve quotes when serializing a value through interpolation?

Use meta.inspect() from the sass:meta module when the serialized value must retain its quotes. For ordinary property values, pass the Sass variable directly instead.

### 18. Do I need meta.inspect() for every interpolated string?

No. Use it only when the output requires the quoted representation. Selectors and property names generally need unquoted selector or identifier text.

## CSS custom properties

### 19. Why are custom properties different from ordinary properties?

CSS custom property values can contain almost arbitrary CSS text, so Sass passes their values through rather than evaluating SassScript in the ordinary way.

### 20. How do I insert a Sass variable into a CSS custom property?

Use interpolation inside the value, for example --accent: #{$accent}.

### 21. What happens if I write a Sass variable directly in a custom property?

The dollar-prefixed text may be emitted literally as part of the custom property value rather than being replaced by the Sass variable.

### 22. How do I preserve a quoted font stack in a custom property?

Use meta.inspect() inside interpolation so the quotes are included in the serialized CSS value.

## Reuse and variable lookup

### 23. Should I build selectors dynamically for a one-off static rule?

Usually not. A literal selector is often easier to read and find. Interpolation is more useful when a repeated pattern truly depends on a value.

### 24. Can interpolation create a Sass variable name to look up?

No. Sass does not support dynamically interpolated variable names. Use a map when a name should select a value.

### 25. How can a map replace dynamic variable names?

Store the names and values in a map, then call map.get() with the chosen key. This makes the available choices explicit.

### 26. Is interpolation the same as string concatenation?

It is a way to insert an expression into text. In selectors, Sass parses the result as selector syntax, so it should not be treated as unconstrained string formatting.

### 27. Can interpolation generate selectors inside a mixin?

Yes. A mixin can interpolate parameters into selector names or property names. Keep its inputs documented and use expected values.

### 28. Does interpolation in a property value always improve compatibility?

No. Interpolation can remove quotes and number types. Forwards-compatible Sass often means using Sass values directly and reserving interpolation for CSS syntax construction.

### 29. Is interpolation needed to use a variable inside calc()?

Usually no. Sass calculations understand Sass variables directly. Interpolating inside a calculation can prevent Sass from simplifying or type-checking the expression.

### 30. What should I inspect after adding interpolation?

Compile the stylesheet and check the final selector, property name, and value. Confirm that the selector matches the intended markup and that quotes and units survived as intended.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="016-parent-selector.md">016 - The Parent Selector: Q&amp;A</a></td><td><a href="../lessons/017-interpolation-in-selectors-and-property-names.md">Lesson</a></td><td><a href="../examples/017-interpolation-in-selectors-and-property-names.md">Example</a></td><td><a href="018-mixin-basics.md">018 - Mixin Basics: Q&amp;A</a></td></tr>
  </tbody>
</table>
