# Sass CLI, watch mode, and build options — Examples

These command examples use Dart Sass. Run them from the project root, or adjust paths to match the current directory. Check the installed compiler with <code>sass --version</code> or <code>npx sass --version</code>.

## 1. Compile one entry point to a CSS file

~~~sh
sass src/styles/app.scss public/css/app.css
~~~

Sass writes the compiled CSS to the output path. The entry point can load shared partials with <code>@use</code>.

## 2. Print CSS to the terminal

~~~sh
sass src/styles/app.scss
~~~

When no output path is supplied in one-to-one mode, Sass prints the compiled CSS to standard output.

## 3. Map two entry files to two outputs

~~~sh
sass src/styles/light.scss:public/css/light.css src/styles/dark.scss:public/css/dark.css
~~~

Each input is compiled into its corresponding output file. Keep the entry files intentional; do not include shared partials as independent entries.

## 4. Compile a directory of themes

Source:

~~~text
src/themes/
  light.scss
  dark.scss
  _shared-tokens.scss
~~~

Command:

~~~sh
sass src/themes:public/css/themes
~~~

The non-partial Sass files become CSS outputs. The underscore-prefixed shared partial is not emitted as its own output.

## 5. Watch a source directory

~~~sh
sass --watch src/styles:public/css
~~~

Sass performs an initial update and remains open to compile later edits. Stop the process with Ctrl+C.

## 6. Watch an entry file and its output

~~~sh
sass --watch src/styles/app.scss:public/css/app.css
~~~

Use this form when one known entry should produce one CSS file. Confirm that files outside the watched directories are covered through the input’s parent directories or configured load paths.

## 7. Poll for file changes

~~~sh
sass --watch --poll src/styles:public/css
~~~

Use polling only if the filesystem does not reliably report changes. Sass checks for changes periodically instead of relying on normal notifications.

## 8. Incrementally update a theme directory

~~~sh
sass --update src/themes:public/css/themes
~~~

Sass checks the source files and dependencies against output modification times and recompiles what is out of date.

## 9. Stop a multi-input build on the first error

~~~sh
sass --stop-on-error src/themes:public/css/themes
~~~

This is useful in CI when later compilation errors are not helpful after one entry has already failed.

## 10. Compare expanded and compressed output

~~~sh
sass --style=expanded src/styles/app.scss public/css/app-expanded.css
sass --style=compressed src/styles/app.scss public/css/app.min.css
~~~

Expanded output is readable. Compressed output removes formatting characters. Separate output names prevent the second command from overwriting the first result.

## 11. Generate the default source map

~~~sh
sass src/styles/app.scss public/css/app.css
~~~

Dart Sass generates a source map by default. The CSS references the map, which helps browser tools trace rules back to Sass source.

## 12. Disable a source map for one build

~~~sh
sass --no-source-map --style=compressed src/styles/app.scss public/css/app.min.css
~~~

Use this for outputs where publishing a separate source map is not part of the workflow.

## 13. Set relative or absolute source-map URLs

~~~sh
sass --source-map-urls=relative src/styles/app.scss public/css/app.css
~~~

Relative URLs are the default and usually travel with the project’s CSS and source tree.

~~~sh
sass --source-map-urls=absolute src/styles/app.scss public/css/app.css
~~~

Absolute file URLs point to paths on the build machine. They generally are not useful after the output is moved elsewhere.

## 14. Embed Sass sources in the map

~~~sh
sass --embed-sources src/styles/app.scss public/css/app.css
~~~

The source map contains source text. This may make the map larger, but can make the source available when the original Sass files are not served.

## 15. Embed the map in CSS

~~~sh
sass --embed-source-map src/styles/app.scss public/css/app.css
~~~

This places source-map data inside the CSS rather than writing a separate map file. Consider the resulting CSS size before using this for a public output.

## 16. Add more than one load path

~~~sh
sass --load-path=src/shared --load-path=node_modules src/styles/app.scss public/css/app.css
~~~

Sass checks earlier load paths before later ones. Relative imports from the current stylesheet still take precedence over load paths.

## 17. Enable the Node package importer

