# 005. SCSS Syntax and Rules - Examples

This guide uses a small stylesheet to identify SCSS rules, declarations, blocks, expressions, comments, and the CSS output produced by Sass.

## Example 1: A complete SCSS stylesheet

Create this project:

~~~text
scss-syntax-demo/
├── index.html
├── src/
│   └── main.scss
└── dist/
~~~

Create src/main.scss:

~~~scss
$brand: #1769aa;
$text: #243447;

// This source-only note will not appear in the CSS.

/* This block comment is emitted in expanded CSS. */

:root {
  --brand: #{$brand};
}

.card {
  padding: 1rem;
  color: $text;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;

  .card__title {
    margin: 0;
    color: $brand;
  }

  p {
    line-height: 1.6;
  }
}
~~~

Compile it:

~~~sh
npx sass src/main.scss dist/main.css
~~~

The generated stylesheet is ordinary CSS:

~~~css
/* This block comment is emitted in expanded CSS. */
:root {
  --brand: #1769aa;
}

.card {
  padding: 1rem;
  color: #243447;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
}
.card .card__title {
  margin: 0;
  color: #1769aa;
}
.card p {
  line-height: 1.6;
}
~~~

The Sass variables have been resolved, nested rules have become ordinary selectors, and the silent comment is absent. Dart Sass also writes a source map by default when it writes this CSS.

Create index.html:

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>SCSS syntax example</title>
    <link rel="stylesheet" href="./dist/main.css">
  </head>
  <body>
    <main class="card">
      <h1 class="card__title">A rule has a block</h1>
      <p>Declarations inside the block define the presentation.</p>
    </main>
  </body>
</html>
~~~

The HTML links to the generated CSS output. The browser does not load the SCSS file.

## Example 2: Identify the parts of a rule

Consider this rule:

~~~scss
.card {
  padding: 1rem;
  border: 1px solid #cbd5e1;
}
~~~

| Part | Text | Purpose |
| --- | --- | --- |
| Selector | .card | Selects matching elements. |
| Block | The contents between { and } | Groups declarations for the selector. |
| Property | padding | Names the CSS setting. |
| Colon | : | Separates a property from its value. |
| Value | 1rem | Sets the property value. |
| Semicolon | ; | Ends the declaration. |

The final semicolon before the closing brace is optional in SCSS, but keeping it makes the style consistent and easier to extend.

## Example 3: Comments and generated output

Create src/comments.scss:

~~~scss
// This is a silent comment. It stays out of the generated CSS.

/* This is a block comment. It is normally kept in expanded CSS. */

/*! This comment is marked to remain in compressed CSS. */

.message {
  color: #243447;
}
~~~

Compile with expanded output:

~~~sh
npx sass --style=expanded src/comments.scss dist/comments.css
~~~

The first comment is omitted, while the ordinary block comment and the /*! comment are kept. Now compile with compressed output:

~~~sh
npx sass --style=compressed src/comments.scss dist/comments.min.css
~~~

In compressed mode, Sass removes the ordinary block comment but retains the comment that begins with /*!.

## Example 4: Sass variable or CSS custom property?

A Sass variable is evaluated during compilation:

~~~scss
$accent: #1769aa;

.link {
  color: $accent;
}
~~~

The output contains the resolved color:

~~~css
.link {
  color: #1769aa;
}
~~~

A CSS custom property remains in CSS so the browser can use it at runtime:

~~~scss
$accent: #1769aa;

:root {
  --accent: #{$accent};
}

.link {
  color: var(--accent);
}
~~~

The interpolation markers insert the Sass value into the custom property text. Without interpolation, a value that looks like SassScript can be passed through literally:

~~~scss
:root {
  --accent: $accent;
}
~~~

That does not substitute the Sass variable. Use the interpolated form when a Sass value needs to become part of a custom property value.

## Example 5: SCSS and indented Sass

The SCSS form uses braces and semicolons:

~~~scss
.notice {
  color: navy;
  padding: 1rem;
}
~~~

The equivalent indented syntax uses indentation and newlines:

~~~sass
.notice
  color: navy
  padding: 1rem
~~~

The output CSS is equivalent:

~~~css
.notice {
  color: navy;
  padding: 1rem;
}
~~~

The file extension determines the syntax: use .scss for the first form and .sass for the second.

## Example 6: Find and fix syntax errors

This rule is missing the semicolon between declarations:

~~~scss
.card {
  color: navy
  padding: 1rem;
}
~~~

Correct it by separating the declarations:

~~~scss
.card {
  color: navy;
  padding: 1rem;
}
~~~

This example is missing a closing brace:

~~~scss
.card {
  padding: 1rem;
~~~

Add the missing closing brace:

~~~scss
.card {
  padding: 1rem;
}
~~~

When Sass reports a parsing error, start with the first diagnostic and inspect the lines immediately before it. A missing delimiter earlier in the file can make later lines appear invalid.

## Try these changes

1. Add a declaration to .card and recompile.
2. Change the value of $brand and check both the CSS declaration and custom property output.
3. Add a // comment and a /* block comment */, then compare the generated CSS.
4. Compile comments.scss in expanded and compressed modes and compare the comments.
5. Add a nested rule and write down the flat selector Sass produces.
6. Remove a brace or semicolon, read the Sass error, and then fix the source.
7. Rewrite one SCSS rule in the indented .sass syntax.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="004-compiling-scss-to-css.md">004 - Compiling SCSS to CSS - Examples</a></td><td><a href="../lessons/005-scss-syntax-and-rules.md">Lesson</a></td><td><a href="../qnas/005-scss-syntax-and-rules.md">Q&amp;A</a></td><td><a href="006-sass-variables-and-scope.md">006 - Sass Variables and Scope - Examples</a></td></tr>
  </tbody>
</table>
