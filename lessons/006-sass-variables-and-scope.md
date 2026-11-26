# 006. Sass Variables and Scope

## Learning goals

After this lesson, you should be able to:

- Declare and use Sass variables in SCSS.
- Explain when a Sass variable is evaluated and how reassignment works.
- Distinguish a Sass variable from a CSS custom property.
- Describe top-level, local, and module variable scope.
- Recognize shadowing and the special scope behavior of flow-control rules.
- Use !default for fallback values and explain the limited role of !global.
- Configure a module variable through @use when a library exposes a default.

## Declaring Sass variables

A Sass variable starts with a dollar sign. Write a colon between its name and value, and end the statement with a semicolon:

~~~scss
$brand-color: #1769aa;
$space: 1rem;

.button {
  color: white;
  background-color: $brand-color;
  padding: $space;
}
~~~

During compilation, Sass substitutes the variable values into declarations. The browser receives CSS values; it does not receive the Sass variables.

~~~css
.button {
  color: white;
  background-color: #1769aa;
  padding: 1rem;
}
~~~

Variables can hold values that are valid in Sass, including numbers, strings, colors, lists, maps, booleans, and null. Those value types have their own chapters. At this stage, focus on assigning a useful name to a value and reusing it.

## Choose meaningful names

Name variables for the role of a value, not only its current appearance:

~~~scss
$brand-color: #1769aa;
$surface-color: #f4f7fb;
$content-width: 68rem;
$control-radius: 0.5rem;
~~~

Names such as $brand-color or $content-width explain why the value exists. A name like $blue may become misleading if the design changes to a different color.

Sass identifiers treat hyphens and underscores as equivalent. For example, $font-size and $font_size refer to the same Sass variable. Choose one naming convention and use it consistently.

## Sass variables are compile-time values

Sass variables are imperative: each use gets the value that is assigned at that point in the source. Reassigning a variable later affects later uses, not earlier output:

~~~scss
$space: 0.5rem;

.compact {
  padding: $space;
}

$space: 1rem;

.roomy {
  padding: $space;
}
~~~

The compiled CSS is:

~~~css
.compact {
  padding: 0.5rem;
}
.roomy {
  padding: 1rem;
}
~~~

After compilation, changing the Sass source and rebuilding is required to produce different CSS. Sass variables do not change in the browser at runtime.

## Sass variables versus CSS custom properties

Both can store reusable values, but they operate at different times:

| Sass variable | CSS custom property |
| --- | --- |
| Starts with a dollar sign, such as $brand. | Starts with two hyphens, such as --brand. |
| Evaluated by Sass during compilation. | Preserved in CSS for the browser. |
| Does not exist in the generated CSS as a Sass variable. | Can be read by the cascade and JavaScript at runtime. |
| Reassignment affects later Sass output according to source order. | The browser resolves the value through inheritance and the cascade. |

Example Sass variable:

~~~scss
$brand: #1769aa;

.button {
  background-color: $brand;
}
~~~

Example CSS custom property:

~~~scss
:root {
  --brand: #1769aa;
}

.button {
  background-color: var(--brand);
}
~~~

A CSS custom property can be changed for a subtree, theme, media query, or at runtime. A Sass variable is useful when a value should be computed or reused during compilation.

### Putting a Sass value in a custom property

Sass treats the value of a CSS custom property as CSS text. Use interpolation to insert a Sass value:

~~~scss
$brand: #1769aa;

:root {
  --brand: #{$brand};
}
~~~

The output is:

~~~css
:root {
  --brand: #1769aa;
}
~~~

Without interpolation, the Sass-looking text in the custom property is passed through literally:

~~~scss
:root {
  --brand: $brand;
}
~~~

That output contains the text $brand; Sass does not replace it. Use interpolation when a SassScript value must be inserted into a custom property value.

## Top-level scope

A variable declared at the top level of a stylesheet is available later in that stylesheet’s module:

~~~scss
$brand: #1769aa;

.header {
  background-color: $brand;
}

.footer {
  border-color: $brand;
}
~~~

With the module system, a stylesheet loaded with @use has its own scope. Its members are accessed through the module namespace unless the author deliberately changes that behavior:

~~~scss
@use "settings";

.button {
  color: settings.$brand;
}
~~~

A variable defined by settings is not automatically copied into the calling stylesheet’s global scope. The namespace makes the value’s source clear. Module loading and forwarding are covered in their own lessons.

## Local scope and shadowing

A variable declared inside a style rule is local to that block:

~~~scss
$color: navy;

.card {
  $color: teal;
  color: $color;
}

.footer {
  color: $color;
}
~~~

The local $color inside .card shadows the top-level $color. It produces teal for .card, while .footer still uses navy. The local assignment does not overwrite the top-level variable.

A variable that exists only inside a block cannot be used after the block:

~~~scss
.card {
  $local-space: 1rem;
  padding: $local-space;
}

// $local-space is not available here.
~~~

Keeping values local can prevent implementation details from becoming accidental shared settings.

## Flow-control scope

