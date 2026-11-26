# 007 - Sass Value Types and Null

## Learning goals

After this lesson, you should be able to:

- Explain what a Sass value is and how expressions and variables use values.
- Recognize numbers, strings, colors, lists, booleans, null, maps, and function references.
- Distinguish Sass data from the CSS text that the compiler emits.
- Use booleans and null intentionally in conditions and optional styles.
- Predict what Sass omits when a property or list item is null.
- Inspect evaluated values with @debug.

## 1. Values are the data Sass works with

A Sass value is the result of an expression. Variables store values, and property declarations pass values toward CSS output.

~~~scss
$base-size: 1rem;
$double-size: $base-size * 2;

@debug $base-size;   // 1rem
@debug $double-size; // 2rem
~~~

Sass evaluates its expressions while compiling. Some values can be written directly into CSS, such as a size, color, or font list. Other values, such as a Sass map or function reference, are useful to Sass logic but are not themselves CSS declarations.

Keep the stages clear:

1. Sass source contains expressions and Sass values.
2. The Sass compiler evaluates those expressions.
3. The compiler emits CSS for the browser.

A value does not automatically appear in generated CSS just because it exists in Sass.

## 2. The main Sass value types

| Type | Example | Typical use |
| --- | --- | --- |
| Number | **12**, **1.25**, **2rem**, **50%** | Measurements, ratios, and calculations |
| String | **"Inter"**, **sans-serif**, **open** | Font names, identifiers, and text |
| Color | **rebeccapurple**, **#336699**, **rgb(51 102 153)** | CSS colors and color calculations |
| List | **0.5rem 1rem**, **"Inter", sans-serif** | Sequences such as spacing and font stacks |
| Boolean | **true**, **false** | Conditions and feature switches |
| Null | **null** | Absence of a value or an optional result |
| Map | **("small": 0.75rem, "large": 1.5rem)** | Named keys associated with values |
| Function reference | A value returned by Sass function-reference APIs | Passing a function as data to advanced Sass utilities |

Numbers, strings, colors, and lists have direct counterparts in CSS. Booleans and null are Sass values used while compiling. Maps and function references are Sass data structures, normally read by Sass logic rather than written directly as CSS values.

Later chapters study these types in depth. This chapter establishes the vocabulary and focuses on booleans and null.

## 3. Numbers, strings, colors, and lists at a glance

### Numbers

A number may be unitless or have units:

~~~scss
$count: 3;
$gap: 1.25rem;
$ratio: 1.5;
$percentage: 50%;
~~~

Units are part of a number's meaning. For example, 12px and 12rem are both numbers but represent different dimensions. Sass can calculate compatible units; unit arithmetic is covered later.

### Strings

Strings may be quoted or unquoted:

~~~scss
$quoted-font: "Inter";
$unquoted-family: sans-serif;
$state: open;
~~~

Quoted text is useful when a value should remain clearly textual, especially for names that could otherwise be parsed as another Sass value. Unquoted strings are common for CSS identifiers such as sans-serif.

### Colors

A color may use a keyword, hexadecimal notation, or a CSS color function:

~~~scss
$brand: rebeccapurple;
$border: #336699;
$surface: rgb(245 247 250);
~~~

Sass also provides color functions. Their calculations and modern color syntaxes are covered in the color chapters.

### Lists

A list contains values separated by spaces, commas, or-in supported CSS contexts-slashes. Lists may be bracketed or unbracketed.

~~~scss
$spacing: 0.5rem 1rem 1.5rem;
$font-stack: "Inter", "Segoe UI", sans-serif;
$grid-line: [main-start] 1fr [main-end];
~~~

A list is Sass's general sequence type. Values that look like one item can still have list behavior, and an empty list is written as (). List separators, indexing, and list functions are covered later.

## 4. Booleans and conditions

A boolean is either true or false. Use booleans when a value represents an explicit yes-or-no choice.

~~~scss
$rounded: true;

.button {
  padding: 0.75rem 1rem;

  @if $rounded {
    border-radius: 0.5rem;
  }
}
~~~

Because $rounded is true, Sass emits the border radius. If it were false, Sass would skip the @if block.

Sass boolean operators are not, and, and or:

~~~scss
$has-icon: true;
$is-disabled: false;

@debug not $is-disabled; // true
@debug $has-icon and not $is-disabled; // true
~~~

Sass truthiness is important: only false and null are falsey. Other values-including 0, an empty string, and an empty list-are truthy. Do not carry JavaScript's truthiness rules into Sass.

~~~scss
@if 0 {
  .zero-is-truthy {
    display: block;
  }
}

@if "" {
  .empty-string-is-truthy {
    display: block;
  }
}
~~~

Both blocks compile because those conditions are truthy in Sass. When a branch should depend on a genuine yes-or-no choice, use true or false instead of relying on a number or string's truthiness.

## 5. What null means

Null is Sass's single value of its type. It means that a value is absent or that an operation has no result.

~~~scss
$optional-color: null;
$optional-color: #2457a7;
~~~

The first assignment has no usable color. The second gives the variable an actual color. Null is not the same as black, an empty string, or the boolean false.

Sass functions can return null when there is no result. For example, looking up a key that is not present in a map returns null:

~~~scss
@use "sass:map";

$font-sizes: ("small": 0.875rem, "large": 1.5rem);

