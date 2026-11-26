# 001. Introduction to Sass and SCSS

## Learning goals

By the end of this lesson, you should be able to:

- Explain what Sass is and why a project might use it.
- Distinguish Sass the language from SCSS, one of its syntaxes.
- Describe how Sass source becomes browser-ready CSS.
- Recognize Sass features without confusing them with CSS or a framework.
- Compile a small SCSS file and connect its generated CSS to an HTML page.

## What is Sass?

Sass is a stylesheet language that extends CSS with features for authoring and organizing styles. A Sass compiler reads Sass source and produces CSS. The browser uses that generated CSS; it does not execute Sass source directly.

Sass helps when stylesheets grow and repeated values or patterns need to stay consistent. It provides:

- Variables for reusable compile-time values.
- Nesting for grouping related selectors.
- Mixins for reusable blocks of styles.
- Functions for calculating or transforming values.
- Modules for organizing and sharing stylesheet members.
- Control flow for generating styles from data or conditions.

These features help authors write stylesheets. They do not replace CSS knowledge: the browser still applies the cascade, inheritance, selectors, layout rules, and media queries to the final CSS.

## Sass and SCSS

Sass supports two syntaxes:

1. **SCSS** uses braces and semicolons and looks similar to CSS. Files normally end in `.scss`.
2. **Indented Sass** uses indentation and line breaks instead of braces and most semicolons. Files normally end in `.sass`.

This course focuses on SCSS. Most valid CSS is also valid SCSS, so an existing CSS rule can usually be placed in an SCSS file and compiled without rewriting it.

SCSS:

~~~scss
.notice {
  color: #263238;
  background-color: #e7f1ff;
}
~~~

Indented Sass expresses the same rule differently:

~~~sass
.notice
  color: #263238
  background-color: #e7f1ff
~~~

Pick the syntax used by a project and keep it consistent. The file extension tells Sass which syntax to parse.

## Sass is compiled; CSS is what the browser receives

A basic Sass workflow has four stages:

~~~text
SCSS source file
      |
      v
Sass compiler
      |
      v
generated CSS file
      |
      v
browser loads and applies CSS
~~~

For example, a project may contain:

~~~text
project/
  index.html
  src/
    styles.scss
  dist/
    styles.css
~~~

The SCSS file is the source you edit. The CSS file is compiler output referenced by the page or application. A build tool may run Sass automatically; the underlying relationship is the same.

Install Sass in a practice project with npm:

~~~bash
npm install --save-dev sass
~~~

Compile one SCSS file to one CSS file:

~~~bash
npx sass src/styles.scss dist/styles.css
~~~

Watch a directory so Sass recompiles when a source file changes:

~~~bash
npx sass --watch src/scss:dist/css
~~~

The project must load the generated CSS. A browser will not compile a stylesheet linked directly to an SCSS file:

~~~html
<link rel="stylesheet" href="dist/styles.css">
~~~

The output path can differ. Use the paths and scripts configured by the project or framework.

## A first improvement: reuse a value

Suppose several parts of a page use the same brand color and spacing. Plain CSS can repeat those values, but changing them requires finding each copy. Sass variables let you define a value once and reuse it:

~~~scss
$brand-color: #2457a7;
$space: 1rem;

.button {
  background-color: $brand-color;
  padding: $space;
}

.notice {
  border-color: $brand-color;
  margin-block: $space;
}
~~~

Sass evaluates `$brand-color` and `$space` during compilation. The variable names do not appear in the resulting CSS:

~~~css
.button {
  background-color: #2457a7;
  padding: 1rem;
}

.notice {
  border-color: #2457a7;
  margin-block: 1rem;
}
~~~

This is a build-time substitution, not a browser variable. CSS custom properties such as `--brand-color` are different: they remain in CSS and can change at runtime. Later lessons compare these two kinds of variables.

## A first improvement: group related selectors

SCSS nesting can keep a component's related styles together:

~~~scss
.card {
  padding: 1rem;

  .card__title {
    margin: 0;
  }

  &:hover {
    box-shadow: 0 0.5rem 1rem rgb(0 0 0 / 15%);
  }
}
~~~

Sass compiles the nested selectors to ordinary CSS:

~~~css
.card {
  padding: 1rem;
}

.card .card__title {
  margin: 0;
}

.card:hover {
  box-shadow: 0 0.5rem 1rem rgb(0 0 0 / 15%);
}
~~~

The ampersand `&` stands for the parent selector, so `&:hover` becomes `.card:hover`. Without the ampersand, a nested selector such as `.card__title` becomes the descendant selector `.card .card__title`.

