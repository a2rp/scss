# 014 — Comments and Generated CSS Output: Examples

These examples show what Sass keeps in its source and what appears in generated CSS. CLI commands use the Dart Sass executable named sass.

## 1. Compare silent and loud comments

Source SCSS:

~~~scss
// Source-only note: this line will not be emitted.

/* This block comment is emitted in expanded output. */
.button {
  color: white;
}
~~~

Expanded CSS:

~~~css
/* This block comment is emitted in expanded output. */
.button {
  color: white;
}
~~~

The // comment is absent from the output.

## 2. Keep a license notice in compressed CSS

Source SCSS:

~~~scss
/*! Copyright 2026 Example Project. See LICENSE for details. */

/* Internal note: use the brand color. */
.button {
  color: white;
  background-color: #2457c5;
}
~~~

Compile it in compressed mode:

~~~sh
sass --style=compressed src/button.scss public/button.min.css
~~~

The ordinary comment is stripped. The /*! license comment remains in the compressed CSS.

## 3. Interpolate a value into a loud comment

Source SCSS:

~~~scss
$release: "2.4.0";

/* Stylesheet release: #{$release} */
.button {
  display: inline-flex;
}
~~~

Generated CSS:

~~~css
/* Stylesheet release: 2.4.0 */
.button {
  display: inline-flex;
}
~~~

Interpolation in the loud comment is evaluated during compilation.

## 4. Document a reusable mixin with SassDoc

~~~scss
/// Adds a visible focus ring to an interactive control.
///
/// @param {Color} $color
///   The ring color.
/// @param {Length} $width
///   The ring width.
@mixin focus-ring($color, $width: 2px) {
  outline: $width solid $color;
  outline-offset: 2px;
}

