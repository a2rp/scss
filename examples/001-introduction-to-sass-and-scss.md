# 001. Example: Compile and use your first SCSS card

## Goal

Create a small card with SCSS variables and nested selectors, compile it to CSS, and load the generated stylesheet in an HTML page.

This example reinforces the first lesson's core workflow: edit SCSS, compile CSS, and let the browser apply the output.

## What you will practice

- Creating an SCSS source file.
- Reusing compile-time values with Sass variables.
- Nesting selectors and using the parent selector.
- Running Dart Sass from the command line.
- Linking generated CSS from HTML.
- Comparing source SCSS with compiled CSS.

## Project structure

Create this small practice project:

~~~text
scss-first-card/
  index.html
  src/
    styles.scss
  dist/
    styles.css
~~~

The CSS file will be created by the compiler. Make the `src` and `dist` folders before compiling.

## 1. Install Sass

Open a terminal in the `scss-first-card` project folder and install Sass:

~~~bash
npm install --save-dev sass
~~~

This adds Sass to the practice project's development dependencies. A framework or existing build tool may manage Sass for you; in that case, use its documented integration instead of adding a separate compile command.

## 2. Write the SCSS source

Create `src/styles.scss`:

~~~scss
$brand-color: #2457a7;
$surface-color: #f3f7fc;
$text-color: #202a35;
$space: 1rem;

.card {
  max-width: 32rem;
  padding: $space;
  color: $text-color;
  background-color: $surface-color;
  border: 1px solid $brand-color;
  border-radius: 0.5rem;

  &__title {
    margin-block: 0 $space;
    color: $brand-color;
  }

  &__description {
    margin-block: 0 $space;
  }

  &__link {
    color: $brand-color;
    font-weight: 700;

    &:hover,
    &:focus-visible {
      color: #163b74;
    }
  }
}
~~~

The variables define values reused in the stylesheet. Sass evaluates them when it compiles the file.

Inside `.card`, `&__title` appends `__title` to the parent selector, producing `.card__title`. The nested hover and keyboard-focus selectors become `.card__link:hover` and `.card__link:focus-visible`.

## 3. Create the HTML page

Create `index.html` in the project folder:

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>First SCSS card</title>
    <link rel="stylesheet" href="dist/styles.css">
  </head>
  <body>
    <main>
      <article class="card" aria-labelledby="card-title">
        <h1 class="card__title" id="card-title">Learning Sass</h1>
        <p class="card__description">
          SCSS source is compiled to CSS before the browser applies the styles.
        </p>
        <a class="card__link" href="#next">Continue learning</a>
      </article>
    </main>
  </body>
</html>
~~~

The stylesheet link points to the generated CSS path, not the SCSS source file. The class names in the HTML match the selectors Sass will emit.

## 4. Compile the SCSS

From the project folder, run:

~~~bash
npx sass src/styles.scss dist/styles.css
~~~

The compiler reads `src/styles.scss` and writes the resulting CSS to `dist/styles.css`. If compilation succeeds, open `index.html` in a browser. The card should have a pale background, a blue border and heading, and a link that changes color on hover or keyboard focus.

For automatic recompilation while you edit:

~~~bash
npx sass --watch src:dist
~~~

Stop watch mode with `Ctrl+C` in the terminal.

## 5. Compare the generated CSS

The output will be equivalent to:

~~~css
.card {
  max-width: 32rem;
  padding: 1rem;
  color: #202a35;
  background-color: #f3f7fc;
  border: 1px solid #2457a7;
  border-radius: 0.5rem;
}

.card__title {
  margin-block: 0 1rem;
  color: #2457a7;
}

.card__description {
  margin-block: 0 1rem;
}

.card__link {
  color: #2457a7;
  font-weight: 700;
}

.card__link:hover,
.card__link:focus-visible {
  color: #163b74;
}
~~~

The generated CSS contains the variable values and expanded selectors. It does not contain Sass variable names or nested SCSS blocks.

## 6. Make a change and recompile

Change `$brand-color` to another valid CSS color, save the file, and run the compiler again. The border, title, and link should all use the new value.

Then change `$space` and compile again. The card padding and the spacing below its title and description should change together. If the browser still shows old styles, confirm that Sass compiled successfully and that the page is loading `dist/styles.css`.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| The page has no card styling. | Confirm the compiler created `dist/styles.css` and that the HTML link uses that exact relative path. |
| The Sass command cannot find the source file. | Run the command from the project folder and verify `src/styles.scss` is spelled correctly. |
| The browser reports a missing stylesheet. | Check the `href` in the HTML and the output path used in the Sass command. |
| Variables appear literally in the browser. | The page may be loading an SCSS file or a different file instead of the generated CSS. Browsers do not compile Sass. |
| The nested title selector does not match. | Check the class on the HTML element and inspect the compiled selector `.card__title`. |
| A change does not appear. | Recompile or confirm watch mode is running, then reload the page and inspect the loaded CSS in developer tools. |

## Try it yourself

1. Add a `$border-radius` variable and use it for the card's rounded corners.
2. Add a nested `.card__badge` rule and a matching element in the HTML.
3. Add a `&:focus-within` rule to give the card an outline when a child control has focus.
4. Change a variable, recompile, and identify every generated declaration that changed.
5. Temporarily change the HTML link to `src/styles.scss`, observe that the browser does not compile it, then restore `dist/styles.css`.

## What this example demonstrates

- Sass variables are resolved during compilation.
- SCSS nesting is expanded into ordinary CSS selectors.
- The browser receives generated CSS, not Sass source.
- A successful compile does not automatically connect the output to a page; the page or build system must load the generated file.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td>Start of course</td><td><a href="../lessons/001-introduction-to-sass-and-scss.md">Lesson</a></td><td><a href="../qnas/001-introduction-to-sass-and-scss.md">Q&amp;A</a></td><td><a href="002-how-sass-fits-css-and-a-web-build.md">002 - How Sass Fits CSS and a Web Build - Examples</a></td></tr>
  </tbody>
</table>
