# 023 - Conditionals and Boolean Logic: Examples

Each snippet is SCSS you can compile with Dart Sass. Sass evaluates **@if** during compilation and emits CSS only from the selected branch.

## 1. Add an optional declaration

A Boolean variable can control whether Sass emits a declaration.

~~~scss
$show-border: true;

.card {
  padding: 1rem;

  @if $show-border {
    border: 1px solid #c9ced8;
  }
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1rem;
  border: 1px solid #c9ced8;
}
~~~

Change **$show-border** to **false** and the border declaration is omitted.

## 2. Choose a theme with @if and @else

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

Since **$theme** is **"dark"**, the output is:

~~~css
.panel {
  background-color: #202124;
  color: #ffffff;
}
~~~

The alternate branch is not emitted.

## 3. Select one option from an @else if chain

~~~scss
$button-size: "large";

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

Generated CSS for **"large"**:

~~~css
.button {
  padding: 0.9rem 1.25rem;
  font-size: 1.125rem;
}
~~~

Only the first matching branch is evaluated.

## 4. Use numeric comparisons

Sass can compare numbers to decide which statements to evaluate.

~~~scss
$columns: 4;

.grid {
  @if $columns > 1 {
    display: grid;
    grid-template-columns: repeat($columns, minmax(0, 1fr));
  } @else {
    display: block;
  }
}
~~~

Generated CSS:

~~~css
.grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
}
~~~

Use numeric comparisons such as **>**, **>=**, **<**, and **<=** when the condition depends on a number.

## 5. Combine checks with and

The **and** operator requires both expressions to be true.

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

The dark high-contrast styles are emitted only when both variables match.

## 6. Combine alternatives with or and parentheses

Use **or** when either of several options should satisfy a part of a condition. Parentheses show how the condition is grouped.

~~~scss
$theme: "dim";
$contrast: "high";

.notice {
  @if ($theme == "dark" or $theme == "dim") and $contrast == "high" {
    background-color: #111827;
    color: #ffffff;
  } @else {
    background-color: #f5f7fa;
    color: #172033;
  }
}
~~~

The condition succeeds because **$theme** matches **"dim"** and **$contrast** matches **"high"**.

## 7. Reverse a condition with not

~~~scss
$disabled: false;

.button {
  @if not $disabled {
    cursor: pointer;
  } @else {
    cursor: not-allowed;
    opacity: 0.55;
  }
}
~~~

Since **$disabled** is false, **not $disabled** is true and Sass emits the pointer cursor.

## 8. Sass truthiness: zero is true

Only **false** and **null** are falsey in Sass. Zero is truthy, so this condition succeeds:

~~~scss
@if 0 {
  .zero-is-truthy {
    display: block;
  }
}
~~~

Generated CSS:

~~~css
.zero-is-truthy {
  display: block;
}
~~~

If the intent is to check whether a number is positive, compare it explicitly:

~~~scss
$count: 0;

@if $count > 0 {
  .has-items {
    display: block;
  }
}
~~~

No CSS is generated for **.has-items** because the comparison is false.

## 9. Optional styles in a mixin

A mixin can use a Boolean argument to choose which declarations it emits.

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

The default call omits the circular style. The second call enables it with a named argument.

## 10. Independent @if rules can both run

Use separate **@if** rules when conditions are independent and both may be true.

~~~scss
$has-icon: true;
$is-disabled: true;

.button {
  @if $has-icon {
    gap: 0.5rem;
  }

  @if $is-disabled {
    opacity: 0.55;
  }
}
~~~

Both conditions are true, so both declarations are generated:

~~~css
.button {
  gap: 0.5rem;
  opacity: 0.55;
}
~~~

If the conditions are alternatives and only one should run, connect the blocks with **@else if**.

## 11. Sass conditionals versus media queries

A Sass conditional is decided at compile time:

~~~scss
$compact-build: true;

.card {
  @if $compact-build {
    padding: 0.75rem;
  } @else {
    padding: 1.5rem;
  }
}
~~~

The output contains only one padding declaration:

~~~css
.card {
  padding: 0.75rem;
}
~~~

A media query stays in the compiled CSS so the browser can evaluate the viewport:

~~~scss
.card {
  padding: 1.5rem;

  @media (max-width: 40rem) {
    padding: 0.75rem;
  }
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1.5rem;
}
@media (max-width: 40rem) {
  .card {
    padding: 0.75rem;
  }
}
~~~

Use a Sass conditional for a build-time choice. Use a media query for a browser-time viewport condition.

## 12. Inspect Boolean results while learning

**@debug** writes a message to the compiler output. It is useful for checking a Sass condition without adding a CSS declaration.

~~~scss
$enabled: true;
$count: 0;

@debug $enabled and not false;
@debug $count > 0;
@debug not null;
~~~

The debug results are **true**, **false**, and **true**. These messages appear in the Sass compiler output, not in the CSS file.

## Practice

1. Set **$theme** to **"light"** or **"dark"** and emit the matching text and background colors.
2. Add a third size to the button's **@else if** chain.
3. Change the grid example to generate a grid only when the column count is between **2** and **6**.
4. Add a **$reduced-motion** flag to a mixin and emit a transition only when it is false.
5. Explain why **@if 0** succeeds but **@if $count > 0** fails when **$count** is zero.

## Key takeaways

- **@if** chooses Sass output at compile time.
- **@else if** chains are for alternatives; independent **@if** rules may both run.
- Use **and**, **or**, and **not** to combine or reverse conditions.
- **false** and **null** are falsey; **0** is truthy.
- Use a Boolean for flags and explicit comparisons for numeric or optional values.
- Use CSS media queries when the browser must make the decision at runtime.

## Compile an example

Save a snippet as **conditionals.scss** and compile it with Dart Sass:

~~~sh
sass conditionals.scss conditionals.css
~~~

Compare the generated CSS with the expected output shown in each example. The **@debug** messages appear in the compiler output and are not emitted as CSS.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="022-built-in-sass-modules.md">022 - Built-in Sass Modules: Examples</a></td><td><a href="../lessons/023-conditionals-and-boolean-logic.md">Lesson</a></td><td><a href="../qnas/023-conditionals-and-boolean-logic.md">Q&amp;A</a></td><td><a href="024-loops-each-for-and-while.md">024 - Loops: each, for, and while: Examples</a></td></tr>
  </tbody>
</table>
