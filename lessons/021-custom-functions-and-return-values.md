# 021 - Custom Functions and Return Values

## Learning goals

After this lesson, you should be able to:

- Define a Sass function with **@function**.
- Accept arguments and use them in a calculation.
- Return a value with **@return**.
- Use a custom function inside an expression or CSS declaration.
- Choose between a function and a mixin.

## 1. What is a custom function?

Sass includes built-in functions for common tasks. When your stylesheets need a project-specific calculation or transformation, you can define a custom function and reuse it.

A function accepts values, computes a result, and returns that result to the expression that called it. Sass evaluates the function while compiling the stylesheet; the function itself does not create a CSS rule.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}

.card {
  padding: spacing(4);
}
~~~

The function call evaluates to **1rem**, so Sass generates:

~~~css
.card {
  padding: 1rem;
}
~~~

The spacing formula lives in one place, while each call site selects a step.

## 2. Function structure and arguments

A function has a name, an argument list, and a body. Its result is supplied by **@return**.

~~~scss
@function function-name($argument) {
  // Compute a Sass value.
  @return $argument;
}
~~~

Arguments become variables that the function can use. You can give an argument a default value so callers may omit it:

~~~scss
@function spacing($step, $base: 0.25rem) {
  @return $base * $step;
}

.stack {
  gap: spacing(3);
}

.panel {
  padding: spacing(4, 0.5rem);
}
~~~

The first call uses the default base and returns **0.75rem**. The second passes a different base and returns **2rem**.

A function can also use local variables to make a multi-step calculation easier to read:

~~~scss
@function spacing($step, $base: 0.25rem) {
  $result: $base * $step;
  @return $result;
}
~~~

Choose arguments that fit the operation. This spacing function expects the step to be a number; a string such as **large** cannot be multiplied by a length.

## 3. How @return works

**@return** specifies the result of a function call. It is only valid inside a **@function** body, and every function must provide a return value.

~~~scss
@function double($number) {
  @return $number * 2;
}
~~~

When Sass reaches **@return**, it immediately ends the function and gives the value to the caller. Any statements after that return are not executed.

~~~scss
@function card-padding($size) {
  $padding: $size * 1rem;
  @return $padding;
}
~~~

A function that calculates a variable but never returns it is incomplete:

~~~scss
// Incorrect: the function does not return a value.
@function double($number) {
  $result: $number * 2;
}
~~~

Do not put **@return** at the top level of a stylesheet or inside a mixin.

## 4. Use the returned value

Call a custom function with its name and arguments in parentheses. Use the resulting value wherever a Sass expression is accepted.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}

$section-gap: spacing(8);

.page {
  padding-inline: spacing(4);
  margin-block: $section-gap;
  gap: spacing(2);
}
~~~

Sass evaluates the calls before writing the CSS:

~~~css
.page {
  padding-inline: 1rem;
  margin-block: 2rem;
  gap: 0.5rem;
}
~~~

A returned value can also be part of a larger expression:

~~~scss
.card {
  margin-block-end: spacing(2) + spacing(1);
}
~~~

That expression returns **0.75rem**. The lengths can be added because they have compatible units.

## 5. Functions and mixins have different jobs

Both features help reuse Sass logic, but they are called differently and solve different problems.

| Use a function when… | Use a mixin when… |
| --- | --- |
| You need a computed value. | You need Sass to insert declarations, selectors, or at-rules. |
| The result belongs in a property value or expression. | The reusable behavior should emit CSS output. |
| The operation is a calculation or transformation. | The caller needs a reusable block of styles. |

A function returns a value:

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}

.card {
  padding: spacing(4);
}
~~~

A mixin inserts declarations:

~~~scss
@mixin visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip-path: inset(50%);
}

.skip-link-text {
  @include visually-hidden;
}
~~~

Keep functions focused on computing values. Use a mixin when you need reusable style output or a content block.

## 6. What can a function return?

A custom function can return Sass values such as numbers, strings, colors, lists, maps, and Booleans. The returned value still needs to make sense where the call is used.

~~~scss
@function brand-color() {
  @return #2457c5;
}

.button {
  background-color: brand-color();
}
~~~

A color is meaningful for **background-color**, but not for **width**. Document expected arguments and return types when a function is part of a shared stylesheet API.

## 7. Naming and organization

Choose a name that explains the result, such as **spacing**, **rem-from-pixels**, or **brand-color**. A descriptive function call is easier to understand than a vague name such as **calculate**.

Sass treats hyphens and underscores as equivalent in identifiers. Pick one style and use it consistently; hyphens commonly match CSS naming.

Shared functions can live in a partial or Sass module and be loaded wherever needed. Module loading is covered in a later lesson. For now, keep the definition above its calls in the same stylesheet.

## 8. Common mistakes

### Forgetting to return

A calculation does not automatically become the function result. Include **@return**.

### Using a function when a mixin is needed

A function computes one value. It does not insert a group of declarations. Use a mixin for reusable CSS output.

### Returning a value that does not fit

Check the result's type and units. A spacing function should return a length that is valid for the property receiving it.

### Expecting Sass functions to run in the browser

Custom Sass functions run during compilation. The browser receives the resulting CSS, not the Sass function definition.

### Giving a function a vague name

Names should describe the returned value or operation so the call site communicates intent.

## 9. Complete example

~~~scss
@function spacing($step, $base: 0.25rem) {
  $result: $base * $step;
  @return $result;
}

$card-space: spacing(4);

.card {
  padding: $card-space;
  gap: spacing(2);
  margin-block-end: spacing(3, 0.5rem);
}
~~~

The calls return **1rem**, **0.5rem**, and **1.5rem**. Sass generates:

~~~css
.card {
  padding: 1rem;
  gap: 0.5rem;
  margin-block-end: 1.5rem;
}
~~~

The function owns the formula, and each call site chooses the step and optional base.

## 10. Practice

1. Write a function named **double** that accepts a number and returns twice that number.
2. Write a **spacing** function with a default base of **0.25rem**.
3. Call the function for padding and gap in a **.layout** rule.
4. Explain why a function can return a padding value, while a mixin should insert several declarations.
5. What is missing from a function that calculates a local variable but has no **@return**?

## Key takeaways

- **@function** defines a reusable Sass calculation.
- Function arguments are available as local variables.
- **@return** provides the result and immediately ends the function.
- Call a custom function with its name and arguments in parentheses.
- Functions compute values; mixins insert reusable CSS output.
- Sass evaluates custom functions during compilation.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="020-content-blocks-and-context-aware-mixins.md">020 - Content Blocks and Context-Aware Mixins</a></td><td><a href="../qnas/021-custom-functions-and-return-values.md">Q&amp;A</a></td><td><a href="../examples/021-custom-functions-and-return-values.md">Example</a></td><td><a href="022-built-in-sass-modules.md">022 - Built-in Sass Modules</a></td></tr>
  </tbody>
</table>
