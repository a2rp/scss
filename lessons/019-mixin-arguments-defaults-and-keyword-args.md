# 019 - Mixin Arguments, Defaults, and Keyword Arguments

## Learning goals

After this lesson, you should be able to:

- Declare mixin parameters and pass values when including the mixin.
- Match positional arguments to parameters in order.
- Give parameters default values so callers can omit them.
- Override defaults with keyword arguments.
- Use keyword arguments to make mixin calls easier to read.
- Understand how default values can refer to earlier parameters.
- Recognize errors caused by missing, unknown, duplicated, or excess arguments.
- Treat parameter names as part of a mixin's public API.

## 1. Why mixins take arguments

An argument lets a mixin adjust its output for each include. Without arguments, every include receives the same fixed declarations. With arguments, a shared pattern can be reused with different values.

~~~scss
@mixin text-color($color) {
  color: $color;
}

.heading {
  @include text-color(#202124);
}

.caption {
  @include text-color(#667085);
}
~~~

Generated CSS:

~~~css
.heading {
  color: #202124;
}
.caption {
  color: #667085;
}
~~~

The value passed in an @include is available in the mixin body through the matching parameter variable.

## 2. Declare parameters

Write the parameter names in parentheses after the mixin name. Each parameter begins with a dollar sign.

~~~scss
@mixin size($width, $height) {
  width: $width;
  height: $height;
}
~~~

The mixin requires two values:

~~~scss
.avatar {
  @include size(3rem, 3rem);
}
~~~

The first positional argument is assigned to $width; the second is assigned to $height.

Generated CSS:

~~~css
.avatar {
  width: 3rem;
  height: 3rem;
}
~~~

Parameter values are SassScript expressions. They can be numbers with units, colors, strings, calculations, variables, or other Sass values that are appropriate for the declarations.

## 3. Positional arguments

Positional arguments are assigned by their order in the include. Use them when the parameter order is obvious and the call stays easy to read.

~~~scss
@mixin inset($block, $inline) {
  padding-block: $block;
  padding-inline: $inline;
}

.card {
  @include inset(1rem, 1.5rem);
}
~~~

Generated CSS:

~~~css
.card {
  padding-block: 1rem;
  padding-inline: 1.5rem;
}
~~~

Positional calls can become hard to scan when a mixin has several values of the same type. In that situation, keyword arguments make the intent clearer.

## 4. Required and optional parameters

A parameter without a default is required. The caller must supply a value for every required parameter.

~~~scss
@mixin border-style($color, $width) {
  border: $width solid $color;
}

.panel {
  @include border-style(#d0d5dd, 1px);
}
~~~

A parameter with a default is optional. Its default is used when the caller omits that argument.

~~~scss
@mixin border-style($color, $width: 1px) {
  border: $width solid $color;
}

.panel {
  @include border-style(#d0d5dd);
}

.dialog {
  @include border-style(#98a2b3, 2px);
}
~~~

Generated CSS:

~~~css
.panel {
  border: 1px solid #d0d5dd;
}
.dialog {
  border: 2px solid #98a2b3;
}
~~~

In this example, $color is required. $width defaults to 1px, but the dialog passes a different width.

## 5. Put required parameters before optional ones

A clear parameter list places required parameters before parameters with defaults.

~~~scss
@mixin transition($property, $duration: 180ms, $easing: ease-out) {
  transition-property: $property;
  transition-duration: $duration;
  transition-timing-function: $easing;
}
~~~

The property is required; duration and easing have defaults. This makes common calls short while leaving less common choices available.

~~~scss
.dialog {
  @include transition(opacity);
}

.drawer {
  @include transition(transform, 240ms, cubic-bezier(0.2, 0.8, 0.2, 1));
}
~~~

Generated CSS:

~~~css
.dialog {
  transition-property: opacity;
  transition-duration: 180ms;
  transition-timing-function: ease-out;
}
.drawer {
  transition-property: transform;
  transition-duration: 240ms;
  transition-timing-function: cubic-bezier(0.2, 0.8, 0.2, 1);
}
~~~

The first include uses both defaults. The second overrides both.

## 6. Use keyword arguments

A keyword argument specifies the parameter name as well as its value. It is especially helpful when a mixin has multiple optional parameters.

~~~scss
@mixin shadow($x, $y, $blur, $color) {
  box-shadow: $x $y $blur $color;
}

.card {
  @include shadow(
    $x: 0,
    $y: 0.25rem,
    $blur: 0.75rem,
    $color: rgb(16 24 40 / 12%)
  );
}
~~~

Generated CSS:

~~~css
.card {
  box-shadow: 0 0.25rem 0.75rem rgb(16 24 40 / 12%);
}
~~~

Keyword arguments make the relationship between values and parameters explicit. Sass allows keyword arguments to be supplied in a different order from the parameter list.

## 7. Combine positional and keyword arguments

You can supply the first arguments positionally and then name later arguments.

~~~scss
@mixin button($background, $foreground, $radius: 0.5rem) {
  color: $foreground;
  background-color: $background;
  border-radius: $radius;
}

.primary-button {
  @include button(#175cd3, white);
}

.pill-button {
  @include button(#175cd3, white, $radius: 999px);
}
~~~

Generated CSS:

~~~css
.primary-button {
  color: white;
  background-color: #175cd3;
  border-radius: 0.5rem;
}
.pill-button {
  color: white;
  background-color: #175cd3;
  border-radius: 999px;
}
~~~

Positional arguments must appear before keyword arguments in a call. Once an argument is named, keep following supplied arguments named as well.

## 8. Use a keyword to skip an optional parameter

Keyword arguments let the caller override a later default without supplying every optional value before it.

~~~scss
@mixin button($background, $foreground: white, $radius: 0.5rem) {
  color: $foreground;
  background-color: $background;
  border-radius: $radius;
}

.icon-button {
  @include button(#175cd3, $radius: 50%);
}
~~~

Generated CSS:

~~~css
.icon-button {
  color: white;
  background-color: #175cd3;
  border-radius: 50%;
}
~~~

The foreground keeps its default, while the radius is explicitly set. This is clearer than passing a placeholder value just to reach a later parameter.

## 9. Defaults can refer to earlier parameters

A default expression can use a parameter that appears earlier in the parameter list.

~~~scss
@mixin outline-button($background, $foreground: white, $border: $background) {
  color: $foreground;
  background-color: $background;
  border: 1px solid $border;
}

.primary-button {
  @include outline-button(#175cd3);
}

.secondary-button {
  @include outline-button(
    white,
    $foreground: #175cd3
  );
}
~~~

Generated CSS:

~~~css
.primary-button {
  color: white;
  background-color: #175cd3;
  border: 1px solid #175cd3;
}
.secondary-button {
  color: #175cd3;
  background-color: white;
  border: 1px solid white;
}
~~~

The default for $border uses the earlier $background value. In the second include, $foreground is named and $border still receives its own default.

## 10. Choose clear parameter names

Parameter names are visible to anyone using the mixin with keyword arguments. Names should describe the value's role.

Less clear:

~~~scss
@mixin card($a, $b, $c) {
  color: $a;
  background-color: $b;
  border-radius: $c;
}
~~~

Clearer:

~~~scss
@mixin card($foreground, $background, $radius) {
  color: $foreground;
  background-color: $background;
  border-radius: $radius;
}
~~~

Named arguments such as $background: white are only helpful when the name explains the meaning.

## 11. Validate the values a mixin accepts

Sass does not infer a CSS design rule from a parameter name. A mixin can emit inappropriate CSS if it receives an inappropriate value.

~~~scss
@mixin width($value) {
  width: $value;
}
~~~

The caller must still pass a value that makes sense for width. When a mixin is intended as a library API, later lessons on conditionals, functions, and warnings can help validate inputs.

For project-local mixins, clear parameter names, defaults, and examples often prevent misuse without building a complicated validation system.

## 12. Parameter names are part of the API

Positional callers depend on argument order. Keyword callers depend on argument names. Changing a public parameter name can therefore break consumers even if the mixin still compiles internally.

Before changing a shared mixin:

- Search for positional and keyword calls.
- Check whether other files or projects use the mixin.
- Keep parameter names stable when callers rely on them.
- If a migration is needed, provide a clear transition rather than silently changing the API.

For private one-file mixins, renaming is simpler, but the include sites still need updating.

## 13. Common argument errors

### Missing required argument

If a required parameter is not supplied and has no default, Sass reports an argument error.

### Too many arguments

If a call supplies more positional arguments than the mixin declares, Sass reports an argument error unless the mixin explicitly accepts arbitrary arguments. Variadic arguments are an advanced topic.

### Unknown keyword

A keyword name must match a declared parameter. Sass reports an error for an unknown name.

### Duplicate argument

Do not provide the same parameter twice, such as once positionally and again by name. Sass reports the argument as passed more than once.

### Positional argument after a keyword

Put positional arguments before keyword arguments. Use names for subsequent values once keyword arguments begin.

## 14. A practical responsive-container mixin

~~~scss
@mixin container(
  $max-width: 72rem,
  $gutter: 1.5rem,
  $center: true
) {
  width: min(100% - 2 * $gutter, $max-width);

  @if $center {
    margin-inline: auto;
  }
}

.page-container {
  @include container;
}

.article-container {
  @include container(
    $max-width: 48rem,
    $gutter: 1rem
  );
}

.flush-container {
  @include container(
    $max-width: 80rem,
    $gutter: 0,
    $center: false
  );
}
~~~

This example previews a mixin whose behavior depends on a Boolean argument. Conditional logic is covered in a later chapter; here, focus on how named arguments make each include readable.

## 15. Exercises

1. Write a mixin with required $color and optional $width defaulting to 1px.
2. Call it once using positional arguments and once using a keyword argument.
3. Write a transition mixin with defaults for duration and easing.
4. Override only the easing value by keyword while keeping the duration default.
5. Create a button mixin with $background, $foreground, and $radius parameters.
6. Use a keyword argument to change the radius while leaving the foreground default.
7. Make one default refer to an earlier parameter and predict the generated CSS.
8. Deliberately misspell a keyword argument and observe the Sass error.
9. Rename a parameter used by keyword at an include site and explain why both sides must be updated.
10. For each mixin call, label which values are positional, defaulted, or keyword arguments.

## Summary

- Declare mixin parameters in parentheses and pass their values through @include.
- Positional arguments match parameters from left to right.
- Parameters with defaults are optional; required arguments should come first for clarity.
- Keyword arguments identify parameters by name and can override a later default.
- A default can refer to an earlier parameter.
- Keyword parameter names are part of a mixin's public API.
- Sass reports errors for missing required values, unknown keywords, duplicate values, and excess arguments.
- Content blocks and arbitrary argument lists are separate topics.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="018-mixin-basics.md">018 - Mixin Basics</a></td><td><a href="../qnas/019-mixin-arguments-defaults-and-keyword-args.md">Q&amp;A</a></td><td><a href="../examples/019-mixin-arguments-defaults-and-keyword-args.md">Example</a></td><td><a href="020-content-blocks-and-context-aware-mixins.md">020 - Content Blocks and Context-Aware Mixins</a></td></tr>
  </tbody>
</table>