Nesting is only an authoring convenience. It still emits CSS selectors, so deep nesting can create long, tightly coupled selectors. Keep nesting shallow and check that the generated selector matches the element you intend.

## What Sass does not do

Sass is a stylesheet language and compiler. It is not:

- A replacement for HTML, CSS, or JavaScript.
- A browser feature or runtime styling engine.
- A UI framework, component library, or application architecture.
- A guarantee that styles are reusable or easy to maintain.
- A requirement for every website.

Modern CSS has native custom properties, nesting, calculations, layers, and many other features. Sass remains useful for its module system, compile-time logic, mixins, functions, and mature tooling. Choose it when the project benefits from those capabilities and the team can maintain the build step.

Use plain CSS when it already expresses the design clearly. Adding Sass introduces a compiler and a source-to-output workflow, so use Sass features to solve real authoring or organization problems rather than adding abstraction for its own sake.

## A small complete example

Create `src/styles.scss`:

~~~scss
$brand-color: #2457a7;
$surface-color: #f3f7fc;
$space: 1rem;

.card {
  max-width: 32rem;
  padding: $space;
  color: #202a35;
  background-color: $surface-color;
  border: 1px solid $brand-color;
  border-radius: 0.5rem;

  &__title {
    margin-block: 0 $space;
    color: $brand-color;
  }

  &__link {
    color: $brand-color;

    &:hover {
      color: #163b74;
    }
  }
}
~~~

Compile it:

~~~bash
npx sass src/styles.scss dist/styles.css
~~~

The generated `dist/styles.css` contains ordinary CSS. The nested title and link rules become `.card__title` and `.card__link`; the hover rule becomes `.card__link:hover`. Sass replaces each variable with its value.

Use the generated stylesheet from `index.html`:

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Sass introduction</title>
    <link rel="stylesheet" href="dist/styles.css">
  </head>
  <body>
    <main>
      <article class="card">
        <h1 class="card__title">Learning Sass</h1>
        <p>Sass source is compiled into CSS before the browser applies it.</p>
        <a class="card__link" href="#next">Continue learning</a>
      </article>
    </main>
  </body>
</html>
~~~

The class names in the HTML and compiled CSS match. The browser only needs `dist/styles.css`; it does not need to know that the styles were authored in SCSS.

## A note about Sass implementations

Dart Sass is the primary, actively maintained Sass implementation. This roadmap uses the Dart Sass version listed in the repository README. Older implementations such as LibSass and Node Sass do not support all current Sass features, especially the modern module system. Use Dart Sass for new projects and check the build-tool documentation when working in an existing codebase.

## Common first questions

### Is SCSS a different language from Sass?

SCSS is one syntax for the Sass language. The other common syntax is indentation-based Sass. Both are compiled by Sass; this course primarily uses SCSS.

### Can I put regular CSS in an SCSS file?

Yes. Most valid CSS is valid SCSS. Sass adds authoring syntax and compiles the file to CSS.

### Can a browser load an SCSS file?

Browsers do not compile Sass. Compile the SCSS to CSS and load that CSS, or use a build tool that runs Sass as part of the application build.

### Do Sass variables become CSS custom properties?

No. Sass variables are evaluated during compilation. CSS custom properties remain in generated CSS and are evaluated by the browser.

### Does every project need Sass?

No. Sass is useful when its compile-time features and organization help a project. Plain CSS may be enough for a small or straightforward stylesheet.

## Key takeaways

- Sass is a stylesheet language compiled into CSS.
- SCSS is the CSS-like Sass syntax used throughout this course.
- The browser applies generated CSS, not Sass source.
- Sass adds authoring features such as variables and nesting; CSS still determines browser styling behavior.
- Sass is optional. Use it when its features solve a real project need.
- Inspect compiled CSS whenever Sass generates or transforms selectors and values.

## Try it yourself

1. Create a small SCSS file with one variable used by two declarations.
2. Compile it and find where the variable value appears in the CSS output.
3. Add one nested child selector and one `&:hover` selector.
4. Compare the SCSS source with the generated CSS.
5. Change a variable, recompile, and confirm every intended use changes.

## Official references

- [Sass documentation](https://sass-lang.com/documentation/)
- [Sass syntax: SCSS and indented syntax](https://sass-lang.com/documentation/syntax/)
- [Sass basics](https://sass-lang.com/guide/)
- [Install Sass](https://sass-lang.com/install/)
- [Dart Sass command-line interface](https://sass-lang.com/documentation/cli/)