Variables inside flow-control rules such as @if, @each, @for, and @while have special behavior. An assignment to a variable that already exists in the surrounding scope updates that variable; a new variable created only inside the flow-control block does not become available outside it.

~~~scss
$status: ready;

@if true {
  $status: active;
}

.status {
  content: $status;
}
~~~

The output uses active for the .status rule.

If a variable is created only inside the @if block, do not expect it to be visible afterward:

~~~scss
@if true {
  $temporary: active;
}

// $temporary is not available here.
~~~

For beginner code, declare a variable in the scope where it is intended to be used. This makes the data flow easier to follow.

## The !default flag

The !default flag assigns a value only when the variable is not already set or is set to null:

~~~scss
$brand: #1769aa !default;
$radius: 0.5rem !default;
~~~

This is useful when a stylesheet or library provides defaults that a caller may configure before those values are used. If the caller has already set a non-null value, the default assignment leaves that value unchanged.

Without !default, assigning a value to an existing variable replaces its current value.

## Configuring a module with defaults

A module can expose top-level variables with !default so a caller can configure them when loading the module.

A library file named _theme.scss:

~~~scss
$brand: #1769aa !default;
$radius: 0.5rem !default;

.button {
  background-color: $brand;
  border-radius: $radius;
}
~~~

A stylesheet that configures it:

~~~scss
@use "theme" with (
  $brand: #7c3aed,
  $radius: 0.75rem
);
~~~

The caller supplies the values before the module is loaded. The generated button uses the configured values.

Only top-level variables marked !default are configurable this way. Configure a module when it is loaded; do not expect to reload it later with a different configuration in the same compilation. Detailed module organization and configuration patterns appear in the module chapters.

## The !global flag

The !global flag changes the assignment target from the current local scope to a top-level variable in the current stylesheet:

~~~scss
$counter: 0;

.item {
  $counter: 1 !global;
}
~~~

Use !global sparingly. Shared mutable state makes it harder to understand why a later rule has a particular value. In most application stylesheets, a clear top-level setting, a local variable, or a configured module is easier to maintain.

A global assignment should target a variable already declared at the top level. Do not use !global as a way to invent undeclared global names.

## Variables and naming collisions

Treat a variable as part of a stylesheet’s internal API when other stylesheets or callers depend on it. Keep names scoped to their module where possible, and choose names that convey purpose.

Avoid declaring a local variable with the same name as an outer variable unless shadowing is intentional. While Sass supports it, the two values can be confusing during debugging.

Because hyphens and underscores are equivalent in Sass identifiers, do not use both spellings to mean separate variables. They refer to the same name.

## Common mistakes

### Forgetting the dollar sign

Sass variable declarations and references begin with $:

~~~scss
$brand: #1769aa;

.link {
  color: $brand;
}
~~~

A bare name such as brand is not a Sass variable.

### Expecting a Sass variable to be available in browser tools

Sass removes its variables during compilation. Inspect the generated CSS value instead. Use a CSS custom property if the value must remain available at runtime.

### Using a local variable outside its scope

Move the declaration to a scope that contains every use, or pass the value through the relevant Sass module, mixin, or function design.

### Expecting a reassignment to change earlier output

Sass uses the value at the point where it encounters each use. A later assignment changes later output, not CSS already emitted earlier in the source order.

### Forgetting interpolation in a custom property value

Custom property values are passed through as CSS text. Write --brand: #{$brand}; when the Sass variable must be substituted.

### Using !default everywhere

!default is for fallback values that callers may configure. Ordinary variables should use normal assignment unless a defaulting behavior is intended.

## Key takeaways

- Sass variables begin with $ and are resolved during compilation.
- Name variables by purpose and use one consistent hyphen or underscore convention.
- Top-level variables are visible within their module; variables inside blocks are usually local.
- Local variables can shadow outer variables without replacing them.
- Flow-control rules have special assignment behavior for variables from the surrounding scope.
- !default preserves an existing non-null value and supports configurable defaults.
- CSS custom properties remain in output and support runtime cascade behavior.
- Use interpolation to insert Sass values into custom property values.
- Use !global rarely; prefer explicit scope and module configuration.

## Practice

1. Create color and spacing variables and use them in two CSS rules.
2. Reassign a variable between two rules and compare the compiled values.
3. Create a local variable inside a selector and try to reference it outside that selector.
4. Shadow a top-level variable in a nested block and inspect both outputs.
5. Assign to a previously declared variable inside an @if block and inspect the final value.
6. Add !default to a variable and set it before the default assignment.
7. Configure a module’s top-level !default variable with @use ... with (...).
8. Compare a Sass variable with a CSS custom property in the generated CSS.

## Further reading

- [Sass variables and scope](https://sass-lang.com/documentation/variables/)
- [Sass module loading with @use](https://sass-lang.com/documentation/at-rules/use/)
- [Property declarations and CSS custom properties](https://sass-lang.com/documentation/style-rules/declarations/)
- [Sass interpolation](https://sass-lang.com/documentation/interpolation/)
