# 004. Compiling SCSS to CSS

## Learning goals

After this lesson, you should be able to:

- Compile one SCSS file to a chosen CSS output file.
- Compile several entrypoints or map a source directory to an output directory.
- Explain how Sass modules and partials participate in compilation.
- Choose readable or compressed CSS output.
- Use watch mode for an editing workflow.
- Understand what Sass source maps are for and where they are written.
- Diagnose common input, output, and path mistakes.

## The compiler’s job

A Sass compiler reads a Sass or SCSS entrypoint, loads the Sass files that entrypoint depends on, evaluates Sass features, and writes CSS.

~~~text
main.scss ──uses──> _tokens.scss
    |
    v
Sass compiler
    |
    +──> main.css
    +──> main.css.map
~~~

The entrypoint is the stylesheet you compile directly. A dependency is another stylesheet loaded by that entrypoint, for example with @use. Sass evaluates dependencies as part of compiling the entrypoint. It does not need to create a separate CSS file for every dependency.

The compiler turns Sass syntax into CSS syntax. Variables are resolved, nested rules are expanded, and Sass functions are evaluated. CSS rules that do not require Sass-specific processing remain CSS in the output.

## Compile one file

With Sass installed locally in an npm project, compile one SCSS file to one CSS file:

~~~sh
npx sass src/styles.scss dist/styles.css
~~~

The first argument is the input. The second is the output. If the output path is omitted, Sass writes the CSS to standard output:

~~~sh
npx sass src/styles.scss
~~~

One-to-one compilation is useful when a project has a single stylesheet entrypoint or when you want to see exactly which source file creates which output file.

The input extension determines its syntax:

- A .scss file uses SCSS syntax.
- A .sass file uses the indented Sass syntax.
- A .css file is parsed as plain CSS.

Keep the source file and generated output file distinct. Edit src/styles.scss; treat dist/styles.css as generated output.

## Compile multiple entrypoints

A project may have separate entrypoints for different parts of an application. Compile each input to its own CSS output:

Use input:output pairs for multiple entrypoints:

~~~sh
npx sass src/site.scss:dist/site.css src/admin.scss:dist/admin.css
~~~

Each entrypoint produces a separate CSS file. If both entrypoints load common Sass modules, Sass evaluates those modules in the context of each compilation. The resulting CSS is not automatically combined into one shared file.

Choose entrypoints based on the CSS bundles the application needs. Do not compile every utility or partial as its own page stylesheet unless it is meant to be loaded independently.

## Map a source directory to an output directory

Many-to-many mode maps a directory of Sass files to a directory of CSS files:

~~~sh
npx sass src/scss:public/css
~~~

For example:

~~~text
src/scss/
├── site.scss
├── admin.scss
├── _tokens.scss
└── components/
    ├── _button.scss
    └── _card.scss
~~~

The CLI compiles standalone Sass files in the input directory and preserves subdirectory structure in the output. Files whose names begin with an underscore are Sass partials; the directory compiler skips them as standalone outputs. They can still be loaded by entrypoints.

A useful convention is:

- Use ordinary names for entrypoints, such as site.scss.
- Use underscore-prefixed names for implementation files that are meant to be loaded by another stylesheet, such as _tokens.scss.
- Load dependencies from the entrypoint with the Sass module system.

For example, site.scss can load a token module:

~~~scss
@use "tokens" as tokens;

body {
  color: tokens.$text;
  background: tokens.$surface;
}
~~~

If the input folder contains only partials, directory compilation produces no CSS files. Compile an entrypoint that uses those partials instead.

## Choose an output style

Dart Sass supports two CSS output styles:

| Style | Result | Typical use |
| --- | --- | --- |
| expanded | Readable formatting with selectors and declarations on separate lines. This is the default. | Local development and debugging. |
| compressed | Removes most optional whitespace and emits compact CSS. | Smaller direct compiler output where the build pipeline does not handle compression. |

Use the style option to select one:

~~~sh
npx sass --style=expanded src/styles.scss dist/styles.css
npx sass --style=compressed src/styles.scss dist/styles.min.css
~~~

Compressed output only changes formatting and removes optional whitespace. It does not replace a complete production asset pipeline that may also handle bundling, hashing, browser targets, or other optimizations.

## Watch while editing

