# 023 — Conditionals and Boolean Logic: Q&A

## Core behavior

### 1. What does Sass @if do?

**@if** evaluates its block when its expression is truthy. If the condition is falsey, Sass skips that block.

~~~scss
$show-border: true;

.card {
  @if $show-border {
    border: 1px solid #c9ced8;
  }
}
~~~

The border declaration is included because the condition is true.

### 2. What happens to an @if rule in the compiled CSS?

Sass evaluates it during compilation. The **@if** rule itself is not sent to the browser; Sass emits the CSS from the selected branch.

### 3. How do @else and @else if work?

**@else** supplies a fallback when the preceding condition is false. **@else if** checks another condition when earlier conditions failed.

~~~scss
@if $size == "small" {
  // Small case.
} @else if $size == "large" {
  // Large case.
} @else {
  // Default case.
}
~~~

In a chain, only the first matching branch is evaluated.

### 4. Is a final @else required?

No. Use it when a default behavior is useful. Without a final **@else**, Sass simply emits nothing from the chain when no condition matches.

### 5. When should I use an @else if chain?

Use it for mutually exclusive alternatives, such as a value that can be small, medium, or large. The first matching branch wins.

### 6. When should I use several separate @if rules?

Use separate rules when their conditions are independent and more than one may be true. They are not alternatives to one another.

## Conditions and operators

### 7. How do I test whether two values are equal?

Use **==**. Use **!=** to test whether they are different.

~~~scss
@if $theme == "dark" {
  color: white;
}
~~~

### 8. Which comparison operators work with numbers?

Sass supports **<**, **<=**, **>**, and **>=**. Use comparisons with numeric values and units that Sass can meaningfully compare.

~~~scss
@if $columns > 1 {
  display: grid;
}
~~~

### 9. How do and, or, and not work?

- **and** succeeds only when both expressions are true.
- **or** succeeds when either expression is true.
- **not** reverses the truth value of an expression.

~~~scss
@if $theme == "dark" and $contrast == "high" {
  // Both settings must match.
}
~~~

### 10. Can I use JavaScript-style && and ||?

No. Sass Boolean operators are the words **and**, **or**, and **not**.

### 11. What is the precedence of Boolean operators?

**and** is evaluated before **or**, so an **and** group can bind more tightly in a mixed expression. Use parentheses to show the intended grouping.

~~~scss
@if ($theme == "dark" or $theme == "dim") and $contrast == "high" {
  // Either theme is accepted, and contrast must also be high.
}
~~~

Parentheses make the logic clear even when you know the precedence rules.

### 12. Can a condition be a comparison or logical expression?

Yes. An **@if** expression can use comparisons and Boolean operators. It can also be a value whose Sass truthiness determines whether the block runs.

## Truthiness and falsiness

### 13. Which Sass values are falsey?

Only **false** and **null** are falsey. Sass treats all other values as truthy.

### 14. Is zero falsey in Sass?

No. **0** is truthy in Sass. If a condition is about whether a number is greater than zero, test it explicitly.

~~~scss
@if $count > 0 {
  // At least one item exists.
}
~~~

### 15. Are an empty string or empty list falsey?

No. They are truthy in Sass. Do not assume Sass follows the truthiness rules of JavaScript, Python, or another language.

### 16. Should I use arbitrary values as Boolean flags?

Prefer explicit **true** and **false** values for flags. Since numbers, strings, lists, and other values are truthy, an unexpected argument may accidentally enable a branch.

### 17. How should I test whether an optional value is missing?

Use an explicit comparison with **null** when absence is what matters.

~~~scss
@if $optional-value != null {
  // Use the configured value.
}
~~~

This communicates the intent more clearly than a general truthiness check.

## Sass conditionals and CSS

### 18. Does @if respond to the browser viewport?

No. Sass evaluates **@if** at compile time. Use a CSS media query when the browser should choose styles based on the current viewport.

### 19. What is the difference between a Sass conditional and a media query?

A Sass conditional decides which Sass statements produce output before the browser receives the CSS. A media query remains in the CSS and the browser evaluates it at runtime.

### 20. Can an @if condition generate a whole CSS rule?

Yes. An **@if** can control Sass statements that emit declarations, selectors, and at-rules. If the condition is false, the corresponding CSS output is omitted.

### 21. When should I use a Sass conditional for a feature flag?

Use a Sass conditional if the feature is decided when the stylesheet is compiled. If the choice depends on browser state, user interaction, or viewport size, use CSS or application logic instead.

## Common design questions

### 22. Can I use a conditional inside a mixin?

Yes. A mixin can use **@if** to decide which declarations it emits based on its arguments.

~~~scss
@mixin avatar($size, $circle: false) {
  width: $size;
  height: $size;

  @if $circle {
    border-radius: 50%;
  }
}
~~~

### 23. Why did two branches both add styles?

They may be separate **@if** rules. Separate rules are evaluated independently. If only one branch should run, connect the branches with **@else if** and **@else**.

### 24. Why did my zero-valued flag enable the conditional?

Zero is truthy in Sass. Use a Boolean flag or an explicit numeric comparison such as **$value > 0**.

### 25. How can I make a long condition easier to maintain?

Split parts into well-named variables or add parentheses around logical groups. Use names that explain the condition instead of repeating complex expressions.

### 26. What if a branch does not match any condition?

If there is no final **@else**, Sass evaluates none of the chain's blocks and emits no CSS from them. Add a fallback branch when an unknown or default value needs handling.

## Quick review

- **@if** selects Sass output at compile time.
- **@else if** checks additional alternatives; **@else** supplies a fallback.
- Sass Boolean operators are **and**, **or**, and **not**.
- Only **false** and **null** are falsey; **0** and empty values are truthy.
- Separate **@if** rules can both run; an if/else chain selects at most one branch.
- Use Sass conditions for compile-time choices and CSS conditions for browser runtime choices.

## Further reading

- [Sass documentation: @if and @else](https://sass-lang.com/documentation/at-rules/control/if/)
- [Sass documentation: operators](https://sass-lang.com/documentation/operators/)
