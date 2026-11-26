# 023 - Conditionals and Boolean Logic

## Learning goals

After this lesson, you should be able to:

- Use **@if** to conditionally evaluate Sass statements.
- Add fallback behavior with **@else** and **@else if**.
- Compare Sass values with equality and numeric comparison operators.
- Combine conditions with **and**, **or**, and **not**.
- Explain Sass truthiness, including why **0** is truthy.
- Keep Sass compile-time decisions distinct from browser-run CSS conditions.

## 1. What is a conditional?

A conditional lets Sass choose which statements to evaluate. The **@if** rule evaluates its block when its expression is true. If the expression is false, Sass skips the block.

~~~scss
$show-border: true;

.card {
  padding: 1rem;

  @if $show-border {
    border: 1px solid #c9ced8;
  }
}
~~~

Because **$show-border** is true, the compiled CSS includes the border:

~~~css
.card {
  padding: 1rem;
  border: 1px solid #c9ced8;
}
~~~

Sass evaluates **@if** during compilation. The browser receives only the CSS produced by the selected Sass statements.

## 2. Add an alternative with @else

Use **@else** for the fallback block. Sass evaluates it only when the preceding **@if** condition is false.

~~~scss
$theme: "dark";

.panel {
  @if $theme == "light" {
    background-color: #ffffff;
    color: #202124;
  } @else {
    background-color: #202124;
    color: #ffffff;
  }
}
~~~

Since **$theme** is **"dark"**, Sass generates:

~~~css
.panel {
  background-color: #202124;
  color: #ffffff;
}
~~~

Only one of the two branches is evaluated.

## 3. Select between several cases with @else if

Use **@else if** when there are several mutually exclusive cases. Sass evaluates the conditions in order and uses the first matching branch. A final **@else** is optional.

~~~scss
$button-size: "medium";

.button {
  @if $button-size == "small" {
    padding: 0.4rem 0.7rem;
    font-size: 0.875rem;
  } @else if $button-size == "medium" {
    padding: 0.65rem 1rem;
    font-size: 1rem;
  } @else if $button-size == "large" {
    padding: 0.9rem 1.25rem;
    font-size: 1.125rem;
  } @else {
    padding: 0.65rem 1rem;
    font-size: 1rem;
  }
}
~~~

The value **"medium"** selects the second branch. If no size matched, the final **@else** would supply the fallback styles.

Keep a branch chain for values that are alternatives to one another. For independent conditions, separate **@if** rules may be clearer.

## 4. Compare values

Use **==** to test whether two Sass values are equal and **!=** to test whether they differ.

~~~scss
$layout: "wide";

.container {
  @if $layout == "wide" {
    max-width: 72rem;
  } @else {
    max-width: 48rem;
  }
}
~~~

Sass also supports **<**, **<=**, **>**, and **>=** for comparing numbers. Use values whose units can be meaningfully compared.

~~~scss
$columns: 4;

.grid {
  @if $columns > 1 {
    display: grid;
    grid-template-columns: repeat($columns, minmax(0, 1fr));
  }
}
~~~

Sass comparison operators are evaluated while compiling. They are not CSS selectors and do not create browser conditions.

## 5. Combine conditions with Boolean operators

Sass supports **and**, **or**, and **not**.

- **A and B** is true only when both conditions are true.
- **A or B** is true when at least one condition is true.
- **not A** reverses whether a condition is true.

~~~scss
$theme: "dark";
$contrast: "high";

.notice {
  @if $theme == "dark" and $contrast == "high" {
    background-color: #111827;
    color: #ffffff;
  } @else {
    background-color: #f5f7fa;
    color: #172033;
  }
}
~~~

The branch is selected only when both comparisons match.

Use parentheses to make a combined condition easier to read:

~~~scss
@if ($theme == "dark" or $theme == "dim") and $contrast == "high" {
  // Statements for either dark theme with high contrast.
}
~~~

The parentheses make it clear that the first two theme options are grouped together before the contrast check.

## 6. Sass truthiness

An **@if** expression does not have to be a literal **true** or **false**. Sass treats **false** and **null** as falsey. Every other value is truthy.

That means **0**, an empty string, and an empty list are all truthy in Sass. This can surprise developers coming from languages where zero or empty collections are falsey.

~~~scss
@if 0 {
  .zero-is-truthy {
    display: block;
  }
}
~~~

This condition succeeds in Sass because **0** is truthy.

For optional styles, use an explicit Boolean variable:

~~~scss
$show-shadow: false;

.card {
  @if $show-shadow {
    box-shadow: 0 0.25rem 1rem rgb(0 0 0 / 12%);
  }
}
~~~

If the condition is about whether a value exists, check for **null** explicitly. If it is about a particular number, string, or option, compare that value directly. Avoid relying on accidental truthiness.

