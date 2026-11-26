# 021 - Custom Functions and Return Values: Examples

These examples show how to define Sass functions, pass arguments, return values, and use the results in CSS declarations. Each example is self-contained; compile a snippet on its own or combine the function definitions in one stylesheet.

## 1. A spacing function

Use a function when a repeated formula should return one value.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}

.card {
  padding: spacing(4);
  gap: spacing(2);
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1rem;
  gap: 0.5rem;
}
~~~

The function accepts a unitless number and returns a length.

## 2. A configurable base with a default

A default argument lets callers use the common spacing scale or provide a different base for a special case.

~~~scss
@function spacing($step, $base: 0.25rem) {
  @return $base * $step;
}

.default-panel {
  padding: spacing(4);
}

.custom-panel {
  padding: spacing(3, $base: 0.5rem);
}
~~~

Generated CSS:

~~~css
.default-panel {
  padding: 1rem;
}

.custom-panel {
  padding: 1.5rem;
}
~~~

The named argument makes the custom base clear at the call site. It also avoids relying on argument position when a function has several optional arguments.

## 3. Return a color value

A function can return a Sass value other than a number. Here the function returns a project color for use in a declaration.

~~~scss
@function brand-color() {
  @return #2457c5;
}

.button {
  background-color: brand-color();
  color: white;
}
~~~

Generated CSS:

~~~css
.button {
  background-color: #2457c5;
  color: white;
}
~~~

The function returns the color. The selector and declarations are still written by the stylesheet that calls it.

## 4. Use a local variable for a multi-step result

Local variables can make calculations easier to read. The function still needs to return its final value.

~~~scss
@function border-width($factor, $base: 1px) {
  $result: $base * $factor;
  @return $result;
}

.panel {
  border-width: border-width(2);
}

.dialog {
  border-width: border-width(3, $base: 0.5px);
}
~~~

Generated CSS:

~~~css
.panel {
  border-width: 2px;
}

.dialog {
  border-width: 1.5px;
}
~~~

Both arguments represent numbers with compatible units: the factor is unitless, and the base is a length.

## 5. Combine returned values in an expression

A function call returns a Sass value, so it can be combined with other compatible values.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}

.article {
  padding-inline: spacing(4);
  margin-block-end: spacing(2) + spacing(1);
}
~~~

Generated CSS:

~~~css
.article {
  padding-inline: 1rem;
  margin-block-end: 0.75rem;
}
~~~

The addition is valid because both function calls return lengths in rem units.

## 6. Compare a function with a mixin

The function supplies a value for a declaration. The mixin inserts a set of declarations.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}

@mixin focus-outline {
  outline: 3px solid #2457c5;
  outline-offset: 2px;
}

.card {
  padding: spacing(4);
}

.button:focus-visible {
  @include focus-outline;
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1rem;
}

.button:focus-visible {
  outline: 3px solid #2457c5;
  outline-offset: 2px;
}
~~~

Use the function for the calculated padding value and the mixin for the reusable outline declarations.

## 7. A complete small stylesheet

This example combines a function with a default argument, a named argument, a local variable, and several call sites.

~~~scss
@function spacing($step, $base: 0.25rem) {
  $result: $base * $step;
  @return $result;
}

$card-padding: spacing(4);

.card {
  padding: $card-padding;
  gap: spacing(2);
  margin-block-end: spacing(3, $base: 0.5rem);
}

.card-title {
  margin-block-end: spacing(1);
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1rem;
  gap: 0.5rem;
  margin-block-end: 1.5rem;
}

.card-title {
  margin-block-end: 0.25rem;
}
~~~

Each call returns a value. Sass substitutes that value into the declaration during compilation.

## 8. Common mistakes to recognize

### Calculation without a return

~~~scss
// Incorrect: there is no return value.
@function spacing($step) {
  $result: 0.25rem * $step;
}
~~~

Correct it by returning the calculation or local variable:

~~~scss
@function spacing($step) {
  $result: 0.25rem * $step;
  @return $result;
}
~~~

### Putting style declarations in a function

A function is for computing a result, not for emitting a block of CSS. Use a mixin when a reusable set of declarations is needed.

### Passing an unexpected type

The spacing examples expect a unitless numeric step. Passing a string or a length as the step does not represent the intended calculation.

~~~scss
// This is not a valid spacing call for the function above:
.card {
  padding: spacing(large);
}
~~~

Use a numeric step such as **spacing(4)**.

## 9. Practice and solutions

### Practice A

Write a function named **double** that returns twice its numeric argument, then use it for a CSS value.

~~~scss
@function double($number) {
  @return $number * 2;
}

.heading {
  font-size: double(1rem);
}
~~~

Expected declaration: **font-size: 2rem**.

### Practice B

Write a spacing function with a default base of **0.25rem**, then use a custom base for one call.

~~~scss
@function spacing($step, $base: 0.25rem) {
  @return $base * $step;
}

.list {
  gap: spacing(2);
}

.hero {
  padding: spacing(3, $base: 0.5rem);
}
~~~

Expected values: **0.5rem** for the list gap and **1.5rem** for the hero padding.

## Compilation

Save a snippet as a Sass file, then compile it with Dart Sass:

~~~sh
sass input.scss output.css
~~~

Compare the generated values with the expected CSS above. Sass function definitions are compile-time helpers; only the resulting CSS is sent to the browser.

## Key takeaways

- A custom function returns a value that can be used in a Sass expression.
- Use **@return** to provide the result.
- Defaults make common calls concise; named arguments clarify special calls.
- Keep argument types and units compatible with the calculation.
- Use a mixin when reusable CSS declarations need to be emitted.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="020-content-blocks-and-context-aware-mixins.md">020 - Content Blocks and Context-Aware Mixins: Examples</a></td><td><a href="../lessons/021-custom-functions-and-return-values.md">Lesson</a></td><td><a href="../qnas/021-custom-functions-and-return-values.md">Q&amp;A</a></td><td><a href="022-built-in-sass-modules.md">022 - Built-in Sass Modules: Examples</a></td></tr>
  </tbody>
</table>
