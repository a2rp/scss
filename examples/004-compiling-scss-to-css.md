# 004. Compiling SCSS to CSS — Examples

The examples below use the project-local Sass executable installed in the previous chapter. Run the commands from the project root so relative input and output paths resolve as shown.

## Example 1: Compile one stylesheet

Create this small project:

~~~text
scss-compile-demo/
├── index.html
├── src/
│   └── styles.scss
└── dist/
~~~

Create src/styles.scss:

~~~scss
$brand: #1769aa;

.notice {
  padding: 1rem;
  border-left: 0.25rem solid $brand;

  strong {
    color: $brand;
  }
}
~~~

Compile it into dist/styles.css:

~~~sh
npx sass src/styles.scss dist/styles.css
~~~

Sass reads the source file and writes CSS. The result is similar to:

~~~css
.notice {
  padding: 1rem;
  border-left: 0.25rem solid #1769aa;
}
.notice strong {
  color: #1769aa;
}
~~~

Dart Sass also writes dist/styles.css.map by default and adds a sourceMappingURL comment to the CSS.

Create index.html:

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>SCSS compilation</title>
    <link rel="stylesheet" href="./dist/styles.css">
  </head>
  <body>
    <p class="notice">
      <strong>Compiled stylesheet</strong>
      The browser loads the generated CSS output.
    </p>
  </body>
</html>
~~~

The browser links to dist/styles.css. It does not link to src/styles.scss.

If no output path is given, Sass prints the compiled CSS to standard output:

~~~sh
npx sass src/styles.scss
~~~

## Example 2: Compile two entrypoints

A site may have separate stylesheets for its public pages and its admin area. Add these files:

~~~text
src/
└── scss/
    ├── _tokens.scss
    ├── site.scss
    └── admin.scss
~~~

Create src/scss/_tokens.scss:

~~~scss
$brand: #1769aa;
$surface: #f4f7fb;
$text: #17212b;
~~~

Create src/scss/site.scss:

~~~scss
@use "tokens" as tokens;

body {
  color: tokens.$text;
  background: tokens.$surface;
}

.site-header {
  padding: 1rem;
  color: white;
  background: tokens.$brand;
}
~~~

Create src/scss/admin.scss:

~~~scss
@use "tokens" as tokens;

.admin-panel {
  padding: 1.5rem;
  color: tokens.$text;
  border-top: 0.25rem solid tokens.$brand;
}
~~~

Compile each entrypoint to its own CSS file:

~~~sh
npx sass src/scss/site.scss:public/css/site.css src/scss/admin.scss:public/css/admin.css
~~~

This creates two outputs: public/css/site.css and public/css/admin.css. Each entrypoint loads the shared token partial while compiling. The shared partial does not need to become a third stylesheet.

Use the CSS file that matches the page:

~~~html
<link rel="stylesheet" href="./public/css/site.css">
~~~

~~~html
<link rel="stylesheet" href="./public/css/admin.css">
~~~

The Sass module syntax here only demonstrates that an entrypoint can use a dependency. The module system is covered in detail in a later chapter.

## Example 3: Compile a directory tree

With the same source files, map the directory to the output directory:

~~~sh
npx sass src/scss:public/css
~~~

The result will be similar to:

~~~text
public/css/
├── admin.css
├── admin.css.map
├── site.css
└── site.css.map
~~~

Sass emits CSS for the ordinary entrypoint files and skips _tokens.scss as a standalone output. If you add a nested directory of entrypoints, Sass preserves that directory structure below public/css.

The directory command does not combine site.scss and admin.scss into one file. It keeps their outputs separate.

## Example 4: Compare expanded and compressed CSS

Expanded output is readable and is the default:

~~~sh
npx sass --style=expanded src/scss/site.scss public/css/site.css
~~~

Compressed output removes optional formatting whitespace:

~~~sh
npx sass --style=compressed src/scss/site.scss public/css/site.min.css
~~~

Both outputs represent CSS generated from the same source. Compressed output is compact formatting; a full production toolchain may perform additional tasks.

## Example 5: Compile without source maps

Source maps are useful when debugging in browser tools. If a project should not emit a map for a particular compile, use:

~~~sh
npx sass --no-source-map src/scss/site.scss public/css/site.css
~~~

The CSS remains the browser-facing stylesheet. Only the mapping file and corresponding sourceMappingURL comment are omitted.

## Example 6: Watch a source directory

While editing, keep Sass running in watch mode:

~~~sh
npx sass --watch src/scss:public/css
~~~

Sass performs an initial compilation and then reports subsequent updates in the terminal. Leave the watcher active while changing site.scss or admin.scss.

If a dependency is stored outside the input directories and configured load paths, include its directory or load path in the watch command so changes there can trigger recompilation.

Stop the watcher with the terminal interrupt command when finished.

## Example 7: Update only stale outputs

For a one-time build that skips outputs which are already up to date, use:

~~~sh
npx sass --update src/scss:public/css
~~~

Sass compares the input files and their dependencies with the output timestamps. This command exits after its update pass; it does not stay open like watch mode.

## Troubleshooting checklist

### No output file appears

Check the input path, output path, terminal’s current directory, and Sass error messages. Confirm that the source is an ordinary entrypoint rather than an underscore-prefixed partial.

### The page still has old styles

Run the compiler again or confirm the watcher is running. Then check that the browser loads the CSS file that was just generated.

### The command creates more than one stylesheet

Directory mode treats each non-partial Sass file as a possible entrypoint. If a file should only be a dependency, name it as a partial and load it from an entrypoint.

### The browser developer tools show CSS instead of SCSS

Check that the .css.map file exists, that the CSS has its sourceMappingURL comment, and that developer tools have source maps enabled.

### The CSS output shows a Sass error

Read the terminal diagnostic and fix the source. Sass may write error CSS for a failed file compilation, so do not treat the existence of an output file as proof that compilation succeeded.

## Try these changes

1. Add a declaration to site.scss and run the directory compile command again.
2. Add an underscore-prefixed partial and use it from one entrypoint.
3. Add another ordinary entrypoint and observe the extra CSS output.
4. Compile with expanded and compressed modes and compare the formatting.
5. Run watch mode, edit an entrypoint, and observe the terminal update.
6. Disable source maps and compare the output directory contents.
7. Break a Sass declaration, read the compiler diagnostic, fix it, and rebuild.
