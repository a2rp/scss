# 010 - Strings and Interpolation: Examples

These examples focus on Sass strings and interpolation. Each section is an independent snippet unless it says otherwise. In a real stylesheet, keep a single sass:string module load at the top of the file.

## 1. Quoted text and unquoted CSS keywords

Quoted strings are useful for text. Unquoted strings are useful when the result should be CSS syntax such as a keyword or identifier.

~~~scss
$label: "red";
$family: sans-serif;
$weight: bold;

.label::before {
  content: $label;
}

.text {
  font-family: $family;
  font-weight: $weight;
}
~~~

Compiled CSS:

~~~css
.label::before {
  content: "red";
}

.text {
  font-family: sans-serif;
  font-weight: bold;
}
~~~

Even though red is a CSS color name, it remains text here because it is quoted.

## 2. Escaping characters in quoted strings

Escape a quote that matches the surrounding quote style. Escape a backslash when it must appear literally.

~~~scss
$quote: "She said, \"Hello!\"";
$path: "C:\\assets\\icons\\arrow.svg";

.quote::before {
  content: $quote;
}

.path-note::before {
  content: $path;
}
~~~

The serialized CSS keeps the text quoted and escapes characters as needed.

## 3. Interpolation inside a quoted string

Interpolation inserts a Sass value into a larger text string. The outer quotes remain around the final CSS string.

~~~scss
$theme: "ocean";
$version: 2;

.theme-note::before {
  content: "Theme: #{$theme}, version: #{$version}";
}
~~~

Compiled CSS:

~~~css
.theme-note::before {
  content: "Theme: ocean, version: 2";
}
~~~

## 4. Building selectors from controlled values

Use interpolation when a variable supplies part of a selector. Keep the value controlled so the generated selector is predictable.

~~~scss
$component: "card";
$state: "selected";

.#{$component}.is-#{$state} {
  outline: 2px solid royalblue;
}
~~~

Compiled CSS:

~~~css
.card.is-selected {
  outline: 2px solid royalblue;
}
~~~

A list can generate a small, consistent group of state selectors:

~~~scss
$states: success, warning, error;

@each $state in $states {
  .notice-#{$state} {
    border-inline-start: 0.25rem solid currentColor;
  }
}
~~~

This emits .notice-success, .notice-warning, and .notice-error rules.

## 5. Building a property name

Interpolation can place a Sass value into a declaration's property name.

~~~scss
$side: "left";

.panel {
  border-#{$side}-width: 0.25rem;
}
~~~

Compiled CSS:

~~~css
.panel {
  border-left-width: 0.25rem;
}
~~~

Only use known, valid property fragments. Unexpected text can produce an invalid declaration.

## 6. Sass values in custom properties

Custom property values are preserved for the browser. Interpolate Sass values that need to be inserted into them.

~~~scss
$accent: #2864dc;
$label: "Account settings";

:root {
  --accent-color: #{$accent};
  --panel-label: "#{$label}";
}
~~~

Compiled CSS:

~~~css
:root {
  --accent-color: #2864dc;
  --panel-label: "Account settings";
}
~~~

For an ordinary property, use the Sass variable directly:

~~~scss
.button {
  color: $accent;
}
~~~

There is no need to interpolate the value of a regular color declaration.

## 7. Dynamic keyframe names

A generated identifier can be shared between an animation declaration and its keyframes.

~~~scss
$animation-name: fade-in;

.panel {
  animation: $animation-name 180ms ease-out both;
}

@keyframes #{$animation-name} {
  from {
    opacity: 0;
    transform: translateY(0.5rem);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}
~~~

The animation name remains an identifier. Keeping it as an unquoted Sass value makes its CSS role clear.

## 8. Interpolation in a URL

Insert a filename or folder segment into a URL.

~~~scss
$icon-name: "search";

.search-button {
  background-image: url("/assets/icons/#{$icon-name}.svg");
}
~~~

Compiled CSS:

~~~css
.search-button {
  background-image: url("/assets/icons/search.svg");
}
~~~

Use known asset names. Interpolation does not sanitize arbitrary input into a safe URL.

## 9. Interpolation in an at-rule parameter

A Sass value can supply part of an at-rule parameter.

~~~scss
$wide-layout: 48rem;

@media (min-width: #{$wide-layout}) {
  .layout {
    grid-template-columns: 16rem 1fr;
  }
}
~~~

Compiled CSS:

~~~css
@media (min-width: 48rem) {
  .layout {
    grid-template-columns: 16rem 1fr;
  }
}
~~~

## 10. Quoting and unquoting with sass:string

Load the built-in string module with @use. quote() returns quoted text; unquote() returns an unquoted string.

