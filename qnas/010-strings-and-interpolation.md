# 010 - Strings and Interpolation: Q&A

## Core concepts

### 1. What is a string in Sass?

A string is text. Sass supports quoted strings, such as "Hello", and unquoted strings, such as sans-serif or center. Quoted strings preserve their string form; unquoted strings are commonly used for CSS identifiers and other CSS syntax.

### 2. What is the practical difference between quoted and unquoted strings?

The distinction affects how Sass interprets and serializes a value. Quotes make it clear that a value is text. Unquoted values can be interpreted as CSS identifiers, numbers, colors, booleans, null, or other Sass values depending on their spelling and context.

Use quotes for general human-readable text. Use an unquoted value when the intended result is valid CSS syntax, such as a keyword or identifier.

### 3. Why should ordinary text usually be quoted?

Some unquoted words have special meaning to Sass. For example, true and false are booleans, null is the null value, and red is a color. Quoting a label such as "red" makes its text intent unambiguous.

### 4. Are all unquoted values Sass strings?

No. A token that looks like a color, number, boolean, null, list, or another Sass value may be parsed as that value rather than as a string. Use a quoted string when the exact text must remain text.

### 5. Which quote style should I use?

Either single or double quotes can represent a quoted Sass string. Pick one style for consistency. If the text contains the chosen quote character, escape it or use the other quote style when that keeps the string clearer.

### 6. How do I include a quote or backslash inside a quoted string?

Escape the special character with a backslash. For example, a double quote inside a double-quoted string can be written as \". A backslash itself can be escaped as \\. Sass also supports CSS-style hexadecimal escapes for characters when needed.

### 7. What is interpolation?

Interpolation inserts the result of a SassScript expression into a larger piece of text or CSS syntax. The syntax is a hash sign, an opening brace, the expression, and a closing brace: #{expression}.

### 8. Where can interpolation be used?

Interpolation can construct selectors, property names, parts of values, custom property values, at-rule parameters, and other CSS syntax. It can also insert Sass values into quoted strings and comments.

Use it where Sass must produce syntax from a value. Keep the resulting selector, property, or value valid CSS.

### 9. Does interpolation preserve quotes around a quoted string?

No. When a quoted string is inserted through interpolation, its quotes are removed. Interpolation produces an unquoted string result so that the inserted text can become part of the surrounding syntax.

If quotes must remain in emitted CSS, construct the quoted CSS string deliberately and inspect the generated CSS.

### 10. Can I interpolate a Sass variable into a normal CSS property value?

Usually there is no need. Assign the Sass variable directly:

~~~scss
$accent: #2864dc;

.button {
  color: $accent;
}
~~~

Sass already evaluates normal property values. Interpolation is useful when the value is part of syntax Sass would otherwise parse differently, not as a default way to print every variable.

### 11. Why do CSS custom property values often use interpolation?

Custom properties preserve token streams for the browser, so Sass treats their values differently from ordinary SassScript property values. To insert a Sass variable into a custom property, interpolate it:

~~~scss
$accent: #2864dc;

:root {
  --accent: #{$accent};
}
~~~

The interpolation places the Sass value into the CSS custom property value.

### 12. How can interpolation build a selector?

Interpolate a variable into the selector text:

~~~scss
$component: "card";

.#{$component} {
  border-radius: 0.5rem;
}
~~~

This emits a .card selector. Keep the variable restricted to trusted, valid selector text.

### 13. How can interpolation build a property name?

Interpolate the dynamic part of the property name:

~~~scss
$side: "left";

.notice {
  border-#{$side}-width: 2px;
}
~~~

This emits border-left-width. Prefer a small, controlled set of property-name fragments so the result stays predictable.

### 14. Should I use interpolation to create units from numbers?

No. Interpolation turns the result into text and loses numeric type information. Do not build a length by interpolating a number and a unit. Use Sass number arithmetic or unit-aware values instead.

For example, use 2 * 1rem when you need a calculated length. This keeps the value numeric and lets Sass check compatible units.

### 15. What does string.quote() do?

The sass:string module's quote() function returns a quoted string. It is useful when a value should be represented as quoted text.

