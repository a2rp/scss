# 005. SCSS Syntax and Rules

## Learning goals

After this lesson, you should be able to:

- Recognize the basic structure of an SCSS stylesheet.
- Distinguish style rules, declarations, blocks, expressions, and at-rules.
- Apply braces, colons, and semicolons correctly.
- Read common Sass statements without confusing them with CSS declarations.
- Use single-line and block comments with the expected output behavior.
- Understand how SCSS differs from the indented .sass syntax.
- Recognize a few places where valid CSS needs special handling in SCSS.

## SCSS is CSS-shaped Sass

SCSS is the most commonly used Sass syntax. It uses the .scss file extension and resembles CSS: rules use braces, declarations use colons, and statements are separated with semicolons.

Most ordinary CSS can be written in an SCSS file unchanged:

~~~scss
.notice {
  color: #243447;
  background-color: #f4f7fb;
}
~~~

SCSS also supports Sass features, such as variables and nested rules. The compiler evaluates Sass syntax and emits CSS.

The other Sass syntax uses the .sass extension. It relies on indentation and newlines instead of braces and semicolons. Both syntaxes support the Sass language features, but this course uses SCSS in most examples.

## A stylesheet is a sequence of statements

A Sass stylesheet is made of statements. Some statements create CSS; others tell Sass how to process the stylesheet.

Common statement categories include:

| Statement | Example | Main purpose |
| --- | --- | --- |
| Style rule | .card { color: navy; } | Selects elements and adds CSS declarations. |
| Variable declaration | $brand: navy; | Stores a Sass value for later use. |
| CSS at-rule | @media (width >= 40rem) { ... } | Uses a CSS feature such as media queries. |
| Sass at-rule | @use "tokens"; | Loads or controls Sass features. |
| Mixin definition or use | @mixin, @include | Defines and applies reusable Sass content. |

Some constructs, such as variables and mixins, are introduced in more depth in later lessons. Here the goal is to recognize the kinds of statements that can appear in an SCSS file.

## Style rules, selectors, blocks, and declarations

A style rule has a selector and a block. The block is enclosed in braces. It usually contains property declarations:

~~~scss
.card {
  padding: 1rem;
  border: 1px solid #cbd5e1;
}
~~~

In this rule:

- .card is the selector.
- The opening and closing braces define the block.
- padding and border are property names.
- The values after the colons are property values.
- Each declaration is separated by a semicolon.

A declaration follows the general form:

~~~text
property: value;
~~~

The property value can be an ordinary CSS value or a Sass expression. Sass expressions may contain values such as variables or function calls; later lessons cover those in detail.

## Braces and semicolons

In SCSS, braces define blocks:

~~~scss
.panel {
  display: grid;
}
~~~

Use semicolons to separate declarations and other statements within a block:

~~~scss
.panel {
  display: grid;
  gap: 1rem;
}
~~~

The semicolon after the final statement immediately before a closing brace is technically optional. Keeping it is a helpful team convention because adding another declaration later is less error-prone.

A semicolon is not a substitute for a closing brace. Every opened block needs its matching closing brace:

~~~scss
.panel {
  .panel__title {
    margin: 0;
  }
}
~~~

The nested .panel__title rule has its own block, so it has its own closing brace. Missing or misplaced braces often cause parser errors far from the actual mistake.

## Nested style rules

SCSS lets you place a style rule inside another style rule. Sass combines the selectors when it compiles:

~~~scss
.card {
  padding: 1rem;

  .card__title {
    margin: 0;
  }
}
~~~

The output is regular CSS:

~~~css
.card {
  padding: 1rem;
}
.card .card__title {
  margin: 0;
}
~~~

Nesting can show that a rule belongs to a component, but deep nesting produces long selectors and makes the output harder to predict. Keep nesting shallow and inspect the generated CSS when a nested rule is not obvious.

Selector details and advanced nesting patterns are covered separately. This example only introduces how nested blocks are written in SCSS.

## At-rules

An at-rule begins with the @ symbol. CSS at-rules such as @media, @supports, @font-face, and @keyframes can appear in an SCSS stylesheet:

~~~scss
@media (width >= 40rem) {
  .layout {
    grid-template-columns: 16rem 1fr;
  }
}
~~~

Some CSS at-rules have blocks; others end with a semicolon:

~~~scss
@charset "UTF-8";

@media (prefers-reduced-motion: reduce) {
  .animated {
    animation-duration: 0.01ms;
  }
}
~~~

Sass also defines its own at-rules, including @use, @mixin, @include, and @function. Their behavior is Sass-specific, but the syntax still uses the @ prefix. Dedicated lessons explain those features.

Prefer the modern Sass module system for loading Sass files. The old Sass @import rule is deprecated in Dart Sass; module syntax is explained in the module lessons.

## Expressions and values

The part after a declaration’s colon is an expression. A simple expression can be a CSS value:

~~~scss
.label {
  font-weight: 700;
  transform: translateY(0.25rem);
}
~~~

It can also use SassScript, the name for Sass’s expression syntax:

~~~scss
$space: 1rem;

.label {
  margin-bottom: $space;
}
~~~

A value may include spaces, commas, quoted strings, colors, numbers, units, or CSS functions. Do not split a value into separate declarations just because it contains spaces or commas:

