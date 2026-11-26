# 010 - Strings and Interpolation

## Learning goals

After this lesson, you should be able to:

- Distinguish quoted strings from unquoted strings in Sass.
- Choose quotes or an unquoted CSS identifier for the value you mean.
- Escape quotes, backslashes, and characters in string values.
- Use interpolation to insert SassScript results into CSS syntax.
- Build selectors, property names, URLs, and custom property values safely.
- Explain why interpolation removes quotes from quoted strings.
- Avoid interpolating ordinary property values and numbers unnecessarily.
- Use basic functions from the sass:string module.

## 1. What is a Sass string?

A string is a sequence of characters. Sass supports two forms:

- A quoted string, written with single or double quotes.
- An unquoted string, commonly used for CSS identifiers such as bold or sans-serif.

Both forms store text, but they serialize differently.

~~~scss
$quoted-font: "Inter";
$unquoted-family: sans-serif;
$display-mode: block;
~~~

~~~scss
body {
  font-family: $quoted-font, "Segoe UI", $unquoted-family;
}

.panel {
  display: $display-mode;
}
~~~

Generated CSS:

~~~css
body {
  font-family: "Inter", "Segoe UI", sans-serif;
}

.panel {
  display: block;
}
~~~

Quoted strings preserve their text and quotes in CSS output. Unquoted strings are appropriate when the CSS grammar expects an identifier or keyword.

## 2. Prefer quoted strings for general text

A quoted string makes it clear that the value is text:

~~~scss
$font-name: "Open Sans";
$label: "Save changes";
$path: "images/banner.svg";
~~~

Quoted strings are useful for font family names, generated content, accessible labels, file names, and values that could otherwise be parsed as another Sass type.

~~~scss
$brand-name: "Blue Harbor";

.banner::after {
  content: $brand-name;
}
~~~

Generated CSS:

~~~css
.banner::after {
  content: "Blue Harbor";
}
~~~

A quoted string containing multiple words remains one string.

## 3. Use unquoted strings for CSS identifiers

Many CSS values are unquoted identifiers:

~~~scss
$display: flex;
$weight: bold;
$fallback: sans-serif;

.layout {
  display: $display;
  font-weight: $weight;
  font-family: $fallback;
}
~~~

Generated CSS:

~~~css
.layout {
  display: flex;
  font-weight: bold;
  font-family: sans-serif;
}
~~~

Not every identifier-looking token becomes an unquoted string. Sass parses certain words as other value types:

- CSS color names such as red are colors.
- true and false are booleans.
- null is the null value.
- and, or, and not are boolean operators.

When text should definitely be treated as text, quote it.

~~~scss
$color-name-as-text: "red";
$feature-label: "true";
~~~

## 4. Single quotes and double quotes

Both quote styles create quoted strings. Choose the style that keeps the text readable and escape only when the string contains its own delimiter.

~~~scss
$double-quoted: "Open Sans";
$single-quoted: 'Open Sans';
$contains-double-quotes: 'She said "hello".';
$contains-single-quote: "Don't stop.";
~~~

Both quote styles have the same Sass string behavior. Be consistent within a project.

## 5. Escape special characters

Use a backslash to include a quote or backslash that would otherwise end or alter a quoted string.

~~~scss
$quote-in-double-string: "She said \"hello\".";
$windows-path: "C:\\projects\\styles";
~~~

A CSS Unicode escape uses a backslash followed by hexadecimal code points. A trailing space can mark where the escape ends:

~~~scss
$check-mark: "\2713 ";

.status::before {
  content: $check-mark;
}
~~~

The escape represents a check mark. Sass emits a CSS string that the browser interprets as that character.

Use escapes when needed for valid CSS text. For ordinary text, literal Unicode characters are also readable and valid in CSS.

## 6. Interpolation inserts a SassScript result

Interpolation uses the form #{expression}. It inserts the result of a SassScript expression into surrounding CSS syntax.

~~~scss
$component: "button";
$state: "active";

.#{$component}--#{$state} {
  color: white;
  background-color: #2457a7;
}
~~~

Generated CSS:

~~~css
.button--active {
  color: white;
  background-color: #2457a7;
}
~~~