~~~sh
sass --pkg-importer=node src/styles/app.scss public/css/app.css
~~~

Use this with a Dart Sass version that supports the Node package importer and Sass package URLs. Check the installed CLI version and dependency configuration first.

## 18. Show diagnostic CSS when compilation fails

~~~sh
sass --error-css src/styles/app.scss public/css/app.css
~~~

When compilation fails, Sass can write a diagnostic stylesheet that displays the error in a browser. The Sass process still reports the failure.

## 19. Disable error CSS

~~~sh
sass --no-error-css src/styles/app.scss public/css/app.css
~~~

With watch or update mode, Sass deletes CSS outputs when compilation fails if error CSS is disabled. Use this when stale or diagnostic CSS should not remain available.

## 20. Show all deprecation warnings

~~~sh
sass --verbose src/styles/app.scss public/css/app.css
~~~

This is useful when reviewing a migration or auditing outdated Sass syntax.

## 21. Suppress dependency deprecation warnings

~~~sh
sass --load-path=node_modules --quiet-deps src/styles/app.scss public/css/app.css
~~~

This suppresses deprecation warnings from dependencies loaded through load paths while preserving warnings from first-party source.

## 22. Make a selected deprecation fail compilation

~~~sh
sass --fatal-deprecation=import src/styles/app.scss public/css/app.css
~~~

This makes Sass <code>@import</code> deprecations fatal for this invocation. Confirm the selected ID is supported by the compiler version used in the project.

## 23. Disable charset markers

~~~sh
sass --no-charset src/styles/app.scss public/css/app.css
~~~

Use this only if the project’s output encoding and delivery path do not require Sass’s default charset marker for non-ASCII output.

## 24. Send SCSS to standard input in PowerShell

~~~powershell
"h1 { color: red; }" | sass --stdin output.css
~~~

Sass reads the piped text and writes the CSS to <code>output.css</code>. Standard input cannot be combined with many-to-many mode.

## 25. Inspect options for the installed compiler

~~~sh
sass --help
sass --version
~~~

For a local npm dependency:

~~~sh
npx sass --help
npx sass --version
~~~

This is useful when a shared project uses a different Sass version or a CLI flag is not recognized.

## 26. Combine common release options

~~~sh
sass --style=compressed --no-source-map --stop-on-error src/styles/app.scss public/css/app.min.css
~~~

This compiles once, uses compressed CSS, omits the source map, and stops on an error. Keep only the flags the actual release workflow needs.

## 27. Combine watcher options

~~~sh
sass --watch --poll --style=expanded src/styles/app.scss:public/css/app.css
~~~

This keeps a watcher open, polls for changes, and writes readable CSS. Polling is optional and should be removed when normal file notifications work.

## 28. Troubleshoot a module that does not resolve

Suppose the project has:

~~~text
src/
  shared/
    _tokens.scss
  styles/
    app.scss
~~~

If the entry point loads the shared module with a relative path, use:

~~~scss
@use "../shared/tokens";
~~~

Alternatively, configure the shared directory as a load path:

~~~sh
sass --load-path=src/shared src/styles/app.scss public/css/app.css
~~~

Then a module URL such as <code>@use "tokens";</code> can resolve through that path.

## 29. Troubleshoot an edit that does not trigger watch

Check the locations included in the watch command and its load paths. If the changed dependency sits outside those locations, include its directory. If the filesystem does not notify Sass about changes, try <code>--poll</code>.

## 30. Practice tasks

1. Compile one entry point to CSS and inspect its source map.
2. Compile two theme entry files to separate outputs.
3. Watch the source directory while editing a loaded partial.
4. Compare expanded output with compressed output.
5. Run an incremental update on a directory of Sass entry files.
6. Use <code>--verbose</code> to review deprecation warnings.
7. Decide whether error CSS should be enabled in local watch mode.
8. Confirm which CLI options are available in the project’s locked Sass version.

## Official references

- [Dart Sass CLI](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass CLI overview](https://sass-lang.com/documentation/cli/)
- [Sass installation](https://sass-lang.com/install/)
- [Sass breaking changes and deprecations](https://sass-lang.com/documentation/breaking-changes/)