~~~scss
@use "sass:string";

$quoted-keyword: string.quote(alert);
$css-family: string.unquote("sans-serif");

.message::before {
  content: $quoted-keyword;
}

.text {
  font-family: $css-family;
}
~~~

The first value is appropriate for content text. The second is used as a CSS font-family keyword. Only unquote values when their unquoted form is valid in that CSS position.

## 11. Inspecting and transforming strings

String indices start at 1. string.slice() includes both the start and end indices; a negative index counts backward from the end.

~~~scss
@use "sass:string";

$font: "Helvetica Neue";
$position: string.index($font, "Neue");
$family: string.slice($font, 1, 9);
$mono-font: string.insert($font, " Mono", 7);
$upper: string.to-upper-case("button");

.result::before {
  content: "position=#{$position}; family=#{$family}; mono=#{$mono-font}; upper=#{$upper}";
}
~~~

The intermediate values are:

- $position is 11.
- $family is Helvetica.
- $mono-font is Helvetica Mono Neue.
- $upper is BUTTON.

## 12. Finding a substring that may not exist

string.index() returns null when the search text is absent. Check the result before using it in calculations or generated CSS.

~~~scss
@use "sass:string";

$filename: "logo-dark.svg";
$separator: string.index($filename, "-");

@if $separator {
  .filename::before {
    content: string.slice($filename, 1, $separator - 1);
  }
}
~~~

This emits a pseudo-element whose content is logo. If the filename has no hyphen, the rule is not emitted.

## 13. Splitting a string for inspection

string.split() returns a Sass list. Use @debug to inspect it during compilation; debug output is not part of the CSS file.

~~~scss
@use "sass:string";

$parts: string.split("ui-button-primary", "-");

@debug $parts;
~~~

The debug output is a bracketed list containing "ui", "button", and "primary".

## 14. Creating a unique CSS identifier

string.unique-id() generates a valid identifier that is unique within the current Sass compilation. Its exact text changes, so do not depend on a particular suffix.

~~~scss
@use "sass:string";

$clip-id: string.unique-id();

.clip-#{$clip-id} {
  overflow: hidden;
}
~~~

The compiled selector will have a generated suffix such as .clip-u1abc23. The actual suffix is not predictable.

## 15. Do not turn numbers into text to create units

Avoid constructing a number and unit with interpolation. That produces text and removes Sass's ability to treat the result as a number.

~~~scss
$size: 2;

// Avoid this pattern:
// .box { width: #{$size}rem; }

.box {
  width: $size * 1rem;
}
~~~

The valid calculation remains a Sass number with a unit and compiles to 2rem.

## 16. Common mistakes and corrections

### Mistake: interpolating every regular property value

~~~scss
$color: tomato;

// Unnecessary for an ordinary property:
.badge {
  color: #{$color};
}
~~~

Prefer direct assignment:

~~~scss
.badge {
  color: $color;
}
~~~

### Mistake: using unquoted text where literal text is intended

~~~scss
// red may be parsed as a Sass color.
$label: red;
~~~

Quote the label:

~~~scss
$label: "red";

.label::before {
  content: $label;
}
~~~

### Mistake: interpolating uncontrolled text into a selector

~~~scss
// Do not build selectors from arbitrary external text.
~~~

Use a fixed list or otherwise constrained Sass values when generating selectors.

## Try it yourself

1. Create a quoted string containing the word warning and output it through content.
2. Generate .tab-active from a component name and state variable.
3. Store a Sass color in a CSS custom property.
4. Find the index of .svg in a filename and inspect the result with @debug.
5. Build a length from a number without turning the number into text.

### Answers

~~~scss
@use "sass:string";

$label: "warning";
$component: "tab";
$state: "active";
$brand: #2457c5;
$filename: "icon.svg";
$extension-position: string.index($filename, ".svg");
$spacing: 3;

.label::before {
  content: $label;
}

.#{$component}-#{$state} {
  color: $brand;
}

:root {
  --brand-color: #{$brand};
}

@debug $extension-position;

.card {
  padding: $spacing * 1rem;
}
~~~

The selector is .tab-active, the custom property receives the compiled color, the debug result is 5, and the padding is a unit-aware 3rem value.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="009-slash-division-and-calculations.md">009 - Slash Division and Calculations: Examples</a></td><td><a href="../lessons/010-strings-and-interpolation.md">Lesson</a></td><td><a href="../qnas/010-strings-and-interpolation.md">Q&amp;A</a></td><td><a href="011-lists-and-list-functions.md">011 - Lists and List Functions: Examples</a></td></tr>
  </tbody>
</table>
