# 006. Sass Variables and Scope — Examples

These examples use the project-local Sass CLI from the repository root. They show how Sass variables are declared, resolved, scoped, and configured.

## Example 1: Reuse and reassign values

Create src/variables.scss:

~~~scss
$brand-color: #1769aa;
$surface-color: #f4f7fb;
$space: 1rem;

.card {
  color: #243447;
  background-color: $surface-color;
  padding: $space;
  border-top: 0.25rem solid $brand-color;
}

.card__title {
  color: $brand-color;
}

$space: 1.5rem;

.card__content {
  padding: $space;
}
~~~

Compile the file:

~~~sh
npx sass src/variables.scss dist/variables.css
~~~

The earlier .card uses the first $space value. The later .card__content uses the reassigned value:

~~~css
.card {
  color: #243447;
  background-color: #f4f7fb;
  padding: 1rem;
  border-top: 0.25rem solid #1769aa;
}
.card__title {
  color: #1769aa;
}
.card__content {
  padding: 1.5rem;
}
~~~

The Sass variables themselves do not appear in the generated CSS.

## Example 2: Local scope and shadowing

Create src/scope.scss:

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

Compile it:

~~~sh
npx sass src/scope.scss dist/scope.css
~~~

The local $color shadows the top-level variable inside .card. The assignment does not replace the top-level value:

~~~css
.card {
  color: teal;
}
.footer {
  color: navy;
}
~~~

A variable declared only inside a block is not available after that block:

~~~scss
.card {
  $local-space: 1rem;
  padding: $local-space;
}

// This would fail because $local-space is outside its scope.
.footer {
  padding: $local-space;
}
~~~

Remove the invalid .footer declaration to compile the example successfully.

## Example 3: Flow-control assignment

Create src/flow-scope.scss:

~~~scss
$status: #1769aa;

@if true {
  $status: #7c3aed;
}

.status {
  color: $status;
}
~~~

Compile the file:

~~~sh
npx sass src/flow-scope.scss dist/flow-scope.css
~~~

The variable was declared before the flow-control block. The assignment inside @if updates that existing variable, so the output is:

~~~css
.status {
  color: #7c3aed;
}
~~~

A variable declared only inside the @if block is not available outside it:

~~~scss
@if true {
  $temporary-status: active;
}

// A later use of $temporary-status would be undefined.
~~~

## Example 4: Sass variable and CSS custom property

Create src/runtime-theme.scss:

~~~scss
$brand: #1769aa;

:root {
  --brand: #{$brand};
}

.button {
  color: white;
  background-color: var(--brand);
}
~~~

Compile it:

~~~sh
npx sass src/runtime-theme.scss dist/runtime-theme.css
~~~

The output keeps the CSS custom property and resolves the Sass variable into it:

~~~css
:root {
  --brand: #1769aa;
}
.button {
  color: white;
  background-color: var(--brand);
}
~~~

The value of --brand can now participate in the browser’s CSS cascade. The Sass variable $brand was used only during compilation.

Compare with this incorrect substitution attempt:

~~~scss
$brand: #1769aa;

:root {
  --brand: $brand;
}
~~~

Sass treats the custom-property value as CSS text, so the output contains the literal text $brand. Use interpolation when inserting a Sass value there.

## Example 5: Apply a default only when no value exists

Create src/defaults.scss:

~~~scss
$brand: #7c3aed;
$brand: #1769aa !default;

.button {
  background-color: $brand;
}
~~~

Compile it:

~~~sh
npx sass src/defaults.scss dist/defaults.css
~~~

The existing non-null value is preserved:

~~~css
.button {
  background-color: #7c3aed;
}
~~~

If the first assignment is removed, the !default assignment provides #1769aa. If the variable has been set to null, !default also provides its fallback.

## Example 6: Configure a module’s defaults

Create src/_theme.scss:

~~~scss
$brand: #1769aa !default;
$radius: 0.5rem !default;

.button {
  color: white;
  background-color: $brand;
  border-radius: $radius;
}
~~~

Create src/configured-theme.scss:

~~~scss
@use "theme" with (
  $brand: #7c3aed,
  $radius: 0.75rem
);
~~~

Compile the configured entrypoint:

~~~sh
npx sass src/configured-theme.scss dist/configured-theme.css
~~~

The module receives the configuration when it is first loaded. The output is:

~~~css
.button {
  color: white;
  background-color: #7c3aed;
  border-radius: 0.75rem;
}
~~~

Only the module’s top-level variables marked !default can be configured this way. The file beginning with an underscore is a dependency, while configured-theme.scss is the entrypoint.

## Example 7: Hyphen and underscore names

Sass treats these spellings as the same identifier:

~~~scss
$font-size: 1.125rem;

.heading {
  font-size: $font_size;
}
~~~

This compiles successfully and uses the value from $font-size. Choose one spelling in a project to avoid making the code look as if it has two separate settings.

## Example 8: Understand !global without relying on it

A small demonstration shows that !global targets an existing top-level variable:

~~~scss
$state: #1769aa;

.item {
  $state: #7c3aed !global;
}

.status {
  color: $state;
}
~~~

The .status rule uses #7c3aed because the local assignment changed the top-level $state.

This pattern creates shared mutable state and is usually harder to maintain. Prefer assigning the value in the intended scope or configuring a module.

## Suggested project layout

A project using these examples could look like:

~~~text
sass-variable-demo/
├── src/
│   ├── _theme.scss
│   ├── configured-theme.scss
│   ├── defaults.scss
│   ├── flow-scope.scss
│   ├── runtime-theme.scss
│   ├── scope.scss
│   └── variables.scss
└── dist/
    ├── configured-theme.css
    ├── defaults.css
    ├── flow-scope.css
    ├── runtime-theme.css
    ├── scope.css
    └── variables.css
~~~

Each source file is an independent demonstration. Compile the one you want to inspect, or compile the directory while keeping underscore-prefixed dependencies as partials. Sass also writes .css.map files by default; they are omitted from this abbreviated tree.

## Try these changes

1. Change $brand-color and inspect every rule that uses it.
2. Move the $space reassignment above .card and compare the compiled result.
3. Rename the local $color variable and observe which scope each rule uses.
4. Change the configured module values and compile the entrypoint again.
5. Replace the CSS custom property interpolation with a literal value, then compare.
6. Add !default to a variable whose existing value is null and inspect the result.
7. Remove !global from its demonstration and compare the top-level value.