Use watch mode to compile initially and then recompile when source files change:

~~~sh
npx sass --watch src/scss:public/css
~~~

Sass prints compilation status in the terminal. Keep the process running while you edit and stop it with the terminal’s interrupt command when finished.

Sass watches the input directories and file parents you pass, as well as configured load paths. If a dependency lives in a location outside the watched directories and load paths, make sure the watch command includes the relevant path. This avoids a stale build when an external dependency changes.

For a single file, use a source-to-output mapping:

~~~sh
npx sass --watch src/styles.scss:dist/styles.css
~~~

## Rebuild only changed files

The update mode compiles only files whose sources or dependencies are newer than the corresponding CSS output:

~~~sh
npx sass --update src/scss:public/css
~~~

This can save work in a larger source directory. For a simple project, a regular compile is often easier to understand. Watch mode performs an initial update and then keeps watching for later changes.

## Source maps

A source map connects generated CSS back to the Sass files that produced it. Browser developer tools can use the map to show the original source location while you inspect a CSS rule.

Dart Sass generates source maps by default when it writes CSS files. The map is typically written next to the CSS output, and the CSS contains a sourceMappingURL comment that points to it.

~~~text
dist/
├── styles.css
└── styles.css.map
~~~

To compile without a source map:

~~~sh
npx sass --no-source-map src/styles.scss dist/styles.css
~~~

Relative source-map URLs are the default and are generally appropriate for a project whose source and output paths are served consistently. Absolute file URLs refer to the compiler machine’s filesystem and are usually unsuitable for CSS deployed to another machine.

Source maps are a debugging aid, not an alternate stylesheet. The browser still applies the generated CSS.

## Handle compilation errors

If Sass cannot compile the input, it reports the error in the terminal with a source location when available. A failed compilation means you should inspect the diagnostic before trusting the output CSS. In watch or update workflows, Sass reports the failure while continuing or stopping according to the command and options in use.

A reliable debugging order is:

1. Read the first error message and identify its file and line.
2. Check the input file and any dependencies named by the diagnostic.
3. Confirm the command’s working directory and input path.
4. Correct the source and run the compile command again.
5. Check that the expected output file has a fresh modification time.
6. Reload the page and confirm that it loads the expected output path.

Do not try to fix generated CSS by hand when the next compilation will overwrite it. Correct the Sass source and compile again.

## Common mistakes

### Reversing the input and output

The input comes first and the output comes second:

~~~sh
npx sass src/styles.scss dist/styles.css
~~~

### Linking the page to the source

The browser should load dist/styles.css, not src/styles.scss.

### Compiling partials as standalone stylesheets

An underscore-prefixed partial is usually a dependency. Compile the entrypoint that loads it.

### Forgetting that directory mode creates several CSS files

A source directory can contain multiple entrypoints. The CLI mirrors them into the output directory. Confirm which of those output files the application should load.

### Expecting changes without running a compiler

Sass output is generated. Use a fresh one-time compile, update command, or running watcher after changing the source.

## Key takeaways

- A compile command maps Sass input to CSS output.
- A project can compile one file, several entrypoints, or a directory tree.
- An entrypoint produces CSS; partials are dependencies and are skipped as standalone outputs in directory mode.
- Expanded CSS is readable and is the default; compressed CSS is more compact.
- Watch mode recompiles during editing, and update mode can skip unchanged outputs.
- Source maps connect generated CSS to its Sass origins for debugging.
- Change the source and rebuild instead of editing generated CSS.

## Practice

1. Compile one SCSS file to a separate output path and inspect both files.
2. Compile two entrypoints and confirm that Sass produces two CSS files.
3. Add an underscore-prefixed Sass file and use it from an entrypoint; check that it is not emitted as an independent CSS file in directory mode.
4. Run watch mode, edit a declaration, and observe the compilation message.
5. Compile with expanded and compressed styles and compare the output formatting.
6. Open a generated CSS file with its source map and locate a rule in the original SCSS.
7. Introduce a temporary syntax error, read the diagnostic, fix it, and compile successfully again.

## Further reading

- [Dart Sass command-line interface](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass @use rule](https://sass-lang.com/documentation/at-rules/use/)
- [Sass stylesheet structure](https://sass-lang.com/documentation/syntax/structure/)