Interpolation is useful when part of a CSS token or name needs to be dynamic. It is not required just to use a variable as a value.

~~~scss
$accent: #2457a7;

.link {
  color: $accent;
}
~~~

Prefer the direct value above over color: #{$accent}. Sass can preserve the value's type when it is passed directly.

## 7. Interpolate inside selectors

Interpolation can add a Sass value to a selector:

~~~scss
$component: "alert";
$variant: "warning";

.#{$component}--#{$variant} {
  border-color: #b45309;
}
~~~

Generated CSS:

~~~css
.alert--warning {
  border-color: #b45309;
}
~~~

The inserted text must form valid selector syntax. Interpolation inserts text; it does not design, validate, or sanitize a selector for you.

For a fixed set of variants, Sass maps and loops can generate selectors. Those tools are covered in later lessons.

## 8. Interpolate in property names

Interpolation can build a property name when that is useful:

~~~scss
$axis: inline-start;

.card {
  margin-#{$axis}: 1rem;
}
~~~

Generated CSS:

~~~css
.card {
  margin-inline-start: 1rem;
}
~~~

Use this sparingly. If a direct CSS property name is clearer, write the property directly.

## 9. Interpolate in URLs and other CSS tokens

Interpolation can insert part of a URL or identifier:

~~~scss
$icon-name: "search";

.search-button {
  background-image: url("/icons/#{$icon-name}.svg");
}
~~~

Generated CSS:

~~~css
.search-button {
  background-image: url("/icons/search.svg");
}
~~~

Make sure the inserted value is safe and valid for the context. For arbitrary text, URL paths, or identifiers, quoting and escaping rules matter.

## 10. Interpolate in quoted strings

Interpolation can insert a variable into a quoted string:

~~~scss
$variant: "compact";

@debug "Current variant: #{$variant}";
~~~

The result is one quoted string:

~~~text
"Current variant: compact"
~~~

When a quoted string is interpolated into another value, its surrounding quotes are removed as part of interpolation. For example:

~~~scss
$selector-part: "alert";
$selector: ".#{$selector-part}:hover";

@debug $selector;
~~~

The interpolated selector text is unquoted:

~~~text
.alert:hover
~~~

This makes it possible to assemble syntax such as selectors. It also means interpolation can change whether the result is quoted or unquoted, so check the context in which the result is used.

## 11. Interpolate into CSS custom property values

CSS custom property values are parsed as CSS tokens. Use interpolation when a Sass value must be inserted into a custom property:

~~~scss
$accent: #2457a7;
$space: 1rem;

:root {
  --accent-color: #{$accent};
  --control-space: #{$space};
}
~~~

Generated CSS:

~~~css
:root {
  --accent-color: #2457a7;
  --control-space: 1rem;
}
~~~

For ordinary CSS properties, use a variable directly. Custom property declarations are a special case because their values are treated as plain CSS text.

## 12. Interpolation in at-rules and comments

Interpolation can create a dynamic name for a CSS at-rule:

~~~scss
$animation-name: fade-in;

@keyframes #{$animation-name} {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.panel {
  animation-name: $animation-name;
}
~~~

Generated CSS:

~~~css
@keyframes fade-in {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.panel {
  animation-name: "fade-in";
}
~~~

Interpolation is also available in loud CSS comments:

~~~scss
$build-label: "theme-v2";

/*! Build: #{$build-label} */
~~~

Generated CSS:

~~~css
/*! Build: theme-v2 */
~~~

Loud comments beginning with /*! are preserved in CSS output. Ordinary Sass comments are removed.

## 13. Interpolation in SassScript creates an unquoted string

When interpolation is used in SassScript, the result is an unquoted string:

~~~scss
$prefix: "ms";
$flex-value: -#{$prefix}-flex;

@debug $flex-value;
~~~

The value is the unquoted identifier -ms-flex. It can be useful for constructing CSS identifiers or slash-separated values.

Do not use interpolation to build a number with units. A string that looks like 12px is still a string, not a numeric Sass value.

~~~scss
$number: 12;

// Avoid this: it creates text, not a length.
// $bad-width: #{$number}px;

// Keep the value numeric:
$good-width: $number * 1px;
~~~

Numeric values should carry real units so Sass can perform arithmetic and report unit errors.

## 14. Quote and unquote with sass:string

Use the built-in string module when the intent is specifically to change whether a string is quoted:

~~~scss
@use "sass:string";

$keyword: bold;
$quoted-keyword: string.quote($keyword);
$selector-text: string.unquote(".widget:hover");

@debug $quoted-keyword; // "bold"
@debug $selector-text;  // .widget:hover
~~~

string.quote() returns a quoted string. string.unquote() returns an unquoted string. An unquoted string is not automatically valid in every CSS context, so unquote only when the target syntax requires it.

Prefer string.unquote($value) over interpolation as a way to remove quotes. It states the operation directly.

## 15. Basic string functions

The sass:string module also provides functions to inspect and transform strings:

~~~scss
@use "sass:string";

$font-name: "Inter Display";

@debug string.length($font-name); // 13
@debug string.index($font-name, "Display"); // 7
@debug string.slice($font-name, 1, 5); // "Inter"
@debug string.to-upper-case("primary"); // "PRIMARY"
@debug string.to-lower-case("SECONDARY"); // "secondary"
~~~

String indexes are one-based: the first character is at index 1. Negative indexes count from the end, so -1 refers to the last character.

~~~scss
@use "sass:string";

@debug string.slice("Roboto Mono", -4); // "Mono"
~~~

The string module is useful when a Sass function or mixin must inspect or transform text. For normal CSS declarations, passing a string variable directly is usually enough.

## 16. Common mistakes

### Interpolating a value that can be passed directly

~~~scss
$accent: #2457a7;

.link {
  color: $accent;
}
~~~

Sass already accepts a variable as a property value. Interpolation is unnecessary here.

### Assuming every unquoted word is a string

Words such as red, null, true, and false may be parsed as colors or special Sass values. Quote text when it must remain a string.

### Forgetting that interpolation removes quotes

Quoted strings lose their surrounding quotes when interpolated into another value. This is useful for selectors, but can produce the wrong CSS if you expected the inserted text to stay quoted.

### Interpolating numbers to add units

Interpolation creates an unquoted string and bypasses Sass's numeric unit checks. Give the number a real unit or use unit arithmetic.

### Unquoting without checking the CSS context

An unquoted string may contain text that is invalid as a CSS identifier or property value. Only unquote when the target syntax expects unquoted text.

### Treating interpolation as sanitization

Interpolation inserts the supplied text. It does not guarantee the result is a valid selector, property name, URL, or at-rule identifier.

## 17. Worked example: icon and animation names

~~~scss
@use "sass:string";

$icon: "search";
$motion-name: inline-#{string.unique-id()};

.icon-#{$icon} {
  background-image: url("/icons/#{$icon}.svg");
  animation-name: $motion-name;
}

@keyframes #{$motion-name} {
  from {
    opacity: 0;
    transform: translateY(0.25rem);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}
~~~

Generated CSS has a selector and URL based on the icon name, plus a unique valid keyframe identifier. The exact unique suffix changes between Sass compilations.

Use dynamic interpolation when a name truly needs to be assembled. Keep ordinary property values as direct Sass values.

## 18. Practice checklist

Before moving on, make sure you can answer these questions:

1. How do quoted and unquoted strings differ in CSS output?
2. Which identifier-like values does Sass parse as colors, booleans, or null?
3. What does interpolation do?
4. Why are quotes removed when a quoted string is interpolated?
5. When should a Sass variable be interpolated into a CSS custom property?
6. Why is interpolation not a good way to add a unit to a number?
7. What is the difference between string.unquote() and interpolation?
8. Which index refers to the first character in a Sass string?
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="009-slash-division-and-calculations.md">009 - Slash Division and Calculations</a></td><td><a href="../qnas/010-strings-and-interpolation.md">Q&amp;A</a></td><td><a href="../examples/010-strings-and-interpolation.md">Example</a></td><td><a href="011-lists-and-list-functions.md">011 - Lists and List Functions</a></td></tr>
  </tbody>
</table>