~~~scss
.page {
  font-family: "Segoe UI", Arial, sans-serif;
  padding: 1rem 1.5rem;
  box-shadow: 0 0.25rem 0.75rem rgb(0 0 0 / 12%);
}
~~~

The declaration still ends at its semicolon. The commas and spaces inside a value are part of that value.

## Sass variables and CSS custom properties

Sass variables begin with a dollar sign and are processed during compilation:

~~~scss
$accent: #1769aa;

.button {
  color: white;
  background-color: $accent;
}
~~~

The compiled CSS contains the resolved color. The Sass variable itself is not sent to the browser.

CSS custom properties begin with two hyphens and remain in the generated CSS. Their values are available to the browser at runtime:

~~~scss
$accent: #1769aa;

:root {
  --accent: #{$accent};
}

.button {
  background-color: var(--accent);
}
~~~

The interpolation markers around the Sass variable in the custom property value are important. Custom property values are passed through as CSS text, so SassScript values must be interpolated into them explicitly.

Sass variables and CSS custom properties solve different problems: Sass values are compiled away, while CSS custom properties remain available to CSS and JavaScript.

## Comments

SCSS supports silent comments and loud comments.

### Silent comments

A double slash starts a single-line comment:

~~~scss
// This comment is only for people reading the source.
.card {
  padding: 1rem; // This note is not emitted as CSS.
}
~~~

Silent comments are removed during compilation and do not appear in the CSS output.

### Loud comments

A block comment uses /* and */:

~~~scss
/*
  This comment is normally included in expanded CSS output.
*/
.card {
  padding: 1rem;
}
~~~

Block comments are normally emitted in CSS. Sass strips them from compressed output unless the comment begins with /*!:

~~~scss
/*! Keep this license or attribution comment in compressed CSS. */
~~~

Use comments to explain intent, constraints, browser workarounds, or non-obvious decisions. Avoid comments that only repeat what the property already says.

## Whitespace and formatting

Whitespace between SCSS statements is mostly for readability. These declarations have the same meaning:

~~~scss
.box {
  padding: 1rem;
  color: navy;
}
~~~

~~~scss
.box{padding:1rem;color:navy}
~~~

The first style is easier for people to read, review, and debug. Use a consistent formatter or team style for indentation, blank lines, quote choices, and selector layout.

A practical convention is:

- Indent nested blocks consistently.
- Put one declaration on each line.
- Keep a semicolon after every declaration, including the last one in a block.
- Put a blank line between logically separate groups.
- Keep selectors and nested blocks easy to scan.
- Save Sass source files as UTF-8.

Formatting does not change the intended CSS, but consistent formatting reduces mistakes and makes diffs easier to review.

## SCSS and indented Sass syntax

The same simple rule can be written in SCSS:

~~~scss
.notice {
  color: navy;
  padding: 1rem;
}
~~~

Or in the indented .sass syntax:

~~~sass
.notice
  color: navy
  padding: 1rem
~~~

SCSS uses braces and semicolons. Indented Sass uses indentation to define blocks and newlines to separate statements. A project should choose a syntax consistently so its source files and tools are predictable.

## Syntax errors

Sass reports invalid SCSS as a parse error and usually identifies a file, line, and nearby source text. Unlike a browser parsing CSS, Sass generally stops compilation when its input cannot be parsed.

Common causes include:

- A missing closing brace.
- A missing semicolon between statements.
- A colon used in the wrong place.
- A quote or block comment that was never closed.
- A property declaration outside a style rule or supported at-rule.
- Sass syntax used where a custom property expects CSS text.

When a compiler reports a location, inspect a few lines before it as well as the named line. A missing brace earlier in the file can cause later code to be parsed in the wrong block.

## Key takeaways

- SCSS is a CSS-shaped Sass syntax that uses braces and semicolons.
- A stylesheet is a sequence of statements; style rules contain declarations.
- Declarations use property: value; and are grouped inside blocks.
- Nested rules compile to ordinary CSS selectors.
- CSS and Sass at-rules begin with @, but Sass-specific rules are processed by the compiler.
- Silent comments do not appear in CSS; block comments normally do.
- CSS custom properties remain in the output, and Sass values in them require interpolation.
- Keep formatting consistent and save Dart Sass source files as UTF-8.

## Practice

1. Create a small SCSS rule and label its selector, block, declarations, properties, values, and semicolons.
2. Remove a closing brace and observe the compiler’s parse error. Restore it and compile again.
3. Add a silent comment and a block comment, compile the file, and compare the output.
4. Put a nested rule inside a component rule and write the equivalent flat CSS selector.
5. Add a Sass variable to a CSS custom property with interpolation, then inspect the generated CSS.
6. Rewrite one simple SCSS rule using the indented .sass syntax and compare the structure.
7. Format a compressed one-line source into readable SCSS without changing the declarations.

## Further reading

- [SCSS and Sass syntax](https://sass-lang.com/documentation/syntax/)
- [Stylesheet structure and statements](https://sass-lang.com/documentation/syntax/structure/)
- [Sass comments](https://sass-lang.com/documentation/syntax/comments/)
- [Parsing Sass stylesheets](https://sass-lang.com/documentation/syntax/parsing/)
- [Property declarations and custom properties](https://sass-lang.com/documentation/style-rules/declarations/)
- [Sass style rules](https://sass-lang.com/documentation/style-rules/)