## 7. Use conditionals inside mixins

A mixin can use a conditional to decide which declarations it emits.

~~~scss
@mixin avatar($size, $circle: false) {
  width: $size;
  height: $size;

  @if $circle {
    border-radius: 50%;
  }
}

.square-avatar {
  @include avatar(4rem);
}

.round-avatar {
  @include avatar(4rem, $circle: true);
}
~~~

Generated CSS:

~~~css
.square-avatar {
  width: 4rem;
  height: 4rem;
}

.round-avatar {
  width: 4rem;
  height: 4rem;
  border-radius: 50%;
}
~~~

The mixin's Boolean argument controls whether the border-radius declaration is generated. A named argument such as **$circle: true** makes the call's meaning clear.

## 8. Conditional rules can select styles

Conditionals can evaluate Sass statements that emit whole style rules, not just declarations.

~~~scss
$include-print-rules: true;

@if $include-print-rules {
  @media print {
    .screen-only {
      display: none;
    }
  }
}
~~~

When the variable is true, Sass emits the **@media** rule. If it were false, Sass would skip that rule entirely.

Use this for build-time choices in Sass. Use CSS media queries or other CSS features when the browser should make the decision based on the current device, user preference, or runtime state.

## 9. Sass conditionals are not browser conditions

A Sass **@if** chooses which Sass statements to compile. It does not remain in the stylesheet for the browser to evaluate.

For example, this uses a Sass variable at compile time:

~~~scss
$compact: true;

.card {
  @if $compact {
    padding: 0.75rem;
  } @else {
    padding: 1.5rem;
  }
}
~~~

The output contains one padding declaration. By contrast, a CSS media query remains in the output so the browser can evaluate the viewport:

~~~scss
.card {
  padding: 1.5rem;

  @media (max-width: 40rem) {
    padding: 0.75rem;
  }
}
~~~

Use Sass conditionals for configuration known when compiling. Use CSS conditions for facts the browser must evaluate.

## 10. Operator precedence

Sass expressions follow an order of operations. Arithmetic and comparisons are evaluated before **and**, and **and** is evaluated before **or**. Parentheses can make the intended grouping explicit.

~~~scss
$wide: true;
$compact: false;

@if $wide or $compact and false {
  // The condition is true because $wide is true.
}
~~~

Even when the language has a defined precedence, complex conditions are easier to maintain with parentheses and well-named intermediate variables.

~~~scss
$use-wide-layout: $wide or ($compact and false);

@if $use-wide-layout {
  // Apply the wide layout.
}
~~~

Prefer readable conditions over relying on readers to calculate precedence mentally.

## 11. Common mistakes

### Treating zero as false

In Sass, **0** is truthy. Compare a number explicitly if the condition depends on whether it is greater than zero.

~~~scss
@if $count > 0 {
  // There is at least one item.
}
~~~

### Using two independent @if rules for alternatives

Two separate rules can both run. Use **@if**, **@else if**, and **@else** when exactly one alternative should be selected.

### Forgetting that Sass decides at compile time

An **@if** does not respond to a browser viewport or a user's interaction. Use CSS conditions for runtime behavior.

### Writing unclear compound logic

Break a long expression into named variables or add parentheses around its logical groups.

### Passing truthy values when a Boolean is expected

Prefer **true** or **false** for flags. A non-empty string, a number, or a list can also make an **@if** condition succeed, which may hide a mistaken argument.

## 12. Practice

1. Use **@if** and **@else** to choose between light and dark text colors.
2. Use **@else if** to generate small, medium, or large padding.
3. Write a condition that applies a rule only when **$columns** is greater than **1**.
4. Combine two checks with **and**, and explain when the branch runs.
5. Explain why **@if 0** succeeds in Sass.
6. Choose whether each requirement needs Sass **@if** or a CSS media query: a build-time feature flag; a style that changes when the viewport becomes narrow.

## Key takeaways

- **@if** evaluates its block only when its expression is truthy.
- Use **@else** and **@else if** to handle alternative cases.
- **and**, **or**, and **not** combine or reverse conditions.
- Only **false** and **null** are falsey in Sass; values such as **0** are truthy.
- Sass conditionals select output at compile time; CSS media queries are evaluated by the browser.
- Parentheses and named variables make compound logic easier to read.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="022-built-in-sass-modules.md">022 - Built-in Sass Modules</a></td><td><a href="../qnas/023-conditionals-and-boolean-logic.md">Q&amp;A</a></td><td><a href="../examples/023-conditionals-and-boolean-logic.md">Example</a></td><td><a href="024-loops-each-for-and-while.md">024 - Loops: each, for, and while</a></td></tr>
  </tbody>
</table>