@debug map.get($font-sizes, "large"); // 1.5rem
@debug map.get($font-sizes, "medium"); // null
~~~

Map lookup details are covered in the maps chapter. The key point here is that a function can use null to report that no value was found.

## 6. Null changes generated CSS

Sass treats null specially when generating CSS.

### A property whose value is null is omitted

~~~scss
$optional-shadow: null;

.card {
  color: #20242b;
  box-shadow: $optional-shadow;
}
~~~

Generated CSS:

~~~css
.card {
  color: #20242b;
}
~~~

The box-shadow declaration is omitted entirely. Sass does not output box-shadow: null.

This makes null useful for optional declarations in reusable styles:

~~~scss
@mixin card-surface($shadow: null) {
  border: 1px solid #d8dde5;
  box-shadow: $shadow;
}

.card {
  @include card-surface;
}

.card--raised {
  @include card-surface(0 0.5rem 1.25rem rgb(20 30 50 / 12%));
}
~~~

The base card has no box-shadow declaration. The raised card receives one because its argument is a real value.

### A null item in a list is omitted

~~~scss
$font-stack: "Inter", null, sans-serif;

body {
  font-family: $font-stack;
}
~~~

Generated CSS:

~~~css
body {
  font-family: "Inter", sans-serif;
}
~~~

The missing item disappears, while the other list items remain. This is not the same as inserting the literal word null into CSS.

### Null is falsey

A null value can also be used as a condition. Since it is falsey, Sass skips the block:

~~~scss
$focus-color: null;

.field {
  border: 1px solid #87909e;

  @if $focus-color {
    &:focus {
      outline: 2px solid $focus-color;
    }
  }
}
~~~

No focus rule is emitted while $focus-color is null. If the variable is assigned a color, the condition becomes truthy and Sass emits the rule.

For an optional style, this pattern makes the intent visible: provide a value to enable the style, and use null to leave it out.

## 7. False and null are different

Both false and null are falsey, so either can make an @if condition fail. Their meanings are different:

- **false** means a known boolean answer of “no.”
- **null** means no value is present.

Use false for a switch, such as whether a feature is enabled. Use null when an optional value or result is absent.

~~~scss
$show-icon: false;
$icon-color: null;

@if not $show-icon {
  // The feature switch is explicitly off.
}

@if not $icon-color {
  // No color value has been supplied.
}
~~~

Avoid using null as a generic replacement for every false-like value. Clear meanings make mixins, functions, and configuration easier to read.

## 8. Inspect values with @debug

The @debug rule prints a Sass value to the compiler's diagnostic output. It does not create a CSS declaration.

~~~scss
$gap: 1rem 1.5rem;
$enabled: true;
$not-configured: null;

@debug $gap;
@debug $enabled;
@debug $not-configured;
~~~

Use it while investigating a variable or expression. Compiler output shows the evaluated value; generated CSS shows which declarations survived compilation.

## 9. Common mistakes

### Expecting null to be printed as a CSS keyword

~~~scss
$width: null;

.box {
  width: $width;
}
~~~

This does not produce width: null. Sass omits the entire declaration.

### Using false when the intent is “no value”

A boolean flag communicates an explicit choice. An optional color, size, or shadow with no supplied value is usually clearer as null.

### Assuming JavaScript truthiness

In Sass, 0, an empty string, and () are truthy. Only false and null are falsey. Check the value you actually intend to test.

### Expecting Sass-only data structures to print as useful CSS

A map is a key/value structure for Sass code. It must be queried or transformed before its contents can be used in declarations.

### Confusing a Sass value with browser behavior

The Sass compiler evaluates Sass values before the browser sees the stylesheet. The browser receives emitted CSS, not Sass variables or Sass maps.

## 10. Complete example

~~~scss
$surface: #ffffff;
$text: #20242b;
$optional-border: null;
$font-stack: "Inter", null, sans-serif;
$show-highlight: true;

.panel {
  color: $text;
  background-color: $surface;
  border: $optional-border;
  font-family: $font-stack;

  @if $show-highlight {
    outline: 2px solid #86a8ff;
  }
}
~~~

Generated CSS:

~~~css
.panel {
  color: #20242b;
  background-color: #ffffff;
  font-family: "Inter", sans-serif;
  outline: 2px solid #86a8ff;
}
~~~

Read the output carefully:

- $text and $surface become ordinary CSS values.
- $optional-border is null, so the border declaration is omitted.
- The null item in $font-stack disappears from the list.
- $show-highlight is true, so Sass includes the outline.

## 11. Practice checklist

Before moving on, make sure you can answer these questions:

1. Which Sass values are falsey?
2. What happens to a declaration whose value is null?
3. What happens to a null item inside a CSS list?
4. Why is false not interchangeable with null?
5. Does @debug write to the generated stylesheet?
6. Which types from the table are primarily useful to Sass itself rather than direct CSS property values?
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="006-sass-variables-and-scope.md">006 - Sass Variables and Scope</a></td><td><a href="../qnas/007-sass-value-types-and-null.md">Q&amp;A</a></td><td><a href="../examples/007-sass-value-types-and-null.md">Example</a></td><td><a href="008-numbers-units-and-arithmetic.md">008 - Numbers, Units, and Arithmetic</a></td></tr>
  </tbody>
</table>
