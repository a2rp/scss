# 019 — Mixin Arguments, Defaults, and Keyword Arguments: Examples

Each example shows how arguments affect the CSS emitted by a Sass mixin.

## 1. Pass required positional arguments

Source SCSS:

~~~scss
@mixin size($width, $height) {
  width: $width;
  height: $height;
}

.avatar {
  @include size(3rem, 3rem);
}
~~~

Generated CSS:

~~~css
.avatar {
  width: 3rem;
  height: 3rem;
}
~~~

The first argument maps to $width and the second maps to $height.

## 2. Override one default

Source SCSS:

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

The panel uses the default width; the dialog supplies a new width positionally.

## 3. Use several defaults

Source SCSS:

~~~scss
@mixin transition($property, $duration: 180ms, $easing: ease-out) {
  transition-property: $property;
  transition-duration: $duration;
  transition-timing-function: $easing;
}

.dialog {
  @include transition(opacity);
}

.drawer {
  @include transition(transform, 240ms, linear);
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
  transition-timing-function: linear;
}
~~~

The dialog omits both optional arguments. The drawer overrides both.

## 4. Supply keyword arguments in a different order

Source SCSS:

~~~scss
@mixin shadow($x, $y, $blur, $color) {
  box-shadow: $x $y $blur $color;
}

.card {
  @include shadow(
    $color: rgb(16 24 40 / 12%),
    $blur: 0.75rem,
    $y: 0.25rem,
    $x: 0
  );
}
~~~

Generated CSS:

~~~css
.card {
  box-shadow: 0 0.25rem 0.75rem rgb(16 24 40 / 12%);
}
~~~

Because each argument is named, its order in the include does not have to match the definition.

## 5. Combine positional and keyword arguments

Source SCSS:

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

The required values are positional. The optional radius is overridden by keyword in the pill button.

## 6. Skip a defaulted parameter

Source SCSS:

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

The foreground keeps its default while the later radius is named explicitly.

## 7. Let a default refer to an earlier parameter

Source SCSS:

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
  @include outline-button(white, $foreground: #175cd3);
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

The default border color follows the earlier background parameter. The secondary include overrides only the foreground.

## 8. Use a default expression

Source SCSS:

~~~scss
@mixin spacing($size, $wide: $size * 1.5) {
  padding-block: $size;
  padding-inline: $wide;
}

.card {
  @include spacing(1rem);
}

.feature-card {
  @include spacing(1rem, 2rem);
}
~~~

Generated CSS:

~~~css
.card {
  padding-block: 1rem;
  padding-inline: 1.5rem;
}
.feature-card {
  padding-block: 1rem;
  padding-inline: 2rem;
}
~~~

The default is an expression evaluated from the earlier argument. The feature-card include supplies a different value.

## 9. Use meaningful keyword names

Source SCSS:

~~~scss
@mixin surface(
  $background,
  $foreground: #202124,
  $border-color: #d0d5dd,
  $radius: 0.75rem
) {
  color: $foreground;
  background-color: $background;
  border: 1px solid $border-color;
  border-radius: $radius;
}

.notice {
  @include surface(
    $background: #eff8ff,
    $foreground: #1849a9,
    $border-color: #b2ddff
  );
}
~~~

Generated CSS:

~~~css
.notice {
  color: #1849a9;
  background-color: #eff8ff;
  border: 1px solid #b2ddff;
  border-radius: 0.75rem;
}
~~~

The keyword names communicate which design values are being changed. The radius keeps its default.

## 10. Use a Boolean parameter with a conditional

Source SCSS:

~~~scss
@mixin container($max-width: 72rem, $gutter: 1.5rem, $center: true) {
  width: min(100% - 2 * $gutter, $max-width);

  @if $center {
    margin-inline: auto;
  }
}

.page-container {
  @include container;
}

.flush-container {
  @include container(
    $max-width: 80rem,
    $gutter: 0,
    $center: false
  );
}
~~~

Generated CSS:

~~~css
.page-container {
  width: min(100% - 3rem, 72rem);
  margin-inline: auto;
}
.flush-container {
  width: min(100%, 80rem);
}
~~~

This previews conditional logic: a Boolean argument can control which declarations are emitted. Conditional rules are explained in a later chapter.

## 11. See an argument error

Mixin definition:

~~~scss
@mixin border-style($color, $width: 1px) {
  border: $width solid $color;
}
~~~

This include has an unknown keyword:

~~~scss
.panel {
  @include border-style(#d0d5dd, $thickness: 2px);
}
~~~

The parameter is named $width, not $thickness. Sass reports an unknown argument instead of silently choosing a value.

## 12. See a duplicate-argument error

Mixin definition:

~~~scss
@mixin size($width, $height) {
  width: $width;
  height: $height;
}
~~~

This call supplies $width twice:

~~~scss
.avatar {
  @include size(3rem, $width: 4rem);
}
~~~

Sass reports that the width argument was passed more than once. Use either a positional or keyword value for a parameter, not both.

## 13. Compile and review

Save the examples to src/mixin-arguments.scss and compile them:

~~~sh
sass src/mixin-arguments.scss dist/mixin-arguments.css
~~~

For each include, identify:

- Which values are positional.
- Which values are keyword arguments.
- Which parameters use defaults.
- Whether any default refers to an earlier parameter.
- What declarations Sass emits.

## 14. Practice tasks

1. Define a size mixin with required width and height arguments.
2. Add a defaulted border width and override it once positionally.
3. Add duration and easing defaults to a transition mixin.
4. Override only easing with a keyword argument.
5. Create a button mixin with background, foreground, and radius values.
6. Use a keyword to change radius while leaving foreground at its default.
7. Make a default value depend on an earlier parameter.
8. Write one call with keyword arguments in a different order from the definition.
9. Try an unknown keyword and read the compiler error.
10. Find a mixin include and label each argument as positional, keyword, or defaulted.