### 16. What does string.unquote() do?

The unquote() function removes quotes from a quoted string and returns an unquoted string. Use it only when the unquoted output is valid for the CSS position where it will appear. Removing quotes can make Sass interpret or emit a value as CSS syntax rather than text.

### 17. How do I load Sass string functions?

Use the module system:

~~~scss
@use "sass:string";

$label: string.quote(hello);
$plain-label: string.unquote("hello");
~~~

The namespace keeps built-in functions organized and avoids relying on deprecated global function names.

### 18. Are string indices zero-based?

No. The sass:string module uses one-based indices. The first character is at index 1. Negative indices count backward from the end, so -1 refers to the last character.

### 19. What do string.index() and string.slice() return?

string.index($string, $substring) returns the one-based position of the first match, or null when there is no match. string.slice($string, $start-at, $end-at) returns a substring; its indices use the same one-based and negative-index rules.

### 20. Can interpolation be used inside a quoted string?

Yes. Sass replaces interpolation inside a quoted string and retains the outer quotes around the resulting string:

~~~scss
$theme: "ocean";

.banner {
  background-image: url("/themes/#{$theme}.svg");
}
~~~

The interpolation inserts the theme name into the URL text.

### 21. Can I interpolate a selector or property without a variable?

Yes, but it is rarely useful. Interpolation is for inserting SassScript results into text. If the text is static, write the selector or property directly. Avoid extra interpolation because it can make styles harder to read.

### 22. What can go wrong when interpolating into CSS syntax?

The inserted text may create an invalid selector, property name, URL, or value. It may also create unintended syntax if the source value contains punctuation or whitespace. Validate or constrain dynamic pieces and check the compiled CSS.

### 23. Is interpolation a way to escape untrusted input?

No. Interpolation does not make arbitrary input safe. Sass is a build-time tool, and interpolation can place raw text into CSS syntax. Do not treat arbitrary user-provided text as a safe selector, property name, or CSS value.

### 24. What is string.unique-id() for?

It creates a unique unquoted string identifier during compilation. It can be useful when a generated identifier is needed, but its exact suffix is not stable or predictable. Do not hard-code expected output from the generated suffix.

### 25. What is a good debugging workflow for string and interpolation issues?

1. Identify whether each value is a quoted string, unquoted string, or another Sass type.
2. Compile the smallest example that reproduces the problem.
3. Inspect the generated CSS and confirm quotes, whitespace, and punctuation.
4. Remove interpolation from ordinary property values unless it is needed.
5. Check that dynamic pieces form valid CSS syntax.

## Quick checks

### 26. Why does a color name sometimes stop behaving like text?

An unquoted color name may be parsed as a Sass color. Quote it when you mean the literal word rather than a color value.

### 27. Why did a quoted string lose its quotes after interpolation?

Interpolation converts the inserted result to an unquoted string so it can participate in the surrounding syntax. This is expected Sass behavior.

### 28. Why does inserting a Sass variable into a custom property sometimes fail or look unusual?

Custom property values are preserved as CSS token streams. Interpolate the Sass value in the custom property's declaration, then inspect the compiled output.

### 29. Why is direct variable assignment clearer for a regular property?

Sass already evaluates regular property values. Direct assignment keeps the value typed and the stylesheet easier to read. Interpolation is reserved for places where Sass needs to construct text or syntax.

### 30. Which approach should I choose?

- Human-readable text: use a quoted string.
- A CSS keyword or identifier: use valid unquoted CSS syntax.
- A Sass value in a regular property: assign it directly.
- A Sass value inside a custom property or dynamic CSS syntax: interpolate it carefully.
- A number with a unit: use Sass arithmetic and unit-aware numbers, not string concatenation.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="009-slash-division-and-calculations.md">009 - Slash Division and Calculations: Q&amp;A</a></td><td><a href="../lessons/010-strings-and-interpolation.md">Lesson</a></td><td><a href="../examples/010-strings-and-interpolation.md">Example</a></td><td><a href="011-lists-and-list-functions.md">011 - Lists and List Functions: Q&amp;A</a></td></tr>
  </tbody>
</table>
