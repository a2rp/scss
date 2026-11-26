# 021 - Custom Functions and Return Values: Q&A

## Core concepts

### 1. What is a custom Sass function?

A custom function is a reusable operation you define with **@function**. It accepts arguments, computes a Sass value, and gives the result back to the expression that called it.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}

.card {
  padding: spacing(4);
}
~~~

Here, the function returns **1rem**, which Sass writes as the padding value.

### 2. What is the basic syntax for defining a function?

Write **@function**, a function name, parentheses containing the argument list, and a body that returns a value.

~~~scss
@function double($number) {
  @return $number * 2;
}
~~~

### 3. How do you call a custom function?

Use the function name followed by its arguments in parentheses, just like a function call in Sass or CSS.

~~~scss
@function double($number) {
  @return $number * 2;
}

.heading {
  font-size: double(1rem);
}
~~~

The call returns **2rem**.

### 4. What does @return do?

**@return** sets the result of the function call and immediately ends the function. The returned Sass value is used at the call site.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}
~~~

### 5. Is @return optional?

No. A Sass function must provide a return value. A function that performs a calculation but never returns it is incomplete.

~~~scss
// Incomplete: there is no return value.
@function double($number) {
  $result: $number * 2;
}
~~~

Add **@return $result;** or return the calculation directly.

### 6. Can @return be used outside a function?

No. **@return** is only allowed inside a **@function** body. It cannot return from a stylesheet or from a mixin.

## Arguments and results

### 7. How are function arguments used?

The arguments become variables inside the function body. Use them in calculations or other Sass expressions.

~~~scss
@function border-width($factor) {
  @return 1px * $factor;
}
~~~

Calling **border-width(3)** returns **3px**.

### 8. Can a function argument have a default value?

Yes. Add a default after the argument name. A caller can omit it or provide another value.

~~~scss
@function spacing($step, $base: 0.25rem) {
  @return $base * $step;
}

.small-gap {
  gap: spacing(2);
}

.large-gap {
  gap: spacing(4, 0.5rem);
}
~~~

The first call returns **0.5rem**. The second returns **2rem**.

### 9. Can a function return a value other than a number?

Yes. A function can return Sass values such as a color, string, list, map, or Boolean. Choose a result that makes sense in the expression that uses it.

~~~scss
@function brand-color() {
  @return #2457c5;
}

.button {
  background-color: brand-color();
}
~~~

### 10. Can a function use local variables?

Yes. Local variables can make a longer calculation easier to understand. Return the final result explicitly.

~~~scss
@function spacing($step, $base: 0.25rem) {
  $result: $base * $step;
  @return $result;
}
~~~

The variable **$result** belongs to the function's calculation; the value becomes available to the caller through **@return**.

### 11. What happens when Sass reaches @return?

The function ends immediately and the returned value is passed to the caller. Any statements after that **@return** are not run.

~~~scss
@function double($number) {
  @return $number * 2;
  // This statement would not be reached.
}
~~~

### 12. Why do units matter in a function?

Sass performs unit-aware arithmetic. Compatible units can be combined; incompatible units or values can produce an error. Design the function's inputs so the calculation is meaningful.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}
~~~

This function expects a unitless numeric step. Passing a length such as **2px** would produce a squared unit, which is not a useful spacing value.

## Functions compared with other Sass features

### 13. How is a function different from a mixin?

A function computes and returns a value for use in an expression. A mixin inserts reusable Sass statements, commonly CSS declarations or rules.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}

@mixin compact-card {
  padding: 1rem;
  gap: 0.5rem;
}

.card {
  padding: spacing(4);
  @include compact-card;
}
~~~

Use the function for a value calculation. Use the mixin when you want to emit styles.

### 14. Does calling a Sass function create a CSS rule?

No. The call evaluates as a value where it is used. The function definition itself does not emit a selector or declarations.

### 15. Does a Sass custom function run in the browser?

No. Sass evaluates it during compilation. The browser receives the generated CSS value, not the Sass function definition.

### 16. What is the difference between a custom Sass function and a built-in function?

A custom function is defined in your Sass stylesheet with **@function**. A built-in function is supplied by Sass. Both can be called as values, but built-in functions have behavior defined by Sass.

### 17. What happens if Sass does not recognize a function name?

An unrecognized function call may be preserved as a plain CSS function call in the output. This supports CSS functions Sass does not implement, but it can also hide a typo. Check unfamiliar function names and inspect compiled CSS when a value looks wrong.

## Debugging and design

### 18. Why might a custom function produce an invalid CSS value?

Common causes include a missing **@return**, an unexpected argument type, incompatible units, or using the returned value in a property that does not accept it. Inspect both the function inputs and the property where the result is used.

### 19. Should a function change global variables or emit styles?

Functions should compute and return values. Although Sass has mechanisms that can create side effects, they make function calls surprising and are discouraged. Use a mixin for reusable CSS output.

### 20. How should a shared function be named?

Use a name that describes the result or operation, such as **spacing** or **brand-color**. Clear names make call sites easier to read. Sass treats hyphens and underscores as equivalent in identifiers, so choose one style and keep it consistent.

### 21. Where should a shared function be defined?

It can be placed in a partial or Sass module and loaded by the stylesheets that need it. Module organization is covered in a later lesson. When learning the syntax, define the function before using it in the same stylesheet.

### 22. How can you check a function's result?

Call it in a declaration with a suitable property, then compile the Sass and inspect the generated CSS. Compare the expected type and units with the emitted value.

~~~scss
@function spacing($step) {
  @return 0.25rem * $step;
}

.example {
  margin: spacing(3);
}
~~~

Expected CSS:

~~~css
.example {
  margin: 0.75rem;
}
~~~

## Quick review

- **@function** defines a custom value calculation.
- Arguments are local variables available to the function.
- **@return** is required and ends the function immediately.
- A function call evaluates to a value; it does not insert a block of CSS.
- Use a mixin when reusable CSS output is required.
- Make sure arguments, units, returned values, and consuming properties fit together.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="020-content-blocks-and-context-aware-mixins.md">020 - Content Blocks and Context-Aware Mixins: Q&amp;A</a></td><td><a href="../lessons/021-custom-functions-and-return-values.md">Lesson</a></td><td><a href="../examples/021-custom-functions-and-return-values.md">Example</a></td><td><a href="022-built-in-sass-modules.md">022 - Built-in Sass Modules: Q&amp;A</a></td></tr>
  </tbody>
</table>