.button:focus-visible {
  @include focus-ring(#2457c5);
}
~~~

Generated CSS:

~~~css
.button:focus-visible {
  outline: 2px solid #2457c5;
  outline-offset: 2px;
}
~~~

The documentation comment stays in the source. SassDoc tooling can read it separately.

## 5. Compile one file to one output file

Given src/styles.scss, compile to public/styles.css:

~~~sh
sass src/styles.scss public/styles.css
~~~

Dart Sass uses expanded output by default. If the output file is omitted, the CSS is printed in the terminal:

~~~sh
sass src/styles.scss
~~~

## 6. Compare expanded and compressed output

Source SCSS:

~~~scss
.card {
  color: #172033;
  border: 1px solid #d0d5dd;
}
~~~

Compile expanded:

~~~sh
sass --style=expanded src/card.scss public/card.css
~~~

Output:

~~~css
.card {
  color: #172033;
  border: 1px solid #d0d5dd;
}
~~~

Compile compressed:

~~~sh
sass --style=compressed src/card.scss public/card.min.css
~~~

Output:

~~~css
.card{color:#172033;border:1px solid #d0d5dd}
~~~

The formatting changes. The declarations and intended visual behavior stay the same.

## 7. Observe Sass syntax becoming CSS

Source SCSS:

~~~scss
$space: 1rem;
$brand: #2457c5;

@mixin panel {
  padding: $space;
  border: 1px solid #d0d5dd;
}

.panel {
  @include panel;
  color: #172033;

  a {
    color: $brand;

    &:hover {
      text-decoration: underline;
    }
  }
}
~~~

Generated CSS:

~~~css
.panel {
  padding: 1rem;
  border: 1px solid #d0d5dd;
  color: #172033;
}

.panel a {
  color: #2457c5;
}

.panel a:hover {
  text-decoration: underline;
}
~~~

The browser receives resolved values, expanded mixin declarations, and flattened selectors.

## 8. Compile multiple themes to an output directory

Dart Sass can compile a directory of Sass files to a directory of CSS files:

~~~sh
sass src/themes:public/css
~~~

If src/themes contains light.scss and dark.scss, Sass writes light.css and dark.css under public/css. Partial files whose names begin with an underscore are not compiled as separate outputs.

## 9. Watch Sass files while developing

Use watch mode to recompile when source files change:

~~~sh
sass --watch src/scss:public/css
~~~

Sass remains running and updates the corresponding CSS output. Stop the process when the development session is finished.

## 10. Create a source map

Dart Sass generates source maps by default when it writes CSS:

~~~sh
sass src/styles.scss public/styles.css
~~~

The output includes a source-map file and a reference that lets browser developer tools find it. Open the browser's developer tools to inspect original Sass sources through the map.

## 11. Turn source maps off

When the build does not need a map, disable it explicitly:

~~~sh
sass --no-source-map src/styles.scss public/styles.css
~~~

This generates the CSS without a source-map file.

## 12. Choose relative or absolute source-map URLs

Relative source URLs are portable when the Sass and map files keep their expected relative locations:

~~~sh
sass --source-map-urls=relative src/styles.scss public/styles.css
~~~

Absolute source URLs refer to the compiling machine's file paths:

~~~sh
sass --source-map-urls=absolute src/styles.scss public/styles.css
~~~

Absolute paths usually do not work on another computer. Use relative URLs for portable project output unless a local debugging setup specifically needs absolute paths.

## 13. Embed Sass sources in the source map

This command puts the contributing Sass source contents into the generated map:

~~~sh
sass --embed-sources src/styles.scss public/styles.css
~~~

The map becomes larger, but its original sources are available wherever it is served. Review this choice before publishing production maps.

## 14. Embed the source map in CSS

This command embeds the map content in the generated CSS instead of writing a separate linked map:

~~~sh
sass --embed-source-map src/styles.scss public/styles.css
~~~

The CSS file grows because the map data is included in it.

## 15. Control charset markers

Source SCSS with non-ASCII output:

~~~scss
.icon::before {
  content: "→";
}
~~~

By default, Dart Sass may add a charset marker when non-ASCII characters are emitted. To prevent that:

~~~sh
sass --no-charset src/icons.scss public/icons.css
~~~

Expanded output may use a @charset declaration; compressed output may use a UTF-8 byte-order mark. Select the behavior appropriate for the build and hosting environment.

## 16. Keep source-only notes out of CSS

~~~scss
// TODO for the Sass author: check this token with design.
$border-color: #d0d5dd;

.card {
  border-color: $border-color;
}
~~~

Generated CSS:

~~~css
.card {
  border-color: #d0d5dd;
}
~~~

This is the right form when a note is only for the maintainers of the source.

## 17. Put a required attribution in CSS

~~~scss
/*! Icon artwork by Example Author, used under the project license. */

.icon {
  display: inline-block;
}
~~~

That attribution remains even if the file is compiled with compressed output.

## 18. Avoid editing generated CSS

Source file:

~~~scss
$brand: #2457c5;

.link {
  color: $brand;
}
~~~

Generated file:

~~~css
.link {
  color: #2457c5;
}
~~~

If the output color needs to change, edit $brand in the SCSS and rebuild. A direct change to the generated file will be overwritten by the next compilation.

## 19. Practice tasks

1. Add a // note and a /* ... */ comment to a stylesheet. Predict which appears in expanded CSS.
2. Add a /*! license notice and compile with compressed output.
3. Use a variable inside a loud comment with interpolation.
4. Compile one input to an output file using expanded style.
5. Compile the same input with compressed style and compare the declarations.
6. Generate a source map, then compile again with source maps disabled.
7. Try relative source-map URLs and explain why absolute URLs are machine-specific.
8. Use a SassDoc comment above a mixin and confirm it is not emitted as CSS.
9. Nest a selector and inspect the flattened CSS.
10. Decide whether embedded source contents should be published with a production build.

### Sample source for tasks 1–5

~~~scss
// Source-only comment.

/* Visible in expanded output; removed in compressed output. */

/*! Preserved license notice. */

$brand: #2457c5;

/* Brand token: #{$brand} */
.button {
  color: white;
  background-color: $brand;
}
~~~

Expanded output contains the ordinary block comment and the preserved notice. Compressed output drops the ordinary block comment but keeps the /*! notice.

## Further reading

- [Sass comments](https://sass-lang.com/documentation/syntax/comments/)
- [Dart Sass CLI](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass documentation comments](https://sass-lang.com/documentation/syntax/comments/#documentation-comments)
